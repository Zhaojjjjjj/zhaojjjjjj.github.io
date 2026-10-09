> CI 跑一次 30 分钟，一天提 10 个 PR，流水线排队 2 小时。这篇讲讲 CI/CD 流水线的优化实战：从 30 分钟压到 5 分钟。

## 症状

```yaml
# .github/workflows/ci.yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci        # 3 分钟
      - run: npm run build # 5 分钟
      - run: npm test      # 20 分钟
      - run: npm run lint  # 2 分钟
# 总计：30 分钟
```

## 优化五式

### 1. 依赖缓存（最立竿见影）

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'npm'  # 一行，npm ci 从 3 分钟 → 20 秒
```

**原理**：`node_modules` 按 lockfile hash 缓存，命中直接解压。**所有 CI 的第一优化永远是缓存**。

### 2. 并行 job（把串行拆开）

```yaml
jobs:
  lint:    # 2 分钟，和其他并行
  build:   # 5 分钟
  test:    # 20 分钟
  # 总计：max(2, 5, 20) = 20 分钟，而不是 27 分钟
```

**再进一步**：测试分片（sharding）。

```yaml
strategy:
  matrix:
    shard: [1, 2, 3, 4]
run: npm test -- --shard=${{ matrix.shard }}/4
# 20 分钟 → 5 分钟（4 台机器并行）
```

### 3. 只跑受影响的（Nx / Turborepo）

```
PR 只改了 packages/ui → 只测 ui，不测 api、web
```

Monorepo 用 Nx 的 `affected` 命令，**90% 的 PR 只跑 10% 的测试**。

```bash
npx nx affected -t test --base=main
```

### 4. Docker 层缓存（见 12 月那篇）

```yaml
- uses: docker/build-push-action@v5
  with:
    cache-from: type=gha  # GitHub Actions 缓存
    cache-to: type=gha,mode=max
```

镜像构建从 8 分钟 → 1 分钟。

### 5. 合并检查（别每个 commit 都跑全量）

```yaml
# push 跑：lint + 相关测试（2 分钟）
# PR 合并前跑：全量（5 分钟）
# main 分支跑：全量 + e2e（10 分钟）
```

**分级 CI**：越接近合并，跑得越全。开发时快速反馈，合并时严格把关。

## 效果

| 优化 | 时间 |
|---|---|
| 原始 | 30 分钟 |
| + 缓存 | 22 分钟 |
| + 并行/分片 | 8 分钟 |
| + affected | 5 分钟（平均） |

## 一句话总结

CI 优化公式：**缓存依赖 + 并行分片 + 只测受影响 + 分级跑**。记住：CI 的目标不是"跑得多"，是"反馈快"。5 分钟内告诉开发者"挂了"，比 30 分钟后告诉他有用 10 倍。

##{"timestamp":1772251200}