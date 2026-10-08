```
npm i unplugin-auto-import -D
```
- vite.config.js
```
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import AutoImport from 'unplugin-auto-import/vite';
 
export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      // auto import Vue function,example:ref, reactive ...
      imports: ['vue', 'vue-router'],
    }),
  ],
});
```
***

> Q : typescript ERROR：‘reactive’ is not defined.

> A :  Configure and generate auto-imports.d.ts in vite ,import in tsconfig.json 
- vite.config.js
```
export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
     // auto import Vue function,example:ref, reactive ...
      imports: ['vue'],
      // generate an automatically imported TS declaration file
       dts: "/auto-import.d.ts", 
    }),
  ],
});
```
- tsconfig.json
```
{
     "include": [
        "src/**/*.d.ts",
        "./*.d.ts",
        "./auto-imports.d.ts" // import the configuration file generated in the previous step
      ],
}
```
***

> Q: eslint error ‘reactive’ is not defined no-undef

> A: automatically import the corresponding eslint rules without configuration, generate the corresponding. eslintrc auto import. json configuration file through the configuration in autoimport, and introduce it in. eslintrc
- vite.config.js
```
{
    // ......
     AutoImport({
      imports: ["vue"],
      dts: "/auto-import.d.ts",
      eslintrc: {
        enabled: true,  // 1. Change to true for generating eslint configuration. 2. Change back to false after generation to avoid duplicate generation
      },
    }),
}
```
- .eslintrc.js
```
extends: [
   "./.eslintrc-auto-import.json",
 ],
```

