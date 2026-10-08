> React Compiler 1.0 在 2025 年 10 月 7 日 GA。它的口号很狂："忘记 useMemo、useCallback 吧"。这篇讲讲它到底是怎么做到的，以及能不能真的删掉那些 hooks。

## 它解决了什么

```jsx
// 之前：手动记忆化，到处是样板
const list = useMemo(() => items.filter(f), [items, f]);
const onClick = useCallback(() => do(x), [x]);

// 之后：直接写，编译器搞定
const list = items.filter(f);
const onClick = () => do(x);
```

React 团队统计：**80% 的 useMemo/useCallback 是"防御性"的**——开发者不确定要不要加，先加上再说。结果是代码里全是缓存样板，依赖数组写错还出 bug。

## 原理：构建期的自动记忆化

Compiler 是一个 Babel/SWC 插件，在构建时：

1. **分析**：对每个组件做数据流分析，找出"纯计算"（同样的输入→同样的输出）。
2. **切片**：把组件拆成"响应式块"，每个块追踪自己的依赖。
3. **插入缓存**：自动生成等价于 useMemo 的代码，但**依赖数组由编译器推导，不会写错**。

```jsx
// 你写的
function App({items}) {
  const list = items.filter(x => x.ok);
  return <List data={list} />;
}

// 编译器生成的（示意）
function App({items}) {
  const $ = useCache();
  let list;
  if ($[0] !== items) { list = items.filter(x => x.ok); $[0] = items; $[1] = list; }
  else { list = $[1]; }
  return <List data={list} />;
}
```

## 边界：不是所有代码都能编译

Compiler 有"逃生舱"：遇到搞不定的代码（比如直接操作 ref、某些副作用模式），就**跳过优化**，保持原语义。`useMemo`/`useCallback` 手写的也不会被删——它们继续work。

但这意味着：**新老代码的性能模型不一样了**。被编译的组件自动优化，没被编译的还靠手写——迁移期要当心。

## 能删 useMemo 了吗

结论：**新项目别写了，老项目别急着删**。

- 新项目：开 Compiler，直接写干净代码。
- 老项目：useMemo 留着，Compiler 会和它们共存。等哪天全量编译通过了，再批量清理。
- **useMemo 的语义陷阱还在**：有人用 useMemo 做"只跑一次"的语义（空依赖数组），Compiler 不会改变这个行为，但别依赖它——那是误用。

## 一句话总结

React Compiler 是"把最佳实践编译进去"。它终结的不是 useMemo 这个 API，是"手动性能优化"这个工种。前端性能优化正在从"手艺"变成"基础设施"——而这，才是 1.0 GA 真正的意义。

##{"timestamp":1760500800}