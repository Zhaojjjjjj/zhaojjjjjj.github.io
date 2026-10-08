 - 元素之间的穿插元素
 ```js
const intersperse = (a, s) => [...Array(2 * a.length - 1)].map((_, i) => (i % 2 ? s : a[i / 2]));
 
intersperse(['A', 'B', 'C'], '/'); 
// ['A', '/', 'B', '/', 'C']
 
intersperse([<li>A</li>, <li>B</li>, <li>C</li>], <li>/</li>); 
// [<li>A</li>, <li>/</li>, <li>B</li>, <li>/</li>, <li>C</li>]
```

- 重复数组
```js
// arr 是传入的数组，n 是重复的次数
const repeat = (arr, n) => [].concat(...Array(n).fill(arr));
 
// Or
const repeat = (arr, n) => Array(n).fill(arr).flat();
 
repeat([1, 2, 3], 3); 
// [1, 2, 3, 1, 2, 3, 1, 2, 3]
```

- 查找数组中最小值的索引
```js
const indexOfMin = (arr) => arr.reduce((prev, curr, i, a) => (curr < a[prev] ? i : prev), 0);
 
indexOfMin([6, 4, 8, 2, 10]); 
// 3
```

- 查找数组中最大值的索引
```js
const indexOfMax = (arr) => arr.reduce((prev, curr, i, a) => (curr > a[prev] ? i : prev), 0);
 
indexOfMax([1, 3, 9, 7, 5]); 
// 2
```

- 获取连续元素的所有数组
```js
const getConsecutiveArrays = (arr, size) =>
    size > arr.length ? [] : arr.slice(size - 1).map((_, i) => arr.slice(i, size + i));
 
getConsecutiveArrays([1, 2, 3, 4, 5], 2); 
// [[1, 2], [2, 3], [3, 4], [4, 5]]
getConsecutiveArrays([1, 2, 3, 4, 5], 3); 
// [[1, 2, 3], [2, 3, 4], [3, 4, 5]]
getConsecutiveArrays([1, 2, 3, 4, 5], 6); 
// []
```

- 将值转换为数组
```js
const castArray = (value) => (Array.isArray(value) ? value : [value]);
 
castArray(1); // [1]
castArray([1, 2, 3]); // [1, 2, 3]
```

- 通过下标交换数组中两个值的位置
``` js
// a 是数组，i j 是交换的两个值的下标，i必须比j小
const swapItems = (a, i, j) =>
    (a[i] && a[j] && [...a.slice(0, i), a[j], ...a.slice(i + 1, j), a[i], ...a.slice(j + 1)]) || a;
 
swapItems([1, 2, 3, 4, 5], 1, 4); // [1, 5, 3, 4, 2]
```

- 删除数组中的重复值
```js
const removeDuplicate = (arr) => arr.filter((i) => arr.indexOf(i) === arr.lastIndexOf(i));
 
removeDuplicate(['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd']); 
//  ['h', 'e', 'w', 'r', 'd']
```

- 获取数组中值的索引
```js
const indices = (arr, value) => arr.reduce((acc, v, i) => (v === value ? [...acc, i] : acc), []);
// Or
const indices = (arr, value) => arr.map((v, i) => (v === value ? i : false)).filter(Boolean);
 
indices(['h', 'e', 'l', 'l', 'o'], 'l'); // [2, 3]
indices(['h', 'e', 'l', 'l', 'o'], 'w'); // []
```

- 获取数组的所有子集
```js
const getSubsets = (arr) => arr.reduce((prev, curr) => prev.concat(prev.map((k) => k.concat(curr))), [[]]);
 
getSubsets([1, 2]); // [[], [1], [2], [1, 2]]
getSubsets([1, 2, 3]); // [[], [1], [2], [1, 2], [3], [1, 3], [2, 3], [1, 2, 3]]
```
- 获取数组的所有第 n 的倍数项
```js
const getNthItems = (arr, nth) => arr.filter((_, i) => i % nth === nth - 1);
 
getNthItems([1, 2, 3, 4, 5, 6, 7, 8, 9], 2); 
// [2, 4, 6, 8]
getNthItems([1, 2, 3, 4, 5, 6, 7, 8, 9], 3); 
// [3, 6, 9]
```

- 按数组中的给定键对数组进行排序
```js
const sortBy = (arr, k) => arr.concat().sort((a, b) => (a[k] > b[k] ? 1 : a[k] < b[k] ? -1 : 0));
 
const people = [
    { name: 'Foo', age: 42 },
    { name: 'Bar', age: 24 },
    { name: 'Fuzz', age: 36 },
    { name: 'Baz', age: 32 },
];
sortBy(people, 'age');
//  [
//      { name: 'Bar', age: 24 },
//      { name: 'Baz', age: 32 },
//      { name: 'Fuzz', age: 36 },
//      { name: 'Foo', age: 42 },
//  ]
```

- 按数组中的给定键查找数组中的最小项
```js
const minBy = (arr, key) => arr.reduce((a, b) => (a[key] < b[key] ? a : b), {});
 
const people = [
    { name: 'Bar', age: 24 },
    { name: 'Baz', age: 32 },
    { name: 'Foo', age: 42 },
    { name: 'Fuzz', age: 36 },
];
minBy(people, 'age'); 
// { name: 'Bar', age: 24 }
```

- 按数组中的给定键查找数组中的最大项
```js
const maxBy = (arr, key) => arr.reduce((a, b) => (a[key] >= b[key] ? a : b), {});
 
const people = [
    { name: 'Bar', age: 24 },
    { name: 'Baz', age: 32 },
    { name: 'Foo', age: 42 },
    { name: 'Fuzz', age: 36 },
];
maxBy(people, 'age'); 
// { name: 'Foo', age: 42 }
```

- 创建笛卡尔乘积
```js
const cartesian = (...sets) => sets.reduce((acc, set) => acc.flatMap((x) => set.map((y) => [...x, y])), [[]]);
 
cartesian([1, 2], [3, 4]); 
// [ [1, 3], [1, 4], [2, 3], [2, 4] ]
 
       3       4
   ---------------
1 | [1, 3]  [1, 4]
  |
2 | [2, 3]  [2, 4]
```

- 检查数组是否为空
```js
// arr 是一个数组
const isEmpty = (arr) => Array.isArray(arr) && !arr.length;
 
isEmpty([]);
// true
isEmpty([1, 2, 3]);
// false
```