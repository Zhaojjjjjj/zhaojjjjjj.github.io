> 周五下午五点半，运维在群里 @ 全体："节点磁盘告警，谁的镜像 2.1GB？Pod 调度不上去了，自己认领。"群里安静了三秒，然后齐刷刷 @ 我。

# Docker 瘦身：2.1GB 到 180MB 的实战记录

周五下午五点半，运维在群里 @ 全体："节点磁盘告警，谁的镜像 2.1GB？Pod 调度不上去了，自己认领。"群里安静了三秒，然后齐刷刷 @ 我。

那是我们的 Node 服务镜像。CI 构建 15 分钟，部署传镜像 8 分钟，回滚一次等于小型发布。那晚我花三小时把它从 2.1GB 瘦到 180MB。下面是完整记录，全是踩出来的坑。

# 一、先诊断：2GB 里到底装了什么

动手前先看体积花在哪：

```bash
$ docker history myapp --human --format "{{.CreatedBy}}\t{{.Size}}" \
    | sort -rk2 -h | head -5
# 再装个 dive，一层层看文件树和"文件效率"评分
$ dive myapp:latest
```

典型发现四类：**基础镜像 1GB+**（`node:20` 光底子就 1 个多 G）；**devDependencies 进生产**（typescript、vite、@types 全躺着）；**本地 `node_modules` 被 COPY 进去**（macOS 编译的 native 模块在 linux 容器里直接崩）；**构建缓存和源码**（npm 缓存、`.git`、覆盖率报告）。

**wasted space 超过 20%，说明层顺序有大问题**——先修顺序，再谈换底子。诊断顺序：先看 wasted（层顺序），再看基础镜像（底子），最后看依赖（内容）。结论只有一句：**生产镜像里 90% 的东西，生产环境根本用不上。**

# 二、多阶段构建：构建工具不进生产

瘦身效果最大的一招：

```dockerfile
# 阶段一：构建（臃肿无所谓）
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 阶段二：运行（干净，只拿结果）
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

typescript、vite、@types 全留在 builder 阶段，镜像直接砍 60%。配合三处细节：`.dockerignore` 挡住 `node_modules`、`.git`、`.env`（**它的地位和 `.gitignore` 一样，不写就是裸奔**）；COPY 顺序把 `package*.json` 放前面（`npm ci` 的层缓存别浪费——**构建从 15 分钟降到 2 分钟就靠这一招**）；BuildKit 缓存挂载让 npm 缓存加速构建但不进镜像层。

基础镜像选型看依赖链不看流行度：`node:20-alpine`（musl，通用首选）→ `node:20-slim`（glibc，native 模块多时更稳）→ `distroless`（连 shell 都没有，**攻击面最小化**，只适合构建期充分验证的服务）。

# 三、理解"层"是不可变的：rm 掉文件，镜像不会变小

最多人误解的一点：

```dockerfile
# 自欺欺人：删除发生在新层，旧层的文件还躺着
RUN npm ci
RUN rm -rf /root/.npm

# 正确：同一层里产生垃圾、清理垃圾，不留痕
RUN npm ci && rm -rf /root/.npm
```

镜像层是不可变的叠加，每层都是前一层的 diff。第 5 层删掉的东西，第 3 层里还在，体积一点没少。**产生临时文件的操作，必须在同一条 RUN 里清理干净。** 同理 apt 标准写法：`apt-get update && apt-get install -y --no-install-recommends curl && rm -rf /var/lib/apt/lists/*` 一层之内装完清完。

# 四、深挖：体积大，真正的代价不是"占地方"

瘦到 180MB 后复盘：2GB 真正的代价，体积本身反而最小。

**部署速度**：2.1GB 内网传输 8 分钟——紧急回滚时，这 8 分钟就是故障时长。180MB 40 秒传完，**回滚从"小型发布"变成"一键操作"**。

**攻击面**：`trivy` 一扫就知道——旧镜像 127 个 CVE，新镜像 9 个。你用不上的 `git`、`curl`、`python`，黑客可能正好用得上。**瘦身是最便宜的安全加固。**

**扩容速度**：流量突增扩 20 个 Pod，每个节点都要拉镜像。在弹性架构里，**镜像体积直接决定扩容速度**。

还有个被忽略的红利：**分层传输**。`docker pull` 只拉变化的层——瘦身后日常部署传的不是 180MB，而是"代码层"那几 MB。但前提是层要"稳定"：基础层、依赖层常年不变，只有顶层代码层天天变。**把不常变的放底层，是构建加速和部署加速的同一招。** 反例：`COPY . .` 放 `npm ci` 前面，代码一改依赖层缓存失效，构建部署双输。

# 结语：30 分钟体检清单

> **生产镜像里只放"运行必需"的东西。**

下周一花 30 分钟，按这个顺序：`docker history` 找出最大的 3 层；加 `.dockerignore`；换基础镜像（alpine 或 distroless）；拆多阶段构建；调 COPY 顺序保层缓存；合并 RUN 并同层清理；用 `dive` 和 `trivy` 验证——体积降了，CVE 也降了，才算真瘦下来。

以及：给镜像体积设一条 CI 红线（超 300MB 告警）。很多团队的镜像"瘦一次、胖回去"——没有门禁，三个月后又 2GB，之前的三小时白干。改造按风险排序逐步收紧：`.dockerignore`+换底子（低风险）→ 多阶段（中风险）→ distroless（高风险），**每一步单独验证、单独上线**。

##{"timestamp":1766980800}