凌晨 3 点，PagerDuty 把我叫醒：核心服务 P99 从 50ms 涨到 5s，健康检查开始超时，K8s 正在逐个重启 Pod。

我爬起来看监控：CPU 30%，内存 60%，网络正常——**经典的"什么都没满，但服务就是不行"**。重启 Pod 后恢复，6 点又犯。第三次，我决定不重启了，直接连上出问题的 Pod 开始查。

## 真凶：一个报表导出的同步循环

最后定位到的代码，是三个月前上线的一个"临时"报表导出功能：

```javascript
// 把 80 万行数据转成 CSV，一次性 stringify
app.get('/api/export', (req, res) => {
  const rows = db.query('SELECT * FROM events'); // 80 万行
  const csv = rows.map(r => toCSV(r)).join('\n'); // 同步，跑了 4 秒
  res.send(csv);
});
```

4 秒。**在这 4 秒里，这个 Node 进程什么都干不了**：新的 HTTP 请求进不来，健康检查回不上，定时器不触发。K8s 以为 Pod 死了，开始重启——重启期间流量打到其他 Pod，其他 Pod 也开始导出报表……**雪崩就是这么来的**。

测试环境为什么没发现？因为测试只有 1 万行数据，跑 50ms。**没有"很快"的同步代码，只有"数据量还不够大"的同步代码。**

## 为什么同步代码是 Node 的毒药

Node 是单线程事件循环。它的工作方式是：

```
┌─────────────┐
│  同步代码执行  │ ← 你在这里卡 4 秒
├─────────────┤
│  处理 I/O 回调 │ ← HTTP 请求、DB 返回，全在这里排队
├─────────────┤
│  处理定时器   │
├─────────────┤
│  setImmediate │
└─────────────┘
   ↑ 下一轮循环
```

**同步代码跑完，事件循环才能转下一圈。** 一个 200ms 的同步块，意味着这 200ms 内所有的网络响应、数据库回调、健康检查全部被延迟。这就是"Event Loop 饥饿"。

还有两个相关的坑，很多人分不清：

**process.nextTick 饥饿**：`nextTick` 的回调在"当前阶段结束前"全部执行完。如果你递归调用 `process.nextTick`，事件循环永远进不了下一轮——**比同步循环更隐蔽的饿死**。

```javascript
// ❌ 永远跑不完，事件循环被饿死
function foo() { process.nextTick(foo); }
```

**libuv 线程池的误解**：很多人以为"Node 的 I/O 都是异步线程做的"。错了一半——网络 I/O 是 epoll 真异步，但**文件 I/O、DNS、部分 crypto 走的是 libuv 线程池（默认 4 个线程）**。线程池被占满（比如 4 个大文件同时读），后面的文件操作照样排队。`UV_THREADPOOL_SIZE` 可以调，但治标不治本。

## 排查工具箱

### 1. 先确认：event loop lag 探针

```javascript
// 放到服务启动代码里，常驻监控
setInterval(() => {
  const start = Date.now();
  setImmediate(() => {
    const lag = Date.now() - start; // 正常 < 10ms
    metrics.gauge('eventloop.lag', lag);
    if (lag > 50) logger.warn('Event Loop 饥饿！lag =', lag, 'ms');
  });
}, 1000);
```

**lag 持续大于 50ms = 饥饿实锤。** 把这个指标接进 Prometheus，比事后查日志有用 10 倍。我们的事故里，如果早有这个指标，第一次抖动时就能发现。

### 2. 再定位：火焰图找"又宽又平"的块

```bash
# clinic.js：傻瓜式诊断
npx clinic doctor -- node app.js

# 或原生 cpu-prof
node --cpu-prof --cpu-prof-name=lag.cpuprofile app.js
```

在火焰图里找**又宽又平的同步块**——正常的 I/O 等待是"窄"的（不占 CPU），而同步计算是"宽"的。我们的真凶在火焰图里是一个 4 秒宽的 `JSON.stringify`，想认错都难。

### 3. 最小复现：灌生产级数据量

把可疑的同步操作单独拎出来，灌**生产级别**的数据量，计时。测试环境的 1 万行和生产的 80 万行，是两个世界。**能量化，才能定罪。**

## 三种修法，怎么选

| 方案 | 做法 | 适用 | 代价 |
|---|---|---|---|
| 切片 | 大循环拆 chunk，每 N 条 `await setImmediate` 让出 | 数据处理、批量任务 | 代码改动小，有吞吐损失 |
| worker_threads | CPU 密集型扔给 worker 线程 | 加密、压缩、大 JSON 解析 | 进程间通信开销，调试复杂 |
| 流式 | 边读边处理，不全量进内存 | 大文件、大结果集 | 重写逻辑，改动最大 |

我们的报表导出最后用了**流式**：数据库游标一批批读，CSV 一边拼一边往 response 写。内存占用从 2GB 降到 50MB，RT 从 4 秒降到首字节 100ms。

```javascript
// 修完之后的样子
app.get('/api/export', async (req, res) => {
  res.setHeader('Content-Type', 'text/csv');
  for await (const batch of db.queryStream('SELECT * FROM events', { batch: 1000 })) {
    res.write(batch.map(toCSV).join('\n') + '\n');
    await new Promise(r => setImmediate(r)); // 每批让出一次
  }
  res.end();
});
```

## 防患：让这类 bug 进不了生产

1. **监控**：`nodejs_eventloop_lag_seconds` 进 Prometheus，lag > 100ms 持续 1 分钟就告警；
2. **压测看 P99**：平均 RT 会掩盖饥饿，P99 不会。用生产级数据量压；
3. **CR 红线**：请求链路上的同步大循环、同步 crypto、大 JSON.parse/stringify，直接打回；
4. **混沌演练**：故意在预发环境放大同步块，看告警链路能不能在雪崩前 catch 住。

凌晨 3 点的教训很贵，但公式很便宜：**lag 指标确认 → 火焰图定位 → 切片/worker/流式修复**。以及永远记住——在 Node 里，同步代码的性能，要用生产数据量来衡量，测试环境的数据量都是"玩具"。

##{"timestamp":1776830400}