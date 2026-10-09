> 一个查询，数据量不大，却慢得离谱：EXPLAIN 显示走了全表扫描，明明有索引。这篇复盘 PostgreSQL 慢查询排查的完整方法论——从执行计划读起，到统计信息、索引、BUFFERS，一层层剥到根因。

周二下午两点，订单列表接口的 P99 从 80ms 涨到 4s，告警群开始刷屏。

我先看监控：CPU 40%，内存 60%，磁盘 IO 正常——**经典的"什么都没满，但就是慢"**。更诡异的是，慢的只有一个接口：订单列表。其他接口一切正常。

把那条 SQL 拎出来：

```sql
SELECT * FROM orders WHERE user_id = 123 AND status = 'paid'
ORDER BY created_at DESC LIMIT 20;
```

orders 表 500 万行，`(user_id, status)` 上明明有索引。一个带索引的条件查询，凭什么跑 4 秒？

## 第一步：先找到"真凶 SQL"，别急着 EXPLAIN

很多人的第一反应是直接 EXPLAIN。但生产环境有几百条 SQL，先问"到底是哪条慢"，而不是"我猜是这条"：

```sql
-- pg_stat_statements：按总耗时排序，找到真正的 top SQL
SELECT query, calls, mean_exec_time, max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time * calls DESC
LIMIT 5;
```

（没装 pg_stat_statements 的，现在就去装——它是 PG 的"行车记录仪"，没有它，慢查询排查就是盲人摸象。）

确认就是订单列表那条 SQL 之后，才轮到 EXPLAIN 登场。

## 第二步：看执行计划，别猜

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM orders
WHERE user_id = 123 AND status = 'paid'
ORDER BY created_at DESC LIMIT 20;
```

关键看这几行：

```
Seq Scan on orders  (cost=0.00..182341.22 rows=12 width=...)
  Filter: (user_id = 123 AND status = 'paid')
  Rows Removed by Filter: 4999988
```

**Rows Removed by Filter: 4999988**——扫了 500 万行，扔了 499 万。这是全表扫描的铁证：有索引，但优化器没用。

读执行计划的三板斧：

1. **Node 类型**：Seq Scan（全表扫）、Index Scan（索引+回表）、Index Only Scan（只扫索引）、Bitmap Heap Scan（多索引合并）——类型直接告诉你"它选了什么路"；
2. **cost vs actual**：`cost=0.00..182341.22` 是优化器的*估计*，`actual time=... rows=12` 是*实际*。**估计和实际差一个数量级 = 统计信息有问题**，这是最重要的信号；
3. **Planning Time vs Execution Time**：如果 Planning Time 很高（几百 ms），可能是 prepared statement 反复生成计划，或统计信息复杂——慢的不一定是执行。

⚠️ 注意：`ANALYZE` 会**真实执行**这条 SQL。SELECT 没事，但 `EXPLAIN ANALYZE UPDATE/DELETE` 会真的改数据——排查写操作时，先在事务里 `BEGIN; EXPLAIN ANALYZE ...; ROLLBACK;`。

## 第三步：为什么不用索引——统计信息过期

优化器不是傻子，它放着索引不用，一定是"觉得"全表扫描更便宜。它的判断依据只有一个：**统计信息**。

```sql
-- 看优化器的"认知"
SELECT reltuples, relpages FROM pg_class WHERE relname = 'orders';
-- reltuples 显示 10 万，但实际 500 万
```

**根因找到了**：`reltuples` 显示 10 万行，实际 500 万行。在优化器的认知里，这是一张"小表"，全表扫描确实更便宜——它的逻辑没错，错的是它的"认知"。

为什么统计信息会过期？看 autoanalyze 的触发条件：

```
触发阈值 = autovacuum_analyze_threshold(默认50)
         + autovacuum_analyze_scale_factor(默认0.1) × 表行数
```

对 500 万行的表，阈值是 50 万行——还算合理。但对 5 亿行的大表，阈值是 **5000 万行**：业务高峰期一天写入几百万，要攒十几天才触发一次 analyze。**scale_factor 在大表上是线性放大的，写入速度却不是**——这是生产环境最经典的统计信息过期场景。

修法：

```sql
ANALYZE orders;  -- 手动刷新，查询立刻从 4s 降到 20ms
```

治本：

```sql
-- 给大表单独设更激进的阈值
ALTER TABLE orders SET (autovacuum_analyze_scale_factor = 0.02);
-- 或对超大表加定时任务，每天凌晨 ANALYZE
```

想看更细的"认知偏差"，查 `pg_stats`：直方图（histogram_bounds）、最常见值（most_common_vals）——优化器对"这个值有多少行"的估计，全来自这里。

```sql
-- 看优化器对某一列的"认知"细节
SELECT attname, n_distinct, most_common_vals
FROM pg_stats WHERE tablename = 'orders' AND attname = 'status';
-- most_common_vals 告诉你优化器以为 'paid' 占多大比例
-- 如果 ANALYZE 之后还不准，调大该列的统计目标（默认 100）：
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 500;
```

数据倾斜严重的列（比如 99% 是 'paid'），值得给更高的统计目标——采样更细，估计更准。**记住这句话：优化器只信统计信息，不信你的直觉。**

## 第四步：索引选对了吗

就算用了索引，还有三层讲究：

### 讲究一：复合索引的列顺序

```sql
-- 查询：WHERE user_id = 123 AND status = 'paid' ORDER BY created_at DESC
-- 原索引：(user_id, status) → 能用，但 ORDER BY 还要额外 sort

