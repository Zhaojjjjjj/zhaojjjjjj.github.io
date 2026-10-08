- 自动引入 ref reactive...
- npm i unplugin-auto-import -D
```
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import AutoImport from 'unplugin-auto-import/vite';
 
export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      // 自动导入 Vue 相关函数，如：ref, reactive ...
      imports: ['vue', 'vue-router'],
    }),
  ],
});
```


- 使用$ref()
- npm i -D unplugin-vue-macros
```
import vue from '@vitejs/plugin-vue';
import AutoImport from 'unplugin-auto-import/vite';
import VueMacros from 'unplugin-vue-macros/vite';
import { defineConfig } from 'vite';

export default defineConfig({
    plugins: [
        AutoImport({
            imports: ['vue', 'vue-router'],
        }),
        VueMacros({
            plugins: {
                vue: vue(),
            },
        }),
    ],
});
```


- prettier自动排列 Tailwind 的类名顺序
- npm install -D prettier prettier-plugin-tailwindcss
```
// .prettierrc
{
  "plugins": ["prettier-plugin-tailwindcss"]
}
```


- 配置 import 的自动排列
- npm install --save-dev @trivago/prettier-plugin-sort-imports
```
.prettierrc
{
    "printWidth": 400,
    "tabWidth": 4,
    "useTabs": false,
    "semi": true,
    "bracketSameLine": true,
    "quoteProps": "preserve",
    "trailingComma": "all",
    "singleQuote": true,
    "importOrderSeparation": true,
    "importOrderSortSpecifiers": true,
    "plugins": ["@trivago/prettier-plugin-sort-imports"],
    "importOrder": ["^@core/(.*)$", "<THIRD_PARTY_MODULES>", "^@ui/(.*)$", "^[./]"]
}
```