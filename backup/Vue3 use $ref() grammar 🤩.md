1. **INSTALL**
```
npm i @vue-macros/reactivity-transform
```
2. **CONFIG**
```javascript
vite.config.js

import ReactivityTransform from '@vue-macros/reactivity-transform/vite'; 
export default defineConfig({ 
       plugins: [vue({ refTransform: true }),  ReactivityTransform()],
})
```
```js
uniapp 单页面中不用引入 $ref
import { defineConfig } from 'vite';
import uni from '@dcloudio/vite-plugin-uni';

export default defineConfig({
	plugins: [
		uni({
			vueOptions: {
				reactivityTransform: true,
			},
		}),
	],
});
```
3. **IMPORT**
```javascript
<script setup>
import { $ref } from '@vue-macros/reactivity-transform/macros';
<script>
```
> [!TIP]
> EXAMPLE 
```javascript
let msg = ref('hello')
console.log(msg.value) // hello
 
let msg = $ref('hello')
console.log(msg) // hello
```
