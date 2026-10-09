> 2026 年 3 月 12 日，Vite 8.0 发布：打包器统一为 Rust 写的 Rolldown，构建 10-30 倍加速。这是前端工具链"Rust 化"的又一块拼图。这篇讲讲 Rolldown 到底快在哪。

## 前端工具的 Rust 化

```
esbuild（Go）：2019，打包快 100 倍
SWC（Rust）：2020，编译快 20 倍
Turbopack（Rust）：2022，Vercel 的新打包器
Rolldown（Rust）：2026，Vite 8 的默认打包器
Oxc（Rust）：parser/linter 全家桶
```

**JS 写的工具，正在被 Rust 重写**。Vite 8 是标志性事件：最主流的构建工具，彻底换了引擎。

## Rolldown 快在哪

1. **并行**：Rust 的 fearless concurrency，模块解析、transform、打包全并行。JS 是单线程，再怎么优化也追不上。
2. **内存**：零拷贝、arena 分配，大项目的内存占用是 Rollup 的 1/10。
3. **原生**：没有 Node.js 的启动和 IPC 开销，直接跑二进制。

```
实测（10 万模块的中型项目）：
Vite 7（Rollup）：45 秒
Vite 8（Rolldown）：3 秒
```

**15 倍**。这不是优化，是换代。

## 对开发者的影响

1. ** dev 更快了**：Vite 的 dev 本来就快（esbuild 预构建），build 现在也快了——**全链路无等待**。
2. **插件生态**：Rollup 插件大部分兼容（Rolldown 刻意保持 API 一致），但**用到了 Rollup 内部 API 的插件会挂**——升级前先查插件兼容表。
3. **Node 版本**：Vite 8 要求 Node 20.19+/22.12+，老项目先升级 Node。

## 迁移指南

```bash
# 1. 升级
npm i -D vite@8

# 2. 检查不兼容插件
npx vite --debug  # 看警告

# 3. 常见坑
# - vite-plugin-xxx 用了 this.resolve 的内部行为 → 换新版
# - 自定义 Rollup 插件用了 moduleParsed hook 的冷门字段 → 查文档
```

**大部分项目无痛升级**。Vite 团队做了 6 个月的生态兼容工作。

## 一句话总结

Vite 8 + Rolldown 是前端工程化的分水岭：**构建不再是瓶颈**。当 build 从 45 秒变成 3 秒，开发流程会变（更频繁的预览、更大的 monorepo）。记住：工具链的每一代加速，都会释放新的生产力——先升级，再想怎么用。

##{"timestamp":1774152000}