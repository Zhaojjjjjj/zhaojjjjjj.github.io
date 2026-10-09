> 我们的 Node 服务有个"传统"：每周三凌晨 4 点，cron 定时重启一次。没人觉得不对——直到某天业务量翻倍，重启间隔从 7 天缩到 2 天，再缩到 8 小时，最后变成"重启完 3 小时就 OOM"。这篇复盘那次内存泄漏排查：从"重启大法"失效，到 heapdump 双快照锁定真凶，到两行代码修复。

## 症状：慢性泄漏，不是急性雪崩

先说清楚这不是 Event Loop 饥饿那种"突然就崩"的事故，而是慢性病，症状很典型：

- **RSS 缓慢爬升**：从 200MB 起步，每天涨几百 MB，GC 后不回落；
- **heapUsed 和 heapTotal 一起涨**：说明不是"用得多"（那只会涨 heapUsed），是堆本身在膨胀——对象没被释放；
- **涨速和 QPS 正相关**：凌晨低峰期几乎不涨，早高峰一来就往上窜。指向"每次请求泄漏一点点"；
- **重启立即好**：进程重建，堆清空，一切归零——这也是"重启大法"能掩盖它半年的原因。

慢性泄漏最坑的地方在于**它永远在"还行"的区间里**：监控阈值设的是"内存 > 80% 告警"，而泄漏让内存在 40%→80% 之间缓慢爬，告警响起时离 OOM 只剩几小时。等你把阈值调低，又会被正常的流量毛刺打扰。这类问题，靠阈值监控是抓不住的，得靠**趋势**——"heapUsed 的 7 天斜率是否为正"，比"当前值"有用得多。

## 第一步：先证明是泄漏，不是 GC 偷懒

V8 的 GC 是懒惰的：堆里有空闲时，它没动力全量回收。所以"内存涨了"不等于"泄漏了"。排查的第一步永远是排除法：

```bash
node --expose-gc app.js
```

然后在代码里定时手动触发：

```javascript
setInterval(() => {
  global.gc();
  const used = process.memoryUsage().heapUsed / 1024 / 1024;
  console.log(`heapUsed after full GC: ${used.toFixed(1)} MB`);
}, 60000);
```

手动 full GC 后 heapUsed 还在单调涨——**泄漏实锤**。如果手动 GC 后能回落，那就是"GC 懒 + 堆配置小"，调 `--max-old-space-size` 或者接受它。我们那次，手动 GC 后曲线照涨不误，进入下一步。

## 第二步：heapdump 双快照，找"# New 最多"的类型

```javascript
const heapdump = require('heapdump');

// baseline：服务刚启动、预热完成时
heapdump.writeSnapshot('/tmp/base.heapsnapshot');

// 压测 / 跑 10 分钟业务流量后
heapdump.writeSnapshot('/tmp/leak.heapsnapshot');
```

用 Chrome DevTools（Memory 面板）打开两个快照，切到 **Comparison** 视图，按 `# New`（新增对象数）排序。泄漏的对象类型会排在最前面——因为它们是"只增不减"的。

我们那次的结果：第一名是 `Timeout` 对象，第二名是一堆闭包引用的 `IncomingMessage`（也就是 `req`）。看到 `Timeout` 排第一时我愣了一下：timer 对象本身才几十字节，怎么会是泄漏元凶？**这是 heapdump 新手最容易误判的地方**：要看的不是 Shallow Size（对象自身大小），是 **Retained Size（保留大小）**——"如果这个对象消失，能连带释放多少内存"。一个 100 字节的 timer，通过闭包引用链，可能 retain 着 10MB 的请求上下文。点开 retainers 链一层层展开，真相就出来了。

## 第三步：真凶——30 秒的 setTimeout 钉住了整个请求上下文

顺着 retainers 链往上找，定位到一段"祖传"中间件：

```javascript
// 问题代码：一个"慢请求告警"中间件
app.use((req, res, next) => {
  const timer = setTimeout(() => {
    logger.warn('slow request', req.url);  // req 被闭包捕获
  }, 30000);
  // res 正常返回了，但 timer 没清！
  next();
});
```

