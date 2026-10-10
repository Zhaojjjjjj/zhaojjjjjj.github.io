> POST 还没发就死了——死在 OPTIONS 预检这一步。CORS 报错 90% 是响应头没发对，不是逻辑错了。

先说预检为什么存在。浏览器的同源策略是：**默认不允许跨域读响应**。但有些跨域请求是"安全"的（简单的 GET），有些是"危险"的（带自定义头的 PUT）。浏览器用一个规则区分：

```
简单请求：GET/POST + 特定 Content-Type → 直接发，赌一把
复杂请求：PUT/DELETE/自定义头/JSON → 先 OPTIONS 问"我能发吗"
```

**预检是浏览器的"敲门"**：先问服务器"我这个跨域请求行不行"，服务器说行，才发真的。注意这个顺序——**预检失败，真正的请求根本不会发**。所以看到 CORS 报错，先查 OPTIONS，别查 POST。

# 一、复现：用 curl 模拟预检

```bash
curl -X OPTIONS https://api.example.com/data \
  -H "Origin: https://web.example.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type,Authorization" \
  -v
```

看响应头，**必须有三个**：

```
Access-Control-Allow-Origin: https://web.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

缺哪个，补哪个。这是 CORS 排查的起点——**先离开浏览器，用 curl 看裸响应**，把"浏览器行为"和"服务器行为"分开。

# 二、Nginx 的 `always`：90% 的 403 死在这里

```nginx
location /api/ {
    if ($request_method = 'OPTIONS') {
        add_header Access-Control-Allow-Origin $http_origin always;
        add_header Access-Control-Allow-Methods 'GET, POST, PUT, DELETE, OPTIONS' always;
        add_header Access-Control-Allow-Headers 'Content-Type, Authorization' always;
        add_header Access-Control-Max-Age 86400 always;
        return 204;
    }
    proxy_pass http://backend;
}
```

**`always` 是整篇文章最重要的一个词。** Nginx 的 `add_header` 有个反直觉的行为：默认只在 2xx/3xx 响应上加头，4xx/5xx 上不加。而预检返回的是 204——**2xx，本该加的**。但如果你的 `location` 里有别的逻辑（比如 `proxy_pass` 报错、auth 失败），响应变成 4xx，CORS 头就消失了，浏览器看到的就是"没有 CORS 头"→ 403。

**`always` 强制在所有响应上加头**，不管状态码。90% 的"Nginx 配了 CORS 还是 403"，都是缺了 `always`。记住：**CORS 头不是"配了就行"，是"每次响应都必须有"**。

# 三、带 Cookie 的三重门

```javascript
// 前端：credentials: 'include'
fetch(url, { credentials: 'include' });
```

带 cookie 的跨域，要过三重门，**缺一不可**：

```
1. Access-Control-Allow-Origin: https://web.example.com  ← 不能是 *！
2. Access-Control-Allow-Credentials: true
3. 前端 credentials: 'include'
```

第一条是最常见的坑：**用 `*` 图省事，带 cookie 时浏览器直接拒绝**。这是规范定的，不是 bug——`*` 意味着"谁都能读"，和"带凭证"天生矛盾。Nginx 里用 `$http_origin` 变量回显请求来源，是标准解法。

# 四、代理链：每一层都可能吞头

```
浏览器 → CDN → Nginx → 后端
```

CORS 头是 HTTP 头，**经过的每一层都可能改写、缓存、丢弃**：

- CDN 缓存了"不带 CORS 头"的响应 → 后面所有请求都挂。解法：CDN 上配 `Vary: Origin`，让缓存按 Origin 区分；
- Nginx 的 `proxy_hide_header` 可能藏了后端的头；
- 后端框架的 CORS 中间件和 Nginx 的配置**重复加头** → 浏览器看到两个 `Allow-Origin` → 直接拒绝。

**排查原则：从外往里，一层层 curl**。先 curl CDN，看头在不在；再绕过 CDN 直接 curl Nginx；最后直连后端。头在哪一层消失的，问题就在哪一层。

# 结语：排查公式

```
POST 没发就死 → 先看 OPTIONS（curl 模拟）
  → 缺头？补三个 Allow 头
  → Nginx 配了还 403？加 always
  → 带 cookie？Origin 不能是 *
  → 经过 CDN/多层代理？逐层 curl，看头在哪消失
```

> **CORS 是浏览器的保护机制，不是后端的 bug。Postman 能通而浏览器不行，永远先查预检——而预检的问题，90% 是响应头没发对，不是逻辑写错了。**

##{"timestamp":1782705600}