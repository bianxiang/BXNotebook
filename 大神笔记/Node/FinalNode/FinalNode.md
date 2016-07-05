## 异步编程

文章：[利用Promise风格简化异步编程](
http://know.cujojs.com/tutorials/async/async-programming-is-messy)

产生随机字符串：
```js
Math.random().toString(36) // "0.33rg0njmx0iqw7b9"
Math.random().toString(36).substr(3, 11) // "bbv63xqse3a"
```