> 做 AI 应用的流式输出，SSE（Server-Sent Events）是标配。但生产环境跑久了，你会遇到一个幽灵 bug：用户说"回答到一半卡住了"，后端日志却显示"正常返回"。这篇复盘这个经典坑。

## 症状

- 用户侧：流式输出到一半，突然不动了，也没有报错。
- 服务端：OpenAI/自建模型的 API 调用正常完成，200 返回。
- 诡异的是：刷新重试就好，低峰期不出现，高峰期偶发。

## 排查过程

第一反应是模型 API 的问题，加了重试——没用。第二反应是前端渲染，加了日志——发现 **EventSource 的 `onmessage` 在某个时间点后就再也没触发**，但 `onerror` 也没触发，`readyState` 还是 1（OPEN）。

这就是 SSE 最坑的地方：**连接"半死"时，浏览器不会告诉你**。

## 根因：三层超时

最后定位到是三层超时叠加：

1. **Nginx `proxy_read_timeout`**（默认 60s）：上游（模型 API）如果 60 秒内没有数据，Nginx 断开。但 SSE 是长连接，模型思考时（thinking 阶段）可能 60 秒零输出。
2. **浏览器自身的"静默超时"**：某些浏览器/代理对长时间无数据的连接会悄悄掐掉，不触发 error。
3. **EventSource 没有心跳机制**：原生 EventSource 不会自动发 ping，服务端不主动推数据，连接就是"死"的。

## 修复方案

四件套，缺一不可：

```nginx
# 1. Nginx：调大超时 + 关闭缓冲
proxy_read_timeout 300s;
proxy_buffering off;
X-Accel-Buffering: no;
```

```python
# 2. 服务端：定时发心跳注释（SSE 规范里以 : 开头的行是注释，浏览器忽略但会重置超时计时）
async def stream():
    while True:
        yield ": ping\n\n"
        await asyncio.sleep(15)
```

```javascript
// 3. 前端：用 fetch + ReadableStream 代替 EventSource
// EventSource 不支持自定义 header（带不了 Authorization），
// fetch 流可以，而且能自己实现超时检测
const reader = response.body.getReader();
const timer = setTimeout(() => controller.abort(), 30000); // 30s 无数据就重连
```

```javascript
// 4. 断点续传：用 Last-Event-ID
// 服务端给每个 chunk 带 id，前端重连时带上，服务端从断点继续
```

## 为什么用 fetch 代替 EventSource

2025 年的共识：**新项目别用 EventSource 了**。

- 不支持 POST（只能 GET，prompt 太长塞不下 URL）。
- 不支持自定义 header（Authorization 只能放 URL 参数，不安全）。
- 错误处理黑盒（就是本文这个坑）。
- `fetch` + `ReadableStream` + `TextDecoder` 组合，代码多 20 行，但可控性高一个维度。

## 一句话总结

SSE 断流的本质是"长连接在多层代理面前的脆弱性"。记住三板斧：**Nginx 关缓冲调超时、服务端发心跳、前端用 fetch 流+自研重连**。下次再遇到"到一半卡住"，先看 Nginx 日志的 499（客户端主动断开），八成就是它。

##{"timestamp":1748491200}