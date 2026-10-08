## CODE

```html
<div></div>
```
```css
@property --_w {
  syntax: "<length>";
  inherits: true;
  initial-value: 100vw;
}
@property --_h {
  syntax: "<length>";
  inherits: true;
  initial-value: 100vh;
}
:root {
  --w: tan(atan2(var(--_w), 1px));
  --h: tan(atan2(var(--_h), 1px));
}

html,
body {
  height: 100%;
  box-sizing: border-box;
}

div {
  height: 100%;
  background: linear-gradient(
    125deg,
    #ecfcff 0%,
    #ecfcff 40%,
    #b2fcff calc(40% + 1px),
    #b2fcff 60%,
    #5edfff calc(60% + 1px),
    #5edfff 72%,
    #3e64ff calc(72% + 1px),
    #3e64ff 100%
  );
  &::before {
    content: counter(w) "x" counter(h);
    counter-reset: h var(--h) w var(--w);
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 12rem;
    mix-blend-mode: difference;
    white-space: nowrap;
    color: #fff;
    font-weight: 700;
    white-space: nowrap;
  }
}
```

## EXPLAIN

### `@property` 规则

```css
@property --_w {
    syntax: "<length>";
    inherits: true;
    initial-value: 100vw;
}
@property --_h {
    syntax: "<length>";
    inherits: true;
    initial-value: 100vh;
}
```

- `@property` 规则用于定义自定义属性的行为。在这里，`--_w` 和 `--_h` 是两个自定义属性。
- `syntax: "<length>"` 表示这些属性的值应该是长度单位（如 `px`、`vw`、`vh`）。
- `inherits: true` 表示这些属性的值会继承自父元素。
- `initial-value` 指定了这些属性的初始值，`--_w` 的初始值是 `100vw`（视口宽度的 100%），`--_h` 的初始值是 `100vh`（视口高度的 100%）。

### 自定义属性计算

```css
{
    --w: tan(atan2(var(--_w), 1px));
    --h: tan(atan2(var(--_h), 1px));
}
```

- `var(--_w)` 和 `var(--_h)` 来获取之前定义的自定义属性值。
- `atan2()` 和 `tan()` 是数学函数，目的是计算这些属性的某种变换值。
- `--w` 和 `--h` 是计算出来的值，涉及到三角函数的变换。

#### `div` 样式

```css
div {
    ......
    &::before {
        content: counter(w) "x" counter(h);
        counter-reset: h var(--h) w var(--w);
        mix-blend-mode: difference;
        ......
    }
}
```
  - `content: counter(w) "x" counter(h);` 显示了 `counter(w)` 和 `counter(h)` 的值，中间用 "x" 分隔。
  - `counter-reset: h var(--h) w var(--w);` 重置了计数器 `w` 和 `h` 的值，使用自定义属性的值。
  - `mix-blend-mode: difference` 使文字的颜色和背景颜色以差异模式混合。

