### 概念
> ` reduce() ` 方法是一种强大的数组操作工具，允许你对数组中的所有元素执行一个由用户提供的reducer函数，从而将数组“折叠”或“累积”成一个单一的最终值。reduce()方法的核心思想是通过连续迭代数组中的每一个元素，将上一次迭代的结果（称为“累计器”或“累加器”）与当前元素相结合，产生一个新的累计值。这个过程一直持续到数组的末尾，最后返回累计器的最终状态，即数组元素经过一系列计算后的总和

### 基本语法
```js
arr.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])
```
- arr：要进行操作的数组
- callback：这是一个函数，被称为reducer函数，是reduce()方法的核心。它接收以下参数：
  - accumulator：累计器，它是上一次调用回调函数时返回的值，或者是初始值（在第一次迭代时）。在每次迭代中，累计器的值会被传递到回调函数中，并与当前值进行某种运算，然后返回新值作为下一次迭代的累计器
  - currentValue：当前正在处理的数组元素
  - index（可选）：当前元素在数组中的索引
  - array（可选）：调用reduce()方法的原始数组
- initialValue（可选）：提供一个初始值作为第一次调用回调函数时的累计器。如果不提供此参数，reduce()会从数组的第二个元素开始迭代，并使用数组的第一个元素作为初始累计器值。如果提供了初始值，迭代则从数组的第一个元素开始

### 工作流程
1. 初始化：如果有提供initialValue，则将它作为首次迭代时的accumulator值；否则，accumulator取数组的第一个元素值，currentValue取数组的第二个元素值
2. 迭代：对于数组中的每个元素（从第一个或第二个开始，取决于是否提供初始值），调用reducer函数，将accumulator、currentValue（以及可选的index和array）作为参数。回调函数返回的值作为下一次迭代的accumulator
3. 终止：当数组中的所有元素都被处理过后，最后一次迭代返回的accumulator值作为reduce()方法的最终结果

### 应用场景
- 求和
最简单的reduce()用法是计算数组元素的总和：
```js
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((acc, cur) => acc + cur, 0); // sum = 15
```
- 对象属性值的聚合
reduce()也可以用于对象数组，将对象的某个属性值累积起来：
```js
const products = [
  { name: 'Product A', price: 10 },
  { name: 'Product B', price: 20 },
  { name: 'Product C', price: 30 }
];

const totalPrice = products.reduce((total, product) => total + product.price, 0); // totalPrice = 60
```
- 数组去重
借助reduce()创建一个仅包含唯一值的新数组：
```js
const uniqueValues = arr.reduce((acc, value) => {
  if (!acc.includes(value)) {
    acc.push(value);
  }
  return acc;
}, []);
```
- 数组转对象
将键值对数组转化为一个对象：
```js
const keyValuePairs = [['key1', 'value1'], ['key2', 'value2']];
const object = keyValuePairs.reduce((obj, pair) => {
  obj[pair[0]] = pair[1];
  return obj;
}, {});
// object = { key1: 'value1', key2: 'value2' }
```
### 注意 ⚠️
- 返回值：reduce()方法返回的是reducer函数最后一次返回的累计器值
- 空数组：如果数组为空且没有提供initialValue，reduce()会抛出TypeError。若提供initialValue，即使数组为空，reduce()也会返回这个初始值
- 不变性：reduce()方法不会改变原始数组，它仅基于数组元素生成一个新的结果
- 通用性：reduce()方法适用于多种数组操作，包括但不限于求和、乘积、最大值、最小值、查找唯一值、数组扁平化、对象合并等复杂数据转换