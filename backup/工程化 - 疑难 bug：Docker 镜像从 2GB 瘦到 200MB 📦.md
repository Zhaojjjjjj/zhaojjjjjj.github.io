> 一个 Node.js 应用的 Docker 镜像 2GB，CI 构建 15 分钟，部署传半天。这篇讲讲怎么把它瘦到 200MB，构建压到 2 分钟——全是实战经验。

## 症状

```bash
$ docker images
myapp   latest   2.1GB
```

## 瘦身五步

### 1. 换基础镜像：alpine 不是万能，但有用

```dockerfile
# 之前：node:20（1GB+）
FROM node:20
# 之后：node:20-alpine（200MB 不到）
FROM node:20-alpine
```

**注意**：alpine 用 musl 不是 glibc，有些 native 模块（如 sharp、bcrypt）要重新编译。先 `docker build` 试，别直接上生产。

### 2. 多阶段构建：构建工具不进生产镜像

```dockerfile
# 阶段一：构建
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build  # tsc、vite 都在这里

# 阶段二：运行（干净的）
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev  # 只装生产依赖
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

**效果**：typescript、vite、@types 这些构建期依赖，全部留在 builder 阶段。镜像直接砍掉 60%。

### 3. .dockerignore：别把垃圾 COPY 进去

```
node_modules
.git
.env
*.log
coverage
.DS_Store
```

**血泪教训**：有人把本地的 `node_modules`（macOS 编译的）COPY 进去，容器里直接崩。`.dockerignore` 是 Dockerfile 的第一行防线。

### 4. 层缓存：把不常变的放前面

```dockerfile
# ✅ 好：依赖层缓存，代码变了不用重装
COPY package*.json ./
RUN npm ci
COPY . .

# ❌ 差：代码一变，npm ci 重跑
COPY . .
RUN npm ci
```

Docker 按层缓存，`package.json` 没变就跳过 `npm ci`。**构建从 15 分钟降到 2 分钟**，就靠这一招。

### 5. 合并 RUN，减少层数

```dockerfile
# ❌ 3 层，每层都留痕
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# ✅ 1 层，临时文件不留
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```

## 效果对比

|  | 之前 | 之后 |
|---|---|---|
| 镜像体积 | 2.1GB | 180MB |
| 构建时间 | 15min | 2min |
| 部署传输 | 8min | 40s |

## 一句话总结

Docker 瘦身公式：**alpine 基础 + 多阶段构建 + .dockerignore + 层缓存 + 合并 RUN**。记住核心思想：**生产镜像里只放"运行必需"的东西**。构建工具、源码、git 历史，一个都别进。

##{"timestamp":1766980800}