- 在处理文件上传，特别是使用HTML的\<input type="file">元素时，可能会遇到一个问题：如果尝试选择同一个文件两次（没有更改文件选择），某些浏览器可能不会触发change事件。是因为文件的引用没有改变，所以没有触发change事件

- 为了解决这个问题，可以在\<input type="file">上加一个额外的参数（如一个隐藏的字段或数据属性），并在每次选择文件时更新这个参数。这样即使文件本身没有改变，但由于附加参数的变化，change事件会被触发

- 但是直接使用“key”这个词会有误导，因为\<input>元素并没有一个名为“key”的属性。可使用类似的概念，如数据属性（data-属性）或隐藏字段来模拟这个行为。

```html
<input type="file" id="fileInput" data-upload-key="0">  
<button id="uploadButton">上传</button>
```
```js
document.getElementById('fileInput').addEventListener('change', function(event) {  
    // 每次文件改变时，更新数据属性  
    let input = event.target;  
    let currentValue = parseInt(input.getAttribute('data-upload-key'), 10);  
    input.setAttribute('data-upload-key', currentValue + 1);  
  
    // 处理文件上传...  
    // 需要读取event.target.files来获取文件列表  
});  
```