> 一个查询，数据量不大，却慢得离谱：EXPLAIN 显示走了全表扫描，明明有索引。这篇复盘 PostgreSQL 慢查询排查的完整方法论，从执行计划读起。

## 症状

```sql
SELECT * FROM orders WHERE user_id = 123 AND status = 'paid'
ORDER BY created_at DESC LIMIT 20;
```

- orders 表 500 万行，`(user_id, status)` 上有索引。
- 查询要 3 秒，EXPLAIN 显示 Seq Scan。

## 排查四步

### 1. 看执行计划，别猜

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```

关键看三行：

```
Seq Scan on orders  (cost=0.00..182341.22 rows=12 ...)
  Filter: (user_id = 123 AND status = 'paid')
  Rows Removed by Filter: 4999988
```

**Rows Removed by Filter: 4999988**——扫了 500 万行，扔了 499 万。这就是全表扫描的铁证。

### 2. 为什么不用索引：统计信息过期

```sql
-- 看优化器的"认知"
SELECT reltuples, relpages FROM pg_class WHERE relname = 'orders';
-- reltuples 显示 10 万，但实际 500 万
```

**根因**：`auto_analyze` 没跟上。大批量导入数据后，统计信息还是旧的，优化器以为表很小，"全表扫描更便宜"。

```sql
ANALYZE orders;  -- 手动刷新，查询立刻降到 20ms
```

### 3. 索引选对了吗

就算用了索引，还有讲究：

```sql
-- 原索引：(user_id, status)
-- 查询还要 ORDER BY created_at → 需要 sort

-- 优化：覆盖索引
CREATE INDEX idx_orders_user_status_created
ON orders (user_id, status, created_at DESC);
```

**覆盖索引**让查询只扫索引不回表，`Index Only Scan`，再快一个数量级。

### 4. 终极武器：看 BUFFERS

```
Buffers: shared hit=142 read=28103
```

`read=28103` 说明大量物理读——数据不在内存。如果 `shared hit` 高但还是慢，问题在 CPU（比如排序）；如果 `read` 高，问题在 IO（加内存 / 优化索引）。

## 常见坑清单

1. **隐式类型转换**：`user_id = '123'`（字符串），索引失效。类型必须对齐。
2. **`OR` 条件**：`WHERE a=1 OR b=2` 很难用索引，改写成 UNION。
3. **LIKE '%xxx'**：前导通配符杀索引，用 pg_trgm 或全文检索。
4. **`OFFSET` 翻页**：`OFFSET 100000` 要扫 10 万行再扔掉，用 keyset（`WHERE id > last_id`）。

## 一句话总结

PG 慢查询排查公式：**EXPLAIN ANALYZE 看计划 → 查统计信息是否过期 → 索引是否覆盖 → BUFFERS 定 IO 还是 CPU**。90% 的慢查询是统计信息或索引问题，不是 PG 不行。记住：优化器只信统计信息，不信你的直觉。

##{"timestamp":1756440000}