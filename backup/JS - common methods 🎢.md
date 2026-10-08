1. **反转字符串**：使用一行代码反转字符串：
   ```javascript
   const reversedString = str.split('').reverse().join('');
   ```

2. **数组求和**：使用 `reduce()` 方法计算数组中所有元素的和：
   ```javascript
   const sum = numbers.reduce((acc, curr) => acc + curr, 0);
   ```

3. **查找数组中的最大值和最小值**：使用扩展运算符和 `Math.max()`、`Math.min()`：
   ```javascript
   const max = Math.max(...numbers);
   const min = Math.min(...numbers);
   ```

4. **去除数组中的重复项**：使用 `Set` 创建一个不含重复项的新数组：
   ```javascript
   const uniqueArray = [...new Set([1, 2, 2, 3])];
   ```

5. **扁平化多维数组**：使用 `flat()` 方法将嵌套数组扁平化：
   ```javascript
   const flatArray = nestedArray.flat(2);
   ```

6. **复制文本到剪贴板**：使用剪贴板 API 复制文本：
   ```javascript
   function copyText(text) {
       navigator.clipboard.writeText(text);
   }
   ```

7. **检查值是否为数字**：使用 `typeof` 和 `!isNaN()` 检查：
   ```javascript
   const isNumber = value => typeof value === 'number' && !isNaN(value);
   ```

8. **生成范围内的随机数**：生成两个值之间的随机数：
   ```javascript
   const getRandom = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min;
   ```

9. **将字符串转换为数字**：使用 `+` 操作符快速转换：
   ```javascript
   const num = +"42"; // 42
   ```

10. **函数中的默认参数**：设置默认参数值以避免未定义的值：
    ```javascript
    function greet(name = 'Guest') {
        console.log(`Hello, ${name}`);
    }
    ```

11. **将参数对象转换为数组**：将函数中的 `arguments` 转换为实际数组：
    ```javascript
    const argsArray = Array.from(arguments);
    ```

12. **防抖函数**：确保函数不会在短时间内被多次调用：
    ```javascript
    function debounce(func, delay) {
        let timeout;
        return function(...args) {
            clearTimeout(timeout);
            timeout = setTimeout(() => func.apply(this, args), delay);
        };
    }
    ```

13. **节流函数**：确保函数在指定时间内最多只被调用一次：
    ```javascript
    function throttle(func, limit) {
        let lastFunc;
        let lastRan;
        return function(...args) {
            if (!lastRan) {
                func.apply(this, args);
                lastRan = Date.now();
            } else {
                clearTimeout(lastFunc);
                lastFunc = setTimeout(() => {
                    if (Date.now() - lastRan >= limit) {
                        func.apply(this, args);
                        lastRan = Date.now();
                    }
                }, limit - (Date.now() - lastRan));
            }
        };
    }
    ```

14. **检查对象是否为空**：使用 `Object.keys()` 检查对象是否没有属性：
    ```javascript
    const isEmpty = obj => Object.keys(obj).length === 0;
    ```

15. **短路求值**：使用短路运算符条件赋值：
    ```javascript
    const message = user.isLoggedIn && "Welcome back!";
    ```

16. **对象属性简写**：当属性名与变量名相同时，可以使用简写语法：
    ```javascript
    const person = { name, age };
    ```

17. **数组解构赋值**：将数组中的值赋给变量：
    ```javascript
    const [first, second] = [10, 20];
    ```

18. **对象解构赋值**：从对象中提取属性并赋给变量：
    ```javascript
    const {name, age} = {name: "Jane", age: 30};
    ```

19. **模板字面量**：用于方便的字符串插值：
    ```javascript
    const greeting = `Hello, ${name}!`;
    ```

20. **将 NodeList 转换为数组**：使用 `Array.from()` 转换 NodeList：
    ```javascript
    const nodeArray = Array.from(nodeList);
    ```

21. **检查数组是否包含某个元素**：使用 `includes()` 方法检查值是否存在于数组中：
    ```javascript
    console.log(fruits.includes('banana')); // true
    ```

22. **获取数组中的唯一值**：可以使用 `Set` 获取唯一值：
    ```javascript
    const uniqueValues = Array.from(new Set([1, 2, 2, 3, 3]));
    ```

23. **查找对象属性**：使用 `Object.keys()` 和 `Object.values()` 获取对象的键和值：
    ```javascript
    console.log(Object.keys(obj)); // ['name', 'age']
    console.log(Object.values(obj)); // ['Alice', 30]
    ```

