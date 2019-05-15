### 准备

- js 顺序解析 script 标签内的代码。但是每个 script 标签内的代码执行是互不干扰的

```
// 2 可以正常输出
<script>
  error;
  console.log(1);
</script>
<script>
  console.log(2);
</script>
```

- 语法错误无法捕获，一般采用 hint 在编写阶段排除

### 开始捕获

#### try...catch...

- 同步代码

```
try {
    console.log(error);
}catch(e){
    console.log(e);
}
```

- 异步代码
  无法捕获

#### window.onerrror 全局捕获

```
window.onerror = function(msg, url, row, col, error){

    // 阻止页面向上抛出
    return true;
}
```

无法捕获 html 出现异常

```
// 404
<img url="xxx.png"/>
```

#### 捕获 html 请求资源错误

1. 当 JavaScript 运行时错误（包括语法错误）发生时，window 会触发一个`ErrorEvent`接口的 error 事件，并执行 window.onerror()。
2. 当一项资源（如<img>或<script>）加载失败，加载资源的元素会触发一个`Event`接口的 error 事件，并执行该元素上的 onerror()处理函数。这些 error 事件不会向上冒泡到 window，不过（至少在 Firefox 中）能被单一的 window.addEventListener 捕获。

```
 window.addEventListener('error',(e) =>{
     return true;
 }， true);
```

#### 捕获 promise reject

- catch 捕获

```
new Promise((resolve, reject) =>{
    reject('first error');
}).catch(e =>{
    console.log(e);
})
```

- 全局捕获

```
window.addEventListener("unhandlerrejection", (e) =>{
    e.preventDefault();
    console.log(e.reason);
    return true;
})

Promise.reject("first error");
Promise.reject("second error");
Promise.resolve().then(res =>{
    throw "third error";
});
```

### 捕获 node 异常

```
process.on("uncaughtException", (e) => {});
process.on("unhandledRejection", (e) =>{});
app.on("error", (e) =>{});
```

### JS 录屏实现思路

1. 用 XPath 记录用户操作栈
2. 将 XPath 发送给客户端
   - 解析生成 gif
   - socket 将用户操作的图片（用户操作的帧）发送给后端，后端将图片集生成一张图片（html2canvas）

- Session Stack

> 将容错信息发送到后台，最好不要用 ajax,影响性能(浏览器并发有限性)，改用 navigator.sendBeacon()，在浏览器空闲的时候发，
