> Tailwind 4 把配置从 JS 搬进 CSS。这不是语法糖，是"配置应该向标准靠拢"的结论。

先看这个改动到底是什么：

```javascript
// Tailwind 3：配置是 JS 对象
module.exports = {
  theme: { extend: { colors: { brand: '#7C3AED' } } }
}
```

```css
/* Tailwind 4：配置是 CSS */
@import "tailwindcss";
@theme {
  --color-brand: #7C3AED;
  --spacing-18: 4.5rem;
}
```

同样是定义一个品牌色，从"JS 对象"变成了"CSS 变量"。为什么？三个原因，一个比一个根本。

# 一、级联：CSS 变量天生会"继承"

JS 配置的 `extend` 是**合并对象**，CSS 变量的覆盖是**级联**：

```css
/* 基础主题 */
@theme { --color-brand: #7C3AED; }

/* 暗黑模式：直接覆盖，不用 extend、不用深合并 */
@media (prefers-color-scheme: dark) {
  @theme { --color-brand: #A78BFA; }
}
```

JS 里做同样的事，要写条件逻辑、深合并、担心 key 冲突。CSS 里，**覆盖就是语言的原生语义**。Tailwind 4 的选择是：别再用 JS 重新发明 CSS 已经解决的问题。

# 二、工具链：干掉 JS 运行时

Tailwind 3 的构建流程：

```
tailwind.config.js → Node.js 执行 → 生成 CSS
```

Tailwind 4（Oxide 引擎，Rust 重写）：

```
CSS 输入 → Rust 直接解析 @theme → 生成 CSS
```

**配置不再需要 JS 运行时**。构建更快（Rust 解析 CSS 是微秒级），而且**配置和源码用同一种语言**——设计师看得懂 `@theme`，看不懂 `module.exports`。这降低了设计系统和代码之间的翻译成本。

# 三、标准：学一次，终身用

`@theme` 定义的是** CSS 自定义属性**（Custom Properties），这是 Web 标准：

```css
/* Tailwind 4 生成的，本质上是这个 */
:root { --color-brand: #7C3AED; }
.brand-text { color: var(--color-brand); }
```

这意味着：**即使你明天不用 Tailwind 了，这些变量还能用**。而 `tailwind.config.js` 是框架私有的——换框架，配置作废。Tailwind 4 在主动**降低自己的锁定**，换取的是"标准"的合法性。这是个聪明的长期赌注：框架会过时，标准不会。

# 四、争议："class 字符串太长"是个伪问题吗

```html
<div class="flex items-center justify-between px-4 py-2 bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow">
```

批评者说 HTML 臃肿。但算一笔账：

```
传统 CSS：.card { ...50 行样式... } × 100 种卡片 = 5000 行 CSS
Tailwind：HTML 里 100 个长 class 字符串，CSS 只有用到的原子类（几 KB）
```

**臃肿从 CSS 搬到了 HTML，但总量变小了**——因为原子类的复用度极高。而且 gzip 对重复的 class 字符串极其友好（`flex items-center` 出现 1000 次，压缩后几乎不占体积）。

真正的解法也不是"忍受"，而是 `@utility`：重复的模式抽成自定义工具类，**在原子和组件之间找到中间层**。

# 结语

Tailwind 4.3 的 CSS-first，可以浓缩成一句话：

> **样式的未来是"描述"，不是"命名"——而描述的语言，应该是标准 CSS，不是框架方言。**

2026 年的前端样式分工正在定型：Tailwind 做原子层，CSS 变量做主题层，组件库做模式层。三层各干各的，互不打架。而 `@theme` 的真正意义，是让"设计 token"第一次有了**框架无关**的载体——设计系统终于可以独立于技术选型而存在。

##{"timestamp":1780027200}