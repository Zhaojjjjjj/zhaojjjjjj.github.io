
#### 安装 Tailwind CSS
Tailwind CSS 要求 Node.js 版本至少为 12.13.0。

通过 npm 安装 Tailwind 及其对等依赖：

```bash
npm install -D tailwindcss@latest postcss@latest autoprefixer@latest
```

#### 创建配置文件
生成 `tailwind.config.js` 和 `postcss.config.js` 文件：

```bash
npx tailwindcss init -p
```

将会在项目根目录下创建一个基本的 `tailwind.config.js` 文件：

```javascript
// tailwind.config.js
module.exports = {
  purge: [],
  darkMode: false,
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

同时，也会创建一个包含 tailwindcss 和 autoprefixer 配置的 `postcss.config.js` 文件：

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

如果打算使用其他 PostCSS 插件，阅读相关文档以了解如何与 Tailwind 一起使用。

#### 配置 Tailwind 以在生产环境中移除未使用的样式
在 `tailwind.config.js` 文件中，配置 `purge` 选项以包含所有页面和组件的路径，这样 Tailwind 就可以在生产构建中移除未使用的样式：

```javascript
// tailwind.config.js
module.exports = {
  purge: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}'],
  darkMode: false, 
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

#### 在 CSS 中包含 Tailwind
创建 `./src/index.css` 文件，并使用 `@tailwind` 指令来包含 Tailwind 的基础、组件和工具样式，替换原始文件内容：

```css
/* ./src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Tailwind 将在构建时将这些指令替换为基于你配置的设计系统生成的所有样式。

#### 确保 CSS 文件被导入
确保你的 CSS 文件在 `./src/main.js` 文件中被导入：

```javascript
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'
import './index.css'

createApp(App).mount('#app')
```