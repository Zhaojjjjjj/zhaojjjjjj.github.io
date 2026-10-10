> useMemo 是 React 社区最分裂的 API：人人都在写，但 React 团队认为大部分手写记忆化都不需要你写。Compiler 把这句话变成了现实。

# React Compiler：记忆化从"手艺"变成"基础设施"

`useMemo` 可能是 React 社区最分裂的一个 API：面试必考、CR 必问、人人都在用——但 React 团队自己的态度是，大部分手写的 `useMemo` 其实不需要你写。React Compiler 1.0 GA，把这句话变成了现实。

问题是：**凭什么一个编译器敢替你决定"什么该缓存"？** 它怎么证明自己不会把依赖写错——而这正是人类最容易写错的地方。

# 一、它解决的是"人"的问题，不是"性能"问题

看一个典型组件：

```jsx
// 记忆化之前的世界：到处是防御性缓存
function ProductList({ products, filter }) {
  const visible = useMemo(
    () => products.filter(p => p.category === filter),
    [products, filter]
  );
  return <List data={visible} />;
}
```

这段代码有三个问题，全是"人"的问题：**该不该加靠猜**（不确定，先加上——防御性 `useMemo` 就是这么来的）；**依赖数组靠手写**（漏了 `filter`，闭包拿到旧值，bug 藏得极深）；**阅读成本**（业务逻辑两行，缓存样板八行）。

Compiler 的思路：**既然依赖推导是机械劳动，就让机器做**。你写干净代码，编译器在构建时自动插入等价的缓存。

# 二、原理：构建时的数据流分析

Compiler 是一个 Babel/SWC 插件，对每个组件做三件事：

**分析**：数据流分析，找出"纯计算"——同样输入产生同样输出、无副作用的表达式。`products.filter(...)` 是纯的，可以缓存；`ref.current = x` 有副作用，不能动。

**切片**：把组件切成若干"响应式块"，每个块记录自己依赖了哪些值。决定缓存粒度：不是整个组件一个大缓存，而是每个计算独立缓存、独立失效。

**插入缓存**：自动生成等价于 `useMemo` 的代码，但依赖由编译器推导：

```jsx
// 你写的
function ProductList({ products, filter }) {
  const visible = products.filter(p => p.category === filter);
  return <List data={visible} />;
}

// 编译器生成的（示意）
function ProductList({ products, filter }) {
  const $ = useCache();          // 编译器管理的缓存槽位
  let visible;
  if ($[0] !== products || $[1] !== filter) {
    visible = products.filter(p => p.category === filter);
    $[0] = products; $[1] = filter; $[2] = visible;
  } else {
    visible = $[2];              // 命中缓存，直接复用
  }
  return <List data={visible} />;
}
```

和手写 `useMemo` 的本质区别：**缓存的正确性由编译器证明，而不是由你的记忆力保证**。

# 三、逃生舱：为什么这次成了，而 React Forget 没成

老玩家记得 2021 年的 "React Forget"——同样是自动记忆化编译器，后来没了下文。这次 Compiler 成了，三个变化：

**目标收敛**。Forget 想做的太多（连并发特性一起重构），Compiler 只干一件事：自动记忆化。**把 scope 砍到最小，是基础设施落地的第一课。**

**逃生舱哲学**。Forget 的思路是"编译一切"，搞不定就报错；Compiler 是"编译能编译的，搞不定的跳过，保持语义不变"。**从"全有或全无"到"渐进增强"**，这是它能进生产环境的关键。

**可观测性**。`eslint-plugin-react-compiler` 能逐个标出被跳过的组件和原因。编译器不是黑盒——**"为什么没优化我"有答案**，这是工程团队敢用的前提。

触发逃生的写法是明确的：渲染函数里读写 `ref.current`、条件调用 hook、直接操作 DOM、读模块级可变变量。**大部分逃生都是因为"渲染函数不纯"**——把副作用搬进 `useEffect`，组件就"可编译"了。这个过程本身就是一次代码质量清理。

# 四、心智模型转移：从"缓存"到"纯度"

Compiler 时代，性能心智变了：

```
以前："渲染很贵，能缓存就缓存" → 审依赖数组写全了吗
现在："保持渲染纯粹，让编译器工作" → 审组件可编译吗
```

CR 的 checklist 要换：第一，CI 跑 lint，被跳过的组件必须有理由——"暂时跳过优化"应该像 `// eslint-disable` 一样是需要解释的例外；第二，渲染函数纯吗——在渲染里读 `ref.current`、调 `Date.now()`，以前是坏味道，现在是"挡了编译器的路"；第三，**别再手写防御性缓存**——新的 `useMemo` 要多问一句：是语义需要，还是习惯性防御？

还有条必须分清的分界线：**`useMemo` 有两个用法，Compiler 只接管"为了快"的那种**。空依赖数组当"只初始化一次"用，是误用（官方从不保证缓存不被丢弃），该改还得改。记住：**凡是"为了快"写的，编译器接管；凡是"为了对"写的，你自己负责**。

# 结语

React Compiler 终结的不是 `useMemo` 这个 API，而是"每个前端都要懂记忆化"这个要求：

> **优化知识从"每个开发者的脑子"搬到了"构建工具"里。**

就像当年从手动内存管理到 GC——不是程序员变笨了，是基础设施把"正确但繁琐"的事接管了。新项目直接开 Compiler 别写 `useMemo`；老项目先跑通共存，再用 lint 报告逐个修成可编译，最后批量清理。删缓存是个"证明它不需要"的过程，别凭感觉。

##{"timestamp":1760500800}