#### 1. `Object.assign()`

`Object.assign()` 方法用于将一个或多个源对象的可枚举属性复制到目标对象。它返回目标对象。

**语法**:
```javascript
Object.assign(target, ...sources)
```
- **参数**:
  - `target`: 目标对象，接收属性的对象。
  - `...sources`: 一个或多个源对象，从中复制属性。

**示例**:
```javascript
const target = { a: 1 };
const source = { b: 2, c: 3 };
Object.assign(target, source);
console.log(target); // { a: 1, b: 2, c: 3 }
```
- 如果目标对象和源对象有相同的属性，后面的源对象的属性会覆盖前面的属性。

#### 2. `Object.entries()`

`Object.entries()` 方法返回一个给定对象自身可枚举属性的键值对数组，每个键值对都是一个包含两个元素的数组。

**语法**:
```javascript
Object.entries(obj)
```
- **参数**:
  - `obj`: 要提取键值对的对象。

**示例**:
```javascript
const obj = { name: "Alice", age: 25 };
const entries = Object.entries(obj);
console.log(entries); // [['name', 'Alice'], ['age', 25]]
```
- 此方法适合用于需要遍历对象属性的场景，返回的数组顺序与 `for...in` 循环相同，但不包括原型链上的属性。

#### 3. `Object.fromEntries()`

`Object.fromEntries()` 方法将一个键值对列表转换为一个新对象。它通常用于将 `Object.entries()` 返回的数组转换回对象。

**语法**:
```javascript
Object.fromEntries(iterable)
```
- **参数**:
  - `iterable`: 一个可迭代的对象（如数组或 Map），其中每个元素都是一个包含两个元素的数组，第一个元素是键，第二个元素是值。

**示例**:
```javascript
const entries = [['name', 'Alice'], ['age', 25]];
const obj = Object.fromEntries(entries);
console.log(obj); // { name: 'Alice', age: 25 }
```
- 此方法可以用于将数据从数组形式转换为对象形式，便于后续操作。

#### 4. `Math.sqrt()`

`Math.sqrt()` 方法返回非负数的平方根。如果传入负数，则返回 `NaN`（不是一个数字）。

**语法**:
```javascript
Math.sqrt(x)
```
- **参数**:
  - `x`: 要计算平方根的数字。

**示例**:
```javascript
console.log(Math.sqrt(9)); // 3
console.log(Math.sqrt(-1)); // NaN
```
- 常用于数学计算和几何问题中。

#### 5. `Math.abs()`

`Math.abs()` 方法返回一个数的绝对值，即去掉其符号后的值。

**语法**:
```javascript
Math.abs(x)
```
- **参数**:
  - `x`: 要计算绝对值的数字。

**示例**:
```javascript
console.log(Math.abs(-5)); // 5
console.log(Math.abs(5)); // 5
```
- 此方法适合用于需要处理正负数但只关心大小的场景。


## ******************** ARRAY ********************

#### 数组过滤
**描述**: 使用 `filter` 方法从数组中提取符合特定条件的元素

**案例**:
```javascript
const scoresList = [85, 92, 73, 98, 80];
const highScores = scoresList.filter(score => score > 75);
// highScores 将是 [85, 92, 98, 80]
```
**使用场景**: 当需要从数组中移除不符合条件的元素时，使用 `filter` 可以保持数据整洁

#### 数组映射
**描述**: 使用 `map` 方法将数组中的每个元素转换为新的值

**案例**:
```javascript
const sideLengths = [2, 3, 4];
const volumes = sideLengths.map(length => length ** 3);
// volumes 将是 [8, 27, 64]
```
**使用场景**: 当需要对数组中的每个元素进行数学运算或转换时，使用 `map` 是非常方便的

#### 嵌套数组扁平化
**描述**: 使用 `flat` 方法将嵌套数组展平为一维数组。在这个例子中，我们将包含多层嵌套的水果列表展平

**案例**:
```javascript
const fruitBaskets = [
  ["Bananas", ["Ripe", "Green"]],
  ["Oranges", ["Navel", "Blood"]]
];
const flatFruits = fruitBaskets.flat();
// flatFruits 将是 ["Bananas", "Ripe", "Green", "Oranges", "Navel", "Blood"]
```
**使用场景**: 当处理嵌套结构的数据时，展平可以简化后续的数据操作

#### 去重集合
**描述**: 使用 `Set` 来移除数组中的重复元素。通过将原始数组转换为 `Set`，再转换回数组，可以确保每个元素都是唯一的

**案例**:
```javascript
const participants = ["David", "Emma", "Frank", "David"];
const uniqueParticipants = [...new Set(participants)];
// uniqueParticipants 将是 ["David", "Emma", "Frank"]
```
**使用场景**: 数据清理时，确保没有重复项非常重要，特别是在处理用户列表或其他重要数据时

#### 简化条件赋值
**描述**: 使用三元运算符来简化条件判断

**案例**:
```javascript
const temperature = 30;
const weatherMessage = temperature > 25 ? "It's warm!" : "It's cold.";
console.log(weatherMessage); // 输出：It's warm!
```
**使用场景**: 当需要根据简单条件执行赋值操作时，三元运算符可以提高代码的简洁性

#### 字符串反转
**描述**: 一行代码可以反转字符串。通过将字符串拆分为字符数组、反转顺序并重新连接成字符串来实现

**案例**:
```javascript
const greeting = "Goodbye!";
const reversedGreeting = greeting.split('').reverse().join('');
// reversedGreeting 将是 "!eybdooG"
```
**使用场景**: 字符串操作任务，如反转字符串或提取子字符串，可以通过简洁的一行代码完成

#### 属性存在性检查
**描述**: 使用 `in` 运算符检查对象中是否存在特定属性。这可以帮助验证数据的完整性

**案例**:
```javascript
const userProfile = { name: "Bob", age: 25 };
const hasAddress = "address" in userProfile; // hasAddress 将是 false
```
**使用场景**: 在处理用户配置文件或其他对象时，检查属性是否存在是一项常见需求

#### 默认参数设置
**描述**: 在函数定义时为参数提供默认值，以便在未提供参数时使用备用值

**案例**:
```javascript
const welcomeUser = (username = "Visitor") => `Welcome, ${username}!`;
console.log(welcomeUser()); // 输出：Welcome, Visitor!
console.log(welcomeUser("Alice")); // 输出：Welcome, Alice!
```
**使用场景**: 提供默认参数可以提高函数的健壮性，避免因缺少参数而导致错误

#### 数组内容清理
**描述**: 使用 `filter` 方法结合 `Boolean` 构造函数来移除数组中的假值（如 null、undefined、0 等）

**案例**:
```javascript
const mixedValues = [2, null, undefined, "", false, 5];
const truthyValues = mixedValues.filter(Boolean); // truthyValues 将是 [2, 5]
```
**使用场景**: 清理包含无效数据的数组，以确保后续处理的数据是有效的

#### 动态键名生成
**描述**: 使用计算属性名动态创建对象键。可以根据变量或表达式来生成键名

**案例**:
```javascript
const dynamicKey = "age";
const personInfo = { [dynamicKey]: 28 };
// personInfo 将是 {age: 28}
```
**使用场景**: 动态对象键在构建结构未预先设定的对象时非常有用，例如在处理动态表单数据时