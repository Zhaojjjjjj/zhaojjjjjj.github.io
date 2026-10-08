> import 和 import() 函数是 ES6 新增的两种语法，它们都用于在 JavaScript 中导入模块。虽然它们的作用相同，但它们的语法和用法有所不同
- import 是 ES6 中用于在静态环境中导入模块的关键字，它是在代码加载阶段执行的。import 的语法如下：
```javascript
import defaultExport from "module";
import * as name from "module";
import { export1 } from "module";
import { export1 as alias1 } from "module";
import { export1 , export2 } from "module";
import { export1 , export2 as alias2 , [...] } from "module";
import defaultExport, { export1 [ , [...] ] } from "module";
import defaultExport, * as name from "module";
```
- 通过 import 导入的模块，可以直接访问其导出的变量和函数。例如：
```javascript
import { foo } from './moduleA';
foo(); // 调用模块 A 导出的函数 foo
```
- import() 是 ES6 中用于在动态环境中导入模块的函数，它是在运行时执行的，而不是在代码加载阶段执行。import() 函数的语法如下：
```javascript
import(moduleName)
  .then((module) => {
    // 使用模块中的内容
  })
  .catch((error) => {
    // 处理错误
  });
```
- import() 函数返回一个 Promise，可以在 Promise 的 then 方法中使用导入的模块。与 import 不同，import() 可以动态地加载模块，即可以在运行时根据需要动态加载模块，而不需要在代码加载阶段就加载所有模块
- 每个文件中调用 import() 函数时，都会创建一个新的 Promise，这个 Promise 表示将要加载并解析对应的模块，因此每个文件都会获得一个不同的模块实例。但是，实际上整个应用程序只有一个 module.js 模块的实例
- 这是因为，当第一个文件调用 import() 函数时，会开始加载并解析 module.js 模块，当第二个文件调用 import() 函数时，由于该模块已经被加载并解析过了，因此不会重新加载并解析，而是直接返回已经加载并解析好的模块实例。这也意味着，如果在应用程序中多次调用 import() 函数来导入同一个模块，只有第一次调用会执行真正的加载和解析操作，后续调用都会直接返回缓存的模块实例
- 需要注意的是，由于 import() 是异步的，因此在模块加载完成之前，模块中导出的变量或函数是无法使用的。因此，在使用 import() 导入模块时，需要使用 Promise 或 async/await 等机制来处理异步操作

### Summary
1. import 和 import() 都是 ES6 中用于导入模块的语句，而 require() 则是 Node.js 中用于导入模块的函数
2. 使用 import 语句导入模块时，模块会被静态加载，也就是在编译时就已经确定了导入的模块;import() 和 require() 都是动态加载模块的方式。它们都允许在代码运行时根据需要加载模块，而不是在编译时就将所有模块都加载进来。不过两者的实现方式略有不同：import() 是基于 Promise 的异步加载，而 require() 是同步加载模块
3. 在整个应用程序中，使用 import 和 import() 语句导入的模块都是单例模式，也就是共用同一个模块实例，而使用 require() 导入的模块则会因为复制而产生多个实例
4. import 和 import() 语句支持模块的默认导出和命名导出，而 require() 只支持模块的默认导出 (module.exports) 