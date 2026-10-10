> 2026 年 5 月 8 日，Tailwind CSS 4.3 发布：CSS-first 配置成为主流，新增滚动条样式、`zoom`、`tab-size`。Tailwind 4 是 2024 年底的大重构，4.3 让它更完整。这篇聊聊 CSS-first 意味着什么。

## 从 JS 配置到 CSS-first

```javascript
// Tailwind 3：tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: { brand: '#7C3AED' },
      spacing: { 18: '4.5rem' },
    }
  }
}
```

```css
/* Tailwind 4.3：CSS 里直接写 */
@import "tailwindcss";

@theme {
  --color-brand: #7C3AED;
  --spacing-18: 4.5rem;
}
```

**配置从 JS 搬到 CSS**。理由：

1. **级联**：CSS 变量天然支持覆盖、继承，JS 配置要写 `extend`。
2. **工具链**：不需要 JS 运行时，**构建更快**（Rust 写的 Oxide 引擎）。
3. **标准**：`@theme` 用的是 CSS 原生语法，**学一次，终身用**。

## 4.3 的新东西

```css
/* 滚动条样式（终于不用 ::-webkit-scrollbar 手写了） */
@utility scrollbar-thin {
  scrollbar-width: thin;
}

/* zoom（替代 transform: scale，布局友好） */
.zoom-150 { zoom: 1.5; }

/* tab-size（代码块的制表符宽度） */
.tab-4 { tab-size: 4; }
```

## 为什么 Tailwind 赢了

```
2019：Bootstrap（组件库）
2021：Tailwind 崛起（原子化）
2024：Tailwind 4（Rust 引擎，CSS-first）
2026：Tailwind 是默认选项
```

**原子化 CSS 的胜利**：

1. **不用想类名**：`flex items-center gap-4`，描述即样式。
2. **不用清垃圾**：没用到的 class 不会进构建产物（tree-shaking）。
3. **设计系统**：`@theme` 就是设计 token，**设计和代码用同一套变量**。

## 争议：HTML 臃肿

```html
<!-- Tailwind -->
<div class="flex items-center justify-between px-4 py-2 bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow">

<!-- 传统 -->
<div class="card">
```

**class 字符串很长**。解法：

1. **@utility 抽组件**：重复的模式抽成自定义 utility。
2. **接受它**：HTML 长，但 CSS 小（复用度高）。**总包体积更小**。

## 一句话总结

Tailwind 4.3 的 CSS-first 是"**配置向标准靠拢**"。2026 年的前端样式方案：Tailwind 做原子层，CSS 变量做主题层，组件库做模式层。三层分工，互不打架。记住：**样式的未来是"描述"，不是"命名"**。

##{"timestamp":1780027200}