逻辑是：如果 30 秒后请求还没结束，打一条慢日志。问题在于：**正常请求 200ms 就返回了，但这个 30 秒的 timer 还在事件循环里躺着**，而它的回调闭包捕获了 `req`——`req` 又引用着 `res`、中间件链、路由上下文、甚至 body parser 塞进来的解析结果。整个对象图，在这 30 秒里一个字节都释放不了。

高 QPS 下算笔账：每秒 500 请求 × 30 秒 = **任意时刻有 15000 个"已经结束"的请求上下文被 timer 钉在内存里**。每个上下文几十 KB，就是几百 MB。这完美解释了"QPS 越高涨得越快"——泄漏速率 = QPS × 单请求残留 × 残留时长。

修复是两行：

```javascript
app.use((req, res, next) => {
  const timer = setTimeout(() => {
    logger.warn('slow request', req.url);
  }, 30000);
  timer.unref();                              // 不让 timer 阻止进程退出
  res.on('finish', () => clearTimeout(timer)); // 响应结束立刻清理
  next();
});
```

`unref()` 让 timer 不再阻止事件循环退出（顺手治了"优雅停机卡住"的毛病），`res.on('finish')` 保证响应一结束 timer 就被清掉——**创建 timer 的地方，必须同时写好它的葬礼**。

## 更深一层：Node 泄漏的"惯犯名单"

这次是 timer，但 Node 内存泄漏的元凶高度集中，排查时可以按这个名单逐个过：

**1. Timer（setTimeout/setInterval）。** 惯犯之王。回调闭包捕获了什么？要不要 `unref()`？什么时候 `clear`？三问必须在写代码时回答，不是在排查时。

**2. EventEmitter 监听器。** `req.on('data')`、`emitter.on('x')` 注册了，响应结束/任务完成后有没有 `removeListener`？`MaxListenersExceededWarning` 一出现，基本就是泄漏信号。顺手提一句：`once` 不是免死金牌——触发前它一样占着引用。

**3. 全局 Map/Set 缓存。** 无界增长的缓存就是泄漏。`cache.set(key, value)` 有没有配套的过期/淘汰？LRU 只是起点，key 的粒度失控（比如拿整个 URL 当 key 导致 key 空间无限）同样致命。

**4. 闭包捕获。** 比 timer 更隐蔽的是"长寿命函数引用了短寿命变量"：模块级数组里 push 了带闭包的函数、单例里存了请求级对象。heapdump 里看到 `system / Context` 类型异常多，就往这个方向查。

**5. 日志和监控的副作用。** 往日志对象里塞了整个 `req`？metrics 的 label 基数爆炸（拿 userId 当 label）？这两位是"看起来不像泄漏"的泄漏，heapdump 里经常以"一堆字符串"的面目出现。特别是结构化日志库为了"方便排查"默认序列化整个请求对象——一行 `logger.info({ req })`，可能就把整个上下文钉在了日志 buffer 里，等 buffer 刷盘才释放。高 QPS 下这就是缓慢的内存爬升，查的时候记得把日志采样打开看看塞了什么。

## 工具箱：heapdump 之外还有什么

双快照对比是定罪的"法庭"，但排查工具箱里还有几件趁手的兵器，各管一摊：

**1. clinic.js：傻瓜式体检。** `npx clinic doctor -- node app.js` 跑一遍，直接告诉你"是不是泄漏"以及"泄漏类型画像"（event loop 延迟、GC 频率、内存趋势一张图）。它的 `clinic heapprofiler` 还能做堆分配采样——**双快照告诉你"什么在涨"，采样告诉你"谁在分配"**，两者结合，泄漏代码的行号基本跑不掉。适合"我怀疑有泄漏但不知道从哪下手"的开局。

