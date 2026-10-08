> WebSocket 长连接跑久了，服务端内存缓慢上涨，客户端偶发"假在线"——看着连着，其实早断了。这篇复盘百万级连接下的两个经典坑：心跳和内存。

## 症状

- 服务端：连接数显示 100 万，但内存每周涨 10GB。
- 客户端：App 切后台再回来，显示"已连接"，但收不到推送——**假在线**。
- 诡异的是：重启服务端就好，说明状态在服务端堆积。

## 坑一：TCP 半开连接

```
客户端：App 被系统杀掉 / 网络切换（WiFi→4G）
服务端：完全不知道，连接还在 ESTABLISHED
```

TCP 的 keepalive 默认 2 小时才发第一个探测包——**2 小时内，服务端以为 100 万连接都活着**，其实一半已经死了。每个死连接持有：socket fd + 读缓冲 + 用户会话对象。

### 修复：应用层心跳

```javascript
// 服务端：30 秒 ping 一次，60 秒没回 pong 就掐
ws.on('pong', () => { conn.isAlive = true; });
const interval = setInterval(() => {
  wss.clients.forEach((ws) => {
    if (ws.isAlive === false) return ws.terminate();  // 掐掉假活的
    ws.isAlive = false;
    ws.ping();
  });
}, 30000);
```

关键：**用 `terminate()` 不用 `close()`**。`close()` 要走关闭握手，对端都死了，握手永远等不到——又是泄漏。`terminate()` 直接销毁 socket。

## 坑二：心跳风暴

修完坑一，上线后 CPU 飙到 90%。原因：**100 万连接同时心跳**。

```
30 秒一次 ping × 100 万连接 = 每秒 3.3 万次 ping
每次 ping 触发：定时器 + 网络 IO + 状态更新
```

### 修复：心跳分片 + 退避

```javascript
// 1. 把连接打散到 60 个桶，每秒只 ping 一个桶
const buckets = Array.from({length: 60}, () => new Set());
// 2. 移动端退避：App 在后台时，心跳间隔从 30s 降到 5min
//   （反正后台也收不到，不如省电）
if (client.isBackground) interval = 300000;
```

## 坑三：消息积压

断线重连后，服务端把离线期间的消息全推给客户端——如果用户离线 3 天，**重连瞬间收到 10 万条消息**，App 直接卡死。

```javascript
// 离线消息只保留最近 100 条 + 未读数
// 历史消息走 HTTP 分页拉，不走 WebSocket
const offline = await redis.lrange(`offline:${uid}`, 0, 99);
ws.send(JSON.stringify({type: 'sync', messages: offline, unread: total}));
```

**WebSocket 只做"实时"，不做"历史"**——这是铁律。

## 容量规划公式

```
内存 ≈ 连接数 × (socket 20KB + 会话 10KB + 缓冲 50KB)
100 万连接 ≈ 80GB

单机扛不住 → 按 userId 一致性哈希分片 → N 台机器
```

别指望单机百万连接，**先算账，再架构**。

## 一句话总结

WebSocket 的坑都在"连接不是非生即死"这个灰色地带：半开连接、心跳风暴、消息积压。记住三件套——**应用层心跳 + terminate 掐线 + 历史走 HTTP**。长连接做得好不好，看的不是"连上"，是"断得干净"。

##{"timestamp":1753761600}