-- 优化：把排序列也吃进索引
CREATE INDEX idx_orders_user_status_created
ON orders (user_id, status, created_at DESC);
```

列顺序的原则：**等值条件在前，范围条件在后，排序列紧跟**。`(user_id, status, created_at)` 里，前两列等值定位，第三列直接有序输出，sort 步骤消失。

### 讲究二：覆盖索引（Index Only Scan）

```sql
-- 如果查询只取这几列，索引里全有，就不用回表
SELECT user_id, status, created_at FROM orders
WHERE user_id = 123 AND status = 'paid' ORDER BY created_at DESC;
-- → Index Only Scan：只扫索引， Heap Fetches 接近 0
```

覆盖索引的收益取决于 **visibility map**：如果表刚被大量更新、VM 没置位，Index Only Scan 会退化成"索引+回表验证"，收益大打折扣。所以覆盖索引 + 及时的 vacuum 是一对，单用一半功力。

### 讲究三：索引不是越多越好

每个索引都是写放大的来源：一次 INSERT 要更新表 + 所有索引。对写入频繁的表，**多一个索引，写入慢一分**。定期查 `pg_stat_user_indexes`，`idx_scan = 0` 的索引就是吃白饭的，删。

另外两个利器：**部分索引**和**表达式索引**。

```sql
-- 部分索引：只给热数据建，体积小、维护便宜
CREATE INDEX idx_orders_paid_recent ON orders (user_id, created_at DESC)
WHERE status = 'paid';
-- 查询条件必须包含 WHERE status='paid' 才能命中
-- 索引是查询的影子：影子只盖得住它照得见的地方
```

表达式索引（`ON lower(email)`）则让函数查询也能走索引——原理一样：索引里存什么，查询就要长成什么样。

## 第五步：BUFFERS——定性是 IO 还是 CPU 的问题

```
Buffers: shared hit=142 read=28103
```

- `read=28103`：大量物理读——数据不在内存，问题在 **IO**（加内存 / 优化索引减少读取 / 查是不是缓存被挤掉了）；
- `shared hit` 很高但还是慢：数据都在内存，问题在 **CPU**（排序、聚合、函数计算——查 work_mem 是不是溢出了）；
- `temp read/write` 出现：**work_mem 不够**，排序溢出到磁盘了。这是"内存够但配置小"的典型症状，调大 work_mem（按连接数折算，别无脑调）。

**IO 还是 CPU，决定了你加内存还是加 CPU，还是改 SQL**——方向错了，扩容就是烧钱。

## 执行计划里的魔鬼细节：Join 策略

单表查完，生产环境更多慢查询死在 Join 上。优化器有三种 Join 策略，选错就慢一个数量级：

| 策略 | 做法 | 适合 | 计划里的样子 |
|---|---|---|---|
| Nested Loop | 外表每行去内表查一次 | 外表很小（几十行）+ 内表有索引 | `Nested Loop` |
| Hash Join | 内表建 hash 表，外表逐行探测 | 两表都大、无合适索引 | `Hash Join` |
| Merge Join | 两边先排序再归并 | 两边已按 join 键有序 | `Merge Join` |

最常见的翻车：外表实际有 100 万行（优化器以为只有 100 行），选了 Nested Loop——100 万次索引查找，直接跑到天荒地老。**根因还是统计信息**（见第三步），但表象是 Join 策略。

调试时可以用 `SET enable_nestloop = off` 临时禁用某种策略，对比计划变化——但这只是诊断手段，**别把 `SET enable_* = off` 写进生产配置**，那是给优化器截肢。

还有一个隐形杀手：`LIMIT + ORDER BY`。`ORDER BY created_at DESC LIMIT 20` 不需要全排序，用的是 top-N heapsort（只维护 20 个元素的堆）。但如果 `ORDER BY` 的列没有索引支撑，优化器可能先全表排序再截断——5 亿行的全排序，内存和时间双杀。这就是为什么第四步要把排序列吃进复合索引。

## 慢性病：表膨胀（bloat）

有些慢查询不是"突然"变慢的，是**一天比一天慢**——这是表膨胀的典型症状。

PG 的 MVCC 机制下，UPDATE/DELETE 不会原地修改，而是产生 dead tuple，等 vacuum 回收。如果 autovacuum 跟不上大批量更新（又是那个 scale_factor 阈值问题），表里会堆积大量 dead tuple：

```sql
-- 用 pgstattuple 估算膨胀率（大表上采样，别全量扫）
SELECT * FROM pgstattuple('orders');
-- dead_tuple_percent 超过 20%，就该手动 VACUUM 了
```

膨胀的表，全表扫描要读的 page 是实际数据的数倍——Seq Scan 越来越慢，索引扫描也要多跳很多 heap page。**这时候加索引没用，治的是症状**，`VACUUM` 才是病因治疗。

⚠️ `VACUUM FULL` 能彻底重写表回收空间，但会**锁表**——生产环境用 `pg_repack` 做在线重整，别直接 FULL。

## 常见坑清单：90% 的慢查询死在这

1. **隐式类型转换**：`WHERE user_id = '123'`（字符串），索引直接失效。**类型必须对齐**，这是 ORM 时代的第一大坑。
2. **`OR` 条件**：`WHERE a=1 OR b=2` 很难用索引，改写成 `UNION`（两个索引各走各的）。
3. **`LIKE '%xxx'`**：前导通配符杀索引。用 `pg_trgm` 扩展，或上全文检索。
4. **`OFFSET` 深翻页**：`OFFSET 100000 LIMIT 20` 要扫 10 万行再扔掉。改 keyset 翻页：`WHERE id > last_id ORDER BY id LIMIT 20`。
5. **`SELECT *`**：拿了不需要的列，断了覆盖索引的路，还多传了网络 IO。
6. **prepared statement 的 generic plan**：前 5 次用 custom plan（按参数优化），第 6 次起可能切 generic plan（一刀切）。参数倾斜严重的查询（`status='paid'` 占 90% vs `status='refunding'` 占 1%），generic plan 会翻车——用 `plan_cache_mode = force_custom_plan` 救急。
7. **"慢查询"可能是锁等待**：SQL 本身 20ms，但被锁堵了 4s。先查谁在等：

```sql
-- 谁在等锁
SELECT pid, usename, query, wait_event_type, wait_event
FROM pg_stat_activity WHERE wait_event_type = 'Lock';
-- 谁堵了它（PG 14+）
SELECT blocked.pid AS blocked_pid, blocking.pid AS blocking_pid,
       blocking.query AS blocking_query
