#### 1. aspect-ratio
**描述**: 用于定义元素的宽高比，简化了以往使用“填充黑客”的方法。只需指定一个比率，浏览器会自动调整元素的尺寸
```css
.aspect-ratio-hd {
    aspect-ratio: 16/9; 
}
.aspect-ratio-square {
    aspect-ratio: 1; /* 正方形 */
}
```

#### 2. object-fit
**描述**: 控制替换元素（如图像）的内容如何适应其容器，常用值包括 `cover` 和 `scale-down`
```css
.image {
    object-fit: cover; /* 图像覆盖容器 */
    aspect-ratio: 1; /* 可选：约束图像大小 */
    max-block-size: 250px; /* 最大高度 */
}
```

#### 3. margin-inline
**描述**: 用于设置元素的左右边距，简化了以往的 margin-left 和 margin-right 设置
```css
.container {
    margin-inline: auto; /* 水平居中 */
}
```

#### 4. text-underline-offset
**描述**: 控制文本基线与下划线之间的距离，改善可读性
```css
a:not([class]) {
    text-underline-offset: 0.25em; /* 下划线偏移量 */
}
```

#### 5. outline-offset
**描述**: 设置轮廓与元素之间的距离，提升焦点可见性
```css
.outline-offset {
    outline: 2px dashed blue; /* 蓝色虚线轮廓 */
    outline-offset: .5em; /* 偏移量 */
}
```

#### 6. scroll-margin-top/bottom
**描述**: 在滚动到锚点时增加偏移量，避免导航条覆盖内容
```css
[id] {
    scroll-margin-top: 2rem; /* 向上增加空间 */
}
```

#### 7. color-scheme
**描述**: 定义浏览器 UI 元素（如表单控件和滚动条）的颜色方案，以适应深色或浅色主题
```css
:root {
    color-scheme: dark light; /* 优先使用深色主题 */
}
```

#### 8. accent-color
**描述**: 修改复选框和单选按钮等表单控件的颜色，提高主题管理能力
```css
:root {
    accent-color: mediumvioletred; /* 自定义控件颜色 */
}
```

#### 9. width: fit-content
**描述**: 自动调整元素宽度以适应内容，类似于 `inline-block`
```css
.fit-content {
    width: fit-content; /* 根据内容调整宽度 */
}
```

#### 10. overscroll-behavior
**描述**: 控制滚动区域的行为，防止滚动超出边界影响父页面
```css
.sidebar, .article {
    overscroll-behavior: contain; /* 限制滚动行为 */
}
```

#### 11. text-wrap
**描述**: 改善文本排版，防止孤立词和不平衡行
```css
:is(h1, h2, h3, h4, .text-balance) {
    text-wrap: balance; /* 平衡文本行 */
    max-inline-size: 25ch;
}
```
 
#### 12. scrollbar-gutter
**描述**: 为滚动条预留空间，避免因滚动条出现或消失而导致布局偏移
```css
.container {
    scrollbar-gutter: stable; /* 保持滚动条空间 */
}
```