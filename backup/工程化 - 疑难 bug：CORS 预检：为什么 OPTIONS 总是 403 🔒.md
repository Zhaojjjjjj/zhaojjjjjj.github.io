> 前端调 API，浏览器先发一个 OPTIONS 请求，然后报 403。POST 还没发就死了——CORS 预检失败。这篇是 CORS 的终极排查手册。

## 症状

```
浏览器：OPTIONS https://api.example.com/data → 403
       POST 还没发
控制台：Access to fetch blocked by CORS policy
```

## 先理解：为什么有预检

```
简单请求（GET/POST + 特定 Content-Type）：直接发
复杂请求（PUT/DELETE/自定义头/JSON）：先 OPTIONS 问"我能发吗"
```

**预检是浏览器的"敲门"**：先问服务器"我这个跨域请求行不行"，行才发真的。

## 排查四步

### 1. 看 OPTIONS 的响应头

```bash
curl -X OPTIONS https://api.example.com/data \
  -H "Origin: https://web.example.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type,Authorization" \
  -v
```

**必须有的三个头**：

```
Access-Control-Allow-Origin: https://web.example.com（不能是 *，如果带 cookie）
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

缺哪个，补哪个。

### 2. Nginx 配置（最常见）

```nginx
location /api/ {
    # 预检请求直接返回 204，不打到上游
    if ($request_method = 'OPTIONS') {
        add_header Access-Control-Allow-Origin $http_origin always;
        add_header Access-Control-Allow-Methods 'GET, POST, PUT, DELETE, OPTIONS' always;
        add_header Access-Control-Allow-Headers 'Content-Type, Authorization' always;
        add_header Access-Control-Max-Age 86400 always;  # 预检缓存 24 小时
        return 204;
    }
    proxy_pass http://backend;
}
```

**`always` 是关键**：Nginx 的 `add_header` 默认在错误响应上不加，`always` 强制加。**90% 的 CORS 403 是缺了 `always`**。

### 3. 带 Cookie 的坑

```javascript
// 前端
fetch(url, { credentials: 'include' });  // 带 cookie
```

```nginx
# 后端：Origin 不能是 *，必须是具体域名
Access-Control-Allow-Origin: https://web.example.com  # ✅
Access-Control-Allow-Origin: *  # ❌ 带 cookie 时浏览器拒绝
Access-Control-Allow-Credentials: true  # 必须有
```

### 4. 代理链的坑

```
浏览器 → CDN → Nginx → 后端
```

**每一层都要处理 CORS**，或者只在最外层处理。CDN 缓存了不带 CORS 头的响应 → 后面全挂。**CDN 上配 `Vary: Origin`**。

## 常见误区

| 误区 | 真相 |
|---|---|
| "后端加了 CORS 头就行" | Nginx/CDN 可能吞了，要 `always` |
| "用 * 最省事" | 带 cookie 时不行，必须具体域名 |
| "Postman 能调，浏览器不行" | Postman 不 enforce CORS，这是浏览器行为 |
| "OPTIONS 404" | 服务器没处理 OPTIONS，加路由 |

## 一句话总结

CORS 排查公式：**curl 模拟预检 → 看三个 Allow 头 → Nginx 加 always → 带 cookie 不用 ***。记住：CORS 是浏览器的保护，不是后端的 bug——**Postman 能通而浏览器不行，永远先查 CORS**。

##{"timestamp":1782705600}