FROM pg_stat_activity blocked
JOIN pg_stat_activity blocking
  ON blocking.pid = ANY(pg_blocking_pids(blocked.pid));
```

**分不清"执行慢"和"等待久"，就会一直优化错地方。** 锁等待的解法是缩短事务（别在事务里调外部接口）、按固定顺序加锁，**不是**加索引。

## 防患：让慢查询进不了生产

排查是事后，防患是事前：

1. **CI 卡点**：迁移脚本里给大表加索引，必须用 `CREATE INDEX CONCURRENTLY`——普通建索引会锁表，大表上就是一次计划内事故；
2. **影子压测**：预发环境灌**生产级**数据量，用真实流量回放跑一遍，对 top 20 SQL 抽查 EXPLAIN；
3. **慢查询告警**：`log_min_duration_statement = 500`（超 500ms 记日志）+ `pg_stat_statements` 进 Prometheus，top SQL 耗时突增就告警——别等用户投诉才发现；
4. **SQL 评审红线**：`SELECT *`、无 WHERE 的 UPDATE/DELETE、OFFSET 深翻页，CR 直接打回。慢查询多半不是写出来的，是放行进去的。
5. **auto_explain 留证据**：偶发慢查询最难查——`auto_explain.log_min_duration = 1000`，超 1s 的查询自动记录执行计划。复现不了的慢查询，至少有"黑匣子"。

## 回到周二下午两点

根因最后定位到：那张 500 万行的 orders 表，前一天晚上做了一次大批量数据订正，写入了近 100 万行——而 autoanalyze 的触发阈值是 50 万行，还没触发。优化器的"认知"停留在订正之前，果断选择了全表扫描。

修法很便宜，三件事：

1. 给 orders 表单独设 `autovacuum_analyze_scale_factor = 0.02`；
2. 加一条每天凌晨的定时 `ANALYZE`，专治大批量写入后的统计信息滞后；
3. 把 `pg_stat_statements` 的 top SQL 接进告警——**下次 P99 抖动时，先看行车记录仪，再猜**。

复盘会上，没人再问"是不是 PG 不行了"。慢查询排查到最后，拼的从来不是数据库品牌，而是你对"优化器只信统计信息"这句话的理解有多深。

##{"timestamp":1756440000}