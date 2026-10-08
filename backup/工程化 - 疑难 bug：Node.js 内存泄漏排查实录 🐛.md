> Node.js 服务跑着跑着，内存从 200MB 涨到 2GB，然后被 OOM killer 干掉。重启就好，过几小时又来。这篇复盘一次真实的内存泄漏排查，工具链和方法论都讲透。

## 症状

- RSS 持续增长，GC 后不回落。
- heapUsed 涨，heapTotal 跟着涨——说明不是"用得多"，是"漏了"。
- 有趣的是：QPS 越高，涨得越快。指向"每次请求泄漏一点点"。

## 排查四步

### 1. 确认是泄漏，不是正常增长

```bash
# 看 GC 表现
node --expose-gc app.js
# 然后在代码里定时：
global.gc();
console.log(process.memoryUsage().heapUsed / 1024 / 1024);
```

手动 GC 后 heapUsed 还在涨→真泄漏。先排除"只是 GC 懒"。

### 2. heapdump 对比

```javascript
const heapdump = require('heapdump');
//  baseline：服务刚启动
heapdump.writeSnapshot('/tmp/base.heapsnapshot');
// 压测 10 分钟后
heapdump.writeSnapshot('/tmp/leak.heapsnapshot');
```

Chrome DevTools 打开两个 snapshot，用 **Comparison** 视图，按 `# New` 排序。泄漏的对象类型会排在最前面。

### 3. 找到元凶：闭包里的请求上下文

我们的 case 里，`# New` 第一名是 `Timeout` 对象，第二名是闭包引用的 `req` 对象。

```javascript
// 问题代码
app.use((req, res, next) => {
  const timer = setTimeout(() => {
    logger.warn('slow request', req.url);  // req 被闭包捕获
  }, 30000);
  // res 正常返回了，但 timer 没清！
  next();
});
```

**30 秒的 setTimeout 持有 req 引用**。正常请求 200ms 就返回了，但 timer 还在跑 30 秒——这 30 秒内，req、res、整个中间件链的对象图都释放不了。高 QPS 下，每秒几百个 req 被 timer"钉"在内存里。

### 4. 修复

```javascript
app.use((req, res, next) => {
  const timer = setTimeout(() => {
    logger.warn('slow request', req.url);
  }, 30000);
  timer.unref();  // 关键：不让 timer 阻止进程退出和 GC
  res.on('finish', () => clearTimeout(timer));  // 响应结束就清掉
  next();
});
```

两行修复：`unref()` + `res.on('finish')` 清理。

## 更深一层：为什么 heapdump 第一名是 Timeout

V8 的 heap snapshot 里，`Timeout` 对象本身很小，但它通过 `_onTimeout` 闭包引用了整个作用域链。**泄漏分析要看"保留大小"（Retained Size），不是"自身大小"（Shallow Size）**。一个 100 字节的 timer，可能 retain 着 10MB 的请求上下文。

## 防患清单

1. **所有 setTimeout/setInterval 问自己**：回调里引用了什么？需不需要 unref？什么时候 clear？
2. **EventEmitter 监听器**：`req.on('data')` 之类，响应结束后有没有 removeListener？（`emitter.setMaxListeners` 爆了就是信号）
3. **全局 Map/Set 缓存**：有没有过期清理？无界增长的缓存就是泄漏。
4. **压测 + heapdump 进 CI**：每次发版前对比 snapshot，`# New` 异常增长就拦下。

## 一句话总结

Node 内存泄漏十有八九是"本该死掉的东西被意外引用"。排查公式：**手动 GC 确认 → heapdump 双快照对比 → 按 Retained Size 找元凶 → 修引用链**。记住，timer 和 listener 是两大惯犯，写的时候就想好"谁来清理它"。

##{"timestamp":1751169600}