**2. --inspect + Chrome DevTools：活体解剖。** `node --inspect app.js`，Chrome 里连上去，Memory 面板可以对**运行中的进程**直接拍快照，不用改代码埋 heapdump。这对"生产环境不敢重启、不敢改代码"的场景是救命稻草——连上、拍照、断开，全程对业务零侵入。注意生产环境用 `node --inspect=127.0.0.1` 绑定本地，别把调试端口暴露出去。

**3. process.memoryUsage()：读懂四个数。** 很多人只看 heapUsed，其实四个字段各有含义：

```
rss         // 进程总内存（堆 + 栈 + native + 外部引用）
heapTotal   // V8 向 OS 申请的堆大小
heapUsed    // 实际使用的堆
external    // V8 堆外的 C++ 对象（Buffer 走这里！）
```

一个经典误判：`external` 涨而 `heapUsed` 不涨——这是 **Buffer/原生模块泄漏**，heapdump 里看不到（它们不在 V8 堆里）！文件上传、图片处理服务泄漏，先看 external。我们的排查清单里专门有一条："heapUsed 不涨但 rss 涨 → 查 external → 查 Buffer 没释放"。

**4. WeakRef 的正确用法（和误用）。** 缓存场景可以用 `WeakRef` 让缓存不阻止 GC：`cache.set(key, new WeakRef(value))`，取的时候 `ref.deref()` 可能已经没了。但注意两点：一是 WeakRef 不保证"及时"回收，别拿它做精确的资源管理；二是 `FinalizationRegistry` 的回调**不保证执行时机**，拿它做"清理文件句柄"这种强语义就是 bug。弱引用是"允许被回收"，不是"管理生命周期"。

工具选型的心法：**先 clinic 看画像，再 inspect 拍快照，最后双快照对比定罪**。三板斧下来，还没见过定不了罪的泄漏。

## 让这类 bug 进不了生产：四件事

复盘完，团队定了四条规矩，之后再没出现过同类事故。不说"最佳实践"这种虚的，直接说我们落地的：

**第一，timer 三问进 CR 红线。** 任何新增的 `setTimeout`/`setInterval`，CR 时必须回答：回调捕获了什么、要不要 unref、谁来 clear。答不上来打回。这条是血的教训换来的——泄漏代码的作者是三年前离职的同事，注释里只写了"慢请求告警"，没写"谁来清理"。

**第二，内存趋势告警，不是阈值告警。** `heapUsed` 的 24 小时斜率持续为正就告警（配合手动 GC 的探针排除 GC 懒惰）。阈值告警只能抓急性病，趋势告警才能抓慢性病。实现上很简单：Prometheus 里对 `nodejs_heap_size_used_bytes` 做 `deriv()`，斜率连续 6 小时为正就发 warning。我们上线这条规则的第一个月，它就在一次发版后 40 分钟抓到了一个回归泄漏——比用户先发现，比阈值告警早了整整两天。

**第三，压测 + heapdump 进发版流程。** 每次发版前，预发环境跑 10 分钟压测，拉双快照对比，`# New` 异常增长的类型直接拦下。自动化脚本 30 行，拦住一次事故就回本。

**第四，删掉那个周三凌晨的重启 cron。** 定时重启是泄漏最好的共犯——它让"内存曲线"永远看起来健康，让团队对慢性泄漏彻底失明。删掉之后我们补了一条兜底：内存超过阈值才重启（治标），同时趋势告警保证"治本"的排查一定会发生。重启 cron 删掉的那天，我们在 wiki 上记了一句话：**"任何靠重启维持的服务，都是在用运维的勤奋掩盖代码的懒惰。"**

从"每周重启一次"到"连续跑三个月内存纹丝不动"，中间只差两行代码和一次认真的排查——以及删掉那个掩盖问题的定时重启 cron 的勇气。Node 的内存泄漏十有八九不是 V8 的问题，是"本该死掉的东西被意外引用"——排查公式就四步：**手动 GC 确认 → 双快照对比 → 按 Retained Size 找元凶 → 修引用链**。记住，timer 和 listener 是两大惯犯，创建它们的时候，就把葬礼一起写好。

##{"timestamp":1751169600}