24. **合并对象**：使用扩展运算符合并对象：
    ```javascript
    const mergedObj = { ...obj1, ...obj2 };
    ```

25. **映射数组**：使用 `map()` 方法对每个元素应用函数进行转换：
    ```javascript
    const doubled = numbers.map(num => num * 2); // [2, 4, 6]
    ```

26. **过滤数组**：过滤掉不需要的元素：
    ```javascript
    const evens = numbers.filter(num => num % 2 === 0); // [2, 4]
    ```

27. **Promise.all()**：同时处理多个异步任务：
    ```javascript
    Promise.all([fetch('/data1'), fetch('/data2')])
        .then(responses => { /* Handle responses */ })
        .catch(error => console.error(error));
    ```

28. **将 Set 转换为数组**：使用扩展运算符将 Set 转换为数组：
    ```javascript
    const array = [...set];
    ```

29. **对数组进行排序**：对数字数组进行升序或降序排序：
    ```javascript
    numbers.sort((a, b) => a - b); // 升序
    numbers.sort((a, b) => b - a); // 降序
    ```

30. **填充字符串**：在字符串的开始或结束填充以达到指定长度：
    ```javascript
    const paddedStr = str.padStart(3, '0'); // "005"
    ```

31. **去除空格**：使用 `trim()`、`trimStart()` 和 `trimEnd()` 去除空格：
     ```javascript
     console.log(text.trim()); // "Hello"
     ```

32. **检查字符串是否包含子串**：使用 `includes()` 检查字符串中是否包含特定子串：
     ```javascript
     console.log(sentence.includes("awesome")); // true
     ```

33. **查找子串索引**：使用 `indexOf()` 查找子串的位置：
     ```javascript
     console.log(sentence.indexOf("fun")); // 14
     ```

34. **重复字符串**：使用 `repeat()` 重复字符串：
     ```javascript
     console.log(str.repeat(3)); // "hahaha"
     ```

35. **替换子串**：使用 `replace()` 替换子串：
     ```javascript
     const newText = text.replace("fun", "awesome");
     ```

36. **可选链操作符**：安全访问嵌套属性，避免错误：
     ```javascript
     console.log(user?.address?.city); // undefined，无错误发生
     ```

37. **将日期转换为本地字符串格式**：格式化日期为本地日期字符串：
     ```javascript
     console.log(date.toLocaleDateString('en-US')); // MM/DD/YYYY 格式
     ```

38. **检查值是否为数组**：使用 `Array.isArray()` 检查是否为数组：
     ```javascript
     console.log(Array.isArray([1, 2, 3])); // true
     ```

39. **检查 DOM 中元素是否存在**：使用 `document.querySelector()` 检查元素存在性：
     ```javascript
     const elementExists = !!document.querySelector('.my-element');
     ```

40. **获取当前时间戳**：使用 `Date.now()` 获取当前时间戳：
     ```javascript
     const timestamp = Date.now();
     ```

41. **格式化带逗号的数字**：使用 `toLocaleString()` 格式化数字带逗号分隔符：
     ```javascript
     console.log(number.toLocaleString()); // "1,234,567"
     ```

42. **将数字四舍五入到两位小数**：使用 `toFixed()` 四舍五入数字到两位小数：
     ```javascript
     console.log(number.toFixed(2)); // "5.68"
     ```

43. **检查变量是否未定义**：检查变量是否未定义:
      ```javascript 
      if (typeof variable === 'undefined') { /* Handle undefined */ }
      ```
    
44. **从数组中获取随机项**: 使用 Math.random() 从数组中获取随机项:
      ``` javascript 
      const randomItem = items[Math.floor(Math.random() * items.length)];
      ```
    
45. **简洁函数的箭头函数用法:** 使用箭头函数提供简洁语法:
      ```
      const add = (a, b) => a + b;
      ```
    
46. **浅拷贝对象:** 使用 Object.assign() 拷贝对象:
      ```
      const copy = Object.assign({}, original);
      ```
    
47. **类型检查:** 使用 typeof 检查变量类型:
      ```
      console.log(typeof 'Hello'); // "string"
      ```
    
48. **动态属性名:** 使用方括号设置动态属性名:
      ```
      const obj = { [key]: 'John' };
      ```
    
49. **检查空值:** 使用空合并运算符处理空或未定义的值:
      ```
      const value = user.age ?? 'Unknown';
      ```
    
50. **更易调试的表格显示:** 使用 console.table() 在控制台显示表格数据:
      ```
      console.table(users);
      ```