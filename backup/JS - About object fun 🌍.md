#### 详细功能

- **`Object.keys(obj)`**：
  - **定义**：返回对象自身可枚举属性的键名数组。
  - **语法**：`Object.keys(obj)`
  - **示例**：
    ```javascript
    const person = { name: "Alice", age: 30 };
    console.log(Object.keys(person)); // 输出: ["name", "age"]
    ```
  - **用例**：遍历对象属性名，生成表单字段列表，或检查属性是否存在。
  - **限制**：仅返回可枚举属性，不包括原型链属性。

- **`Object.values(obj)`**：
  - **定义**：返回对象自身可枚举属性的值数组。
  - **语法**：`Object.values(obj)`
  - **示例**：
    ```javascript
    const person = { name: "Alice", age: 30 };
    console.log(Object.values(person)); // 输出: ["Alice", 30]
    ```
  - **用例**：处理值而不关心键名，如计算总和或找最大值。
  - **限制**：ES2017 引入，需注意浏览器兼容性。

- **`Object.entries(obj)`**：
  - **定义**：返回对象自身可枚举属性的键值对数组。
  - **语法**：`Object.entries(obj)`
  - **示例**：
    ```javascript
    const person = { name: "Alice", age: 30 };
    console.log(Object.entries(person)); // 输出: [["name", "Alice"], ["age", 30]]
    ```
  - **用例**：同时处理键和值，如按值排序或转换数据结构。
  - **限制**：ES2017 引入，需注意浏览器兼容性。

- **`Object.getOwnPropertyNames(obj)`**：
  - **定义**：返回对象所有属性名（包括不可枚举的）。
  - **示例**：
    ```javascript
    const obj = { a: 1 };
    Object.defineProperty(obj, 'b', { value: 2, enumerable: false });
    console.log(Object.keys(obj)); // 输出: ["a"]
    console.log(Object.getOwnPropertyNames(obj)); // 输出: ["a", "b"]
    ```
  - **用例**：调试或元编程场景，需访问不可枚举属性。

- **`Object.getOwnPropertySymbols(obj)`**：
  - **定义**：返回对象所有符号属性。
  - **示例**：
    ```javascript
    const obj = { a: 1 };
    const sym = Symbol('sym');
    obj[sym] = 'symbol value';
    console.log(Object.keys(obj)); // 输出: ["a"]
    console.log(Object.getOwnPropertySymbols(obj)); // 输出: [Symbol(sym)]
    ```
  - **用例**：处理符号键属性，常见于私有数据或避免命名冲突。

- **`Reflect.ownKeys(obj)`**：
  - **定义**：返回对象所有属性键（字符串和符号，包括不可枚举的）。
  - **示例**：
    ```javascript
    const obj = { a: 1 };
    const sym = Symbol('sym');
    Object.defineProperty(obj, 'b', { value: 2, enumerable: false });
    obj[sym] = 'symbol value';
    console.log(Reflect.ownKeys(obj)); // 输出: ["a", "b", Symbol(sym)]
    ```
  - **用例**：综合访问所有属性键，适用于元编程。


#### 对比分析

| 方法                      | 返回内容                     | 是否包括不可枚举 | 是否包括符号属性 | 常见用例                     |
|---------------------------|------------------------------|-------------------|-------------------|------------------------------|
| `Object.keys`             | 可枚举属性键名数组           | 否                | 否                | 遍历属性名，生成字段列表     |
| `Object.values`           | 可枚举属性值数组             | 否                | 否                | 处理值，计算统计             |
| `Object.entries`          | 可枚举属性键值对数组         | 否                | 否                | 按值排序，数据结构转换       |
| `Object.getOwnPropertyNames` | 所有属性名数组（字符串）   | 是                | 否                | 调试，访问不可枚举属性       |
| `Object.getOwnPropertySymbols` | 所有符号属性数组         | 是                | 是                | 处理符号键，私有数据管理     |
| `Reflect.ownKeys`         | 所有属性键数组（字符串+符号）| 是                | 是                | 元编程，综合属性访问         |

#### Tip
一个意外的细节是，`Object.keys` 等方法不包括原型链上的属性，这与 `for...in` 循环不同，后者会遍历原型链上的可枚举属性。但需注意在处理继承对象时可能需要额外逻辑。