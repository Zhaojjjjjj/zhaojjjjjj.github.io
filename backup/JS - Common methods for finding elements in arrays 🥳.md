1. Array.prototype.indexOf() : Returns the index of the first match in the array, or -1 if no match is found. (Accept the second parameter to specify the starting position of the search)
```js
let arr = [1, 2, 3, 4, 5];
let index = arr.indexOf(3); // Output 2
``` 
2. Array.prototype.includes() : Returns a Boolean value indicating whether the given element exists in the array. (Accept the second parameter to specify the starting position of the search) 🥇 
```js
let arr = [1, 2, 3, 4, 5];
let isIncluded = arr.includes(3); // Output true 
```
4. Array.prototype.find() : Returns the value of the first element in the array that meets the condition, otherwise returns undefined. (Accept the second parameter to specify the starting position of the search)
```js
let arr = [{name: "Tom", age: 18}, {name: "Jerry", age: 20}];
let person = arr.find(item => item.age === 20); // Output {name: "Jerry", age: 20}
```
5. Array.prototype.findIndex() : Returns the index of the first element in the array that meets the condition, otherwise returns -1. (Accept the second parameter to specify the starting position of the search) 🥇 
```js
let arr = [{name: "Tom", age: 18}, {name: "Jerry", age: 20}];
let index = arr.findIndex(item => item.age === 20); // Output 1
```