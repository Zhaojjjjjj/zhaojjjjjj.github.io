> `npm uninstall lodash` 之后，项目居然还能跑——`require('lodash')` 没报错。删了包还能用，这就是"幽灵依赖"。这篇讲讲 npm 依赖地狱的经典坑。

## 症状

```
$ npm uninstall lodash
$ node -e "console.log(require('lodash').VERSION)"
4.17.21  // ？？？删了还能用？
```

## 根因：扁平化安装

npm 3+ 默认**扁平化** `node_modules`：

```
node_modules/
  lodash/          ← 你直接依赖的，删掉了
  some-lib/
    node_modules/  ← 没有！被提升到顶层了
  other-lib/       ← 依赖 lodash，但它的 lodash 被"提升"到顶层
```

`other-lib` 依赖 lodash，npm 把 lodash 提升到顶层。**你删的是"你的" lodash，但顶层的 lodash 还在**（因为 other-lib 还要用）。

更坑的：你的代码 `require('lodash')` 恰好能解析到顶层那个——**你从来没声明依赖它，却一直在用它**。这就是幽灵依赖。

## 为什么这是颗雷

1. **版本漂移**：顶层的 lodash 版本由 `other-lib` 决定，它一升级，你的代码可能就崩了——而你根本不知道你依赖它。
2. **换包管理器就爆**：pnpm 用硬链接+隔离，幽灵依赖直接 `MODULE_NOT_FOUND`。从 npm 切 pnpm 的团队，第一次安装必炸一堆。
3. **CI 玄学**：本地能跑，CI 挂了——因为 CI 是干净安装，依赖树解析顺序不同。

## 排查

```bash
# 谁在用 lodash？
npm ls lodash

# 输出：
# project@1.0.0
# ├── lodash@4.17.21 extraneous  ← 你的代码在用，但 package.json 里没有！
# └── other-lib@2.0.0
#     └── lodash@4.17.21
```

`extraneous` 就是幽灵依赖的标记。

## 根治

```json
// 1. package.json 里显式声明所有直接用的包
// 哪怕它现在能从顶层解析到
{
  "dependencies": {
    "lodash": "^4.17.21",
    "other-lib": "^2.0.0"
  }
}
```

```bash
# 2. 换 pnpm（严格隔离，幽灵依赖直接报错）
pnpm install

# 3. CI 加检查
npx depcheck  # 找出用了但没声明的依赖
```

## 幻影依赖（Phantom Dependency）：反方向的坑

还有个镜像问题：**package.json 里声明了，但代码里没用**。`depcheck` 能扫出来。危害小一些（主要是安装体积），但 monorepo 里很常见——复制粘贴 package.json 留下的遗产。

## 一句话总结

幽灵依赖的本质是"**用了没声明**"。npm 的扁平化是历史包袱，pnpm 的严格隔离是解药。记住两条铁律：**直接用的包必须显式声明；CI 跑 depcheck**。别让你的构建依赖运气。

##{"timestamp":1761710400}