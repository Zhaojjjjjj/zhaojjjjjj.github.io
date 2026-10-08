
### 1. **ES2015 中的 JavaScript Set 基础功能**

#### 1.1 创建 Set 和去重功能
可以创建一个空的 `Set` 或通过数组初始化一个 `Set`，任何重复的值都会被自动移除。
```javascript
const languages = new Set(["JavaScript", "TypeScript", "HTML", "JavaScript"]);
console.log(languages.size);  // 输出 3，因为 "JavaScript" 是重复的
```

#### 1.2 添加元素到 Set
使用 `add` 方法向 `Set` 添加元素。如果该元素已经存在，`Set` 大小不会改变。
```javascript
languages.add("CSS");
languages.add("JavaScript");  // 已存在，不会重复添加
console.log(languages.size);  // 输出 4
```

#### 1.3 删除元素
可以使用 `delete` 方法从 `Set` 中删除元素。
```javascript
languages.delete("TypeScript");
console.log(languages.size);  // 输出 3
```

#### 1.4 检查元素是否存在
`Set` 提供了 `has` 方法用来检查元素是否存在。
```javascript
console.log(languages.has("JavaScript"));  // 输出 true
console.log(languages.has("TypeScript"));  // 输出 false
```

#### 1.5 遍历 Set
可以通过 `forEach` 或 `for...of` 遍历 Set 的所有元素。
```javascript
languages.forEach(element => console.log(element));
// 输出 "JavaScript", "HTML", "CSS"
```

#### 1.6 清空 Set
可以使用 `clear` 方法清空 `Set` 中的所有元素。
```javascript
languages.clear();
console.log(languages.size);  // 输出 0
```

### 2. **新的 Set 操作方法**

在最新的 JavaScript 提案中，`Set` 增加了常见的集合操作，如**并集**、**交集**、**差集**、**对称差集**等，这使得 `Set` 更加完整和实用。

#### 2.1 并集（`Set.prototype.union(other)`）
`union` 方法返回两个集合的并集，即包含两个集合中所有元素的集合，且每个元素只出现一次。
```javascript
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);
const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const allLanguages = frontEndLanguages.union(backEndLanguages);
console.log(allLanguages);  // 输出 Set {"JavaScript", "HTML", "CSS", "Python", "Java"}
```
这是 SQL 中 **FULL OUTER JOIN** 的等价操作。

#### 2.2 交集（`Set.prototype.intersection(other)`）
`intersection` 方法返回两个集合的交集，即同时存在于两个集合中的元素。
```javascript
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);
const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const commonLanguages = frontEndLanguages.intersection(backEndLanguages);
console.log(commonLanguages);  // 输出 Set {"JavaScript"}
```
这是 SQL 中 **INNER JOIN** 的等价操作。

#### 2.3 差集（`Set.prototype.difference(other)`）
`difference` 方法返回当前集合中存在但在另一个集合中不存在的元素。
```javascript
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);
const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const onlyFrontEnd = frontEndLanguages.difference(backEndLanguages);
console.log(onlyFrontEnd);  // 输出 Set {"HTML", "CSS"}

const onlyBackEnd = backEndLanguages.difference(frontEndLanguages);
console.log(onlyBackEnd);  // 输出 Set {"Python", "Java"}
```
差集类似于 SQL 中的 **LEFT JOIN** 操作。

#### 2.4 对称差集（`Set.prototype.symmetricDifference(other)`）
`symmetricDifference` 返回两个集合中，存在于其中一个集合但不在另一个集合中的所有元素。
```javascript
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);
const backEndLanguages = new Set(["Python", "Java", "JavaScript"]);

const diffLanguages = frontEndLanguages.symmetricDifference(backEndLanguages);
console.log(diffLanguages);  // 输出 Set {"HTML", "CSS", "Python", "Java"}
```
这类似于 SQL 中 **FULL OUTER JOIN**，但不包括同时存在于两个集合中的元素。

#### 2.5 子集（`Set.prototype.isSubsetOf(other)`）
`isSubsetOf` 方法用于判断一个集合是否是另一个集合的子集。
```javascript
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);
const declarativeLanguages = new Set(["HTML", "CSS"]);

console.log(declarativeLanguages.isSubsetOf(frontEndLanguages));  // 输出 true
console.log(frontEndLanguages.isSubsetOf(declarativeLanguages));  // 输出 false
```

#### 2.6 超集（`Set.prototype.isSupersetOf(other)`）
`isSupersetOf` 方法用于判断一个集合是否是另一个集合的超集。
```javascript
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);
const declarativeLanguages = new Set(["HTML", "CSS"]);

console.log(frontEndLanguages.isSupersetOf(declarativeLanguages));  // 输出 true
console.log(declarativeLanguages.isSupersetOf(frontEndLanguages));  // 输出 false
```

#### 2.7 不相交集（`Set.prototype.isDisjointFrom(other)`）
`isDisjointFrom` 方法用于判断两个集合是否没有交集。
```javascript
const frontEndLanguages = new Set(["JavaScript", "HTML", "CSS"]);
const interpretedLanguages = new Set(["JavaScript", "Ruby", "Python"]);
const compiledLanguages = new Set(["Java", "C++", "TypeScript"]);

console.log(interpretedLanguages.isDisjointFrom(compiledLanguages));  // 输出 true
console.log(frontEndLanguages.isDisjointFrom(interpretedLanguages));  // 输出 false
```

### 3. **浏览器支持与 Polyfills**
这些新的 Set 方法已在 Safari 17 和 Chrome 122 中支持。如果需要支持旧的 JavaScript 引擎，可以使用 Polyfills（如 `core-js`）来实现这些功能。