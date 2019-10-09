对于 `Web Worker` ，一个 `tab` 页面只能对应一个 `Worker` 线程，是相互独立的；
而 `SharedWorker` 提供了能力能够让不同标签中页面共享的同一个` Worker `脚本线程；
当然，有个很重要的限制就是它们**需要满足同源策略**，也就是需要在同域下；
在页面（可以多个）中实例化` Worker` 线程：

# 创建

```js
// main.js
var myWorker = new SharedWorker("worker.js");
myWorker.port.start();
//start方法手动启动端口
myWorker.port.postMessage("hello, I'm main");
myWorker.port.onmessage = function(e) {
  console.log('Message received from worker');
}
```

# 实例

```html
//index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    1231321
</body>
<script>
    var myWorker = new SharedWorker("../js/worker.js");

myWorker.port.start();
myWorker.port.postMessage("hello, I'm first");

myWorker.port.onmessage = function(e) {
  console.log(e.data);
  console.log('Message received from worker');
}
</script>
</html>
```

```html
//index2.html
​```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    1231321
</body>
<script>
    var myWorker = new SharedWorker("../js/worker.js");

myWorker.port.start();
//start方法手动启动端口
myWorker.port.postMessage("hello, I'm second");

myWorker.port.onmessage = function(e) {
  console.log(e.data);
  console.log('Message received from worker');
}
</script>
</html>
​```
```

```js
//worker.js
​```
onconnect = function(e) {
    var port = e.ports[0];
    port.addEventListener('message', function(e) {
      var workerResult = 'Result: ' + (e.data);
      port.postMessage(workerResult);
    });
    port.start(); //当使用addEventListener就需要， 如果使用onmessage，端口就会被隐式调用
}
​```
```

![1569235404658](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1569235404658.png)

![1569235412154](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\1569235412154.png)