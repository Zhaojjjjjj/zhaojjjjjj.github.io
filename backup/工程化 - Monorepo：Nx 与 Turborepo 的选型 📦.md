> Monorepo 里 10 个包，`npm run build` 要跑 10 次，全量测试 30 分钟。Nx 和 Turborepo 是 2026 年的两个主流方案。这篇讲讲选型。

## Monorepo 的痛

```
packages/
  ui/（组件库）
  api/（后端）
  web/（前端）
  docs/（文档）

改了 ui 的一个按钮 → 全量 build + 全量 test = 30 分钟
```

**90% 的时间浪费在"没改的代码"上**。

## Turborepo：简单高效

```json
// turbo.json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],  // 先 build 依赖
      "outputs": ["dist/**"]    // 缓存产物
    },
    "test": {
      "dependsOn": ["build"]
    }
  }
}
```

```bash
# 只 build 改了的包
turbo run build --filter=...[HEAD]
```

**优点**：

1. **简单**：JSON 配置，10 分钟上手。
2. **缓存**：本地+远程缓存，**没改的代码直接复用**。
3. **Vercel 亲儿子**：和 Vercel 部署无缝集成。

**缺点**：功能相对少，大 monorepo（100+ 包）吃力。

## Nx：全能重器

```bash
# 只测受影响的
nx affected -t test --base=main

# 可视化依赖图
nx graph
```

**优点**：

1. **affected**：精准算出"改了啥→影响啥"，**只跑受影响的**。
2. **插件生态**：React、Node、Go……**50+ 官方插件**。
3. **大而全**：1000 个包的 monorepo 也扛得住。

**缺点**：学习曲线陡，配置复杂。

## 选型

|  | Turborepo | Nx |
|---|---|---|
| 上手 | 10 分钟 | 1 天 |
| 规模 | <50 包 | 50+ 包 |
| 缓存 | ✅ | ✅ |
| affected | 基础 | 强大 |
| 生态 | Vercel | 全 |

**经验法则**：

```
小团队（<20 人），JS 全栈 → Turborepo
大团队（50+ 人），多语言 → Nx
已经在用 → 别换，迁移成本 > 收益
```

## 实测效果

```
之前：全量 build 15 分钟，全量 test 20 分钟
之后（Turborepo + 远程缓存）：
  - 改 1 个包：build 1 分钟，test 2 分钟
  - PR 平均：5 分钟（缓存命中 80%）
```

**10 倍提速**，就靠"不重复造轮子"。

## 一句话总结

Monorepo 工具的本质是"**增量**"：只做改了的部分。Turborepo 简单，Nx 强大——**小用 turbo，大用 nx**。记住：工具是手段，目的是"**5 分钟内给反馈**"（见 2 月 CI 那篇）。选哪个不重要，重要的是**别全量跑**。

##{"timestamp":1787976000}