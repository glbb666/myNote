### promise 三种状态

`promise`是一个容器，里面保存着某个未来才会结束的事件。它的初始状态为`pending`，它的状态由异步操作的结果来决定，一旦状态改变，就不会再改变了。

- `pending`未完成
- `resolve`成功
- `reject`失败

> - promise 初始状态为`pending`，操作结束后，可能变为`resolve`或`reject`
> - 一旦状态改变，不会再变

### promise 用法

#### **构造实例**

- 接收一个函数作为参数，这个函数有两个函数参数。分别将`promise`从`pending`改为成功/失败状态。js引擎提供，不需自己部署。

```javascript
const promise = new Promise(function(resolve, reject) {})
```

#### **then(function(value){},function(err){})**

- then方法接收两个函数（第二个可选），分别为resolve和reject状态的回调函数
- 这两个函数都接受promise传出的值作为参数
- then方法**返回的是一个新的Promise实例**。因此可采用链式写法，前一个状态改变，才会进到下一个

```javascript
const promise = new Promise(function(resolve, reject) {
  resolve(1)
})
promise.then(
  function(value) {
    console.log(value)
  },
  function(err) {
    console.log(err)
  }
)
```

#### `catch`

- 接受一个函数，用于指定发生错误时（`then`方法指定回调运行抛错及`reject`）的回调函数。
- 错误具有“冒泡”性质，错误总是会被下一个`catch`语句捕获
- 该方法返回一个`promise`

> 建议不要在then方法里面定义 Reject 状态的回调函数（即then的第二个参数），总是使用`catch`方法。如下

```javascript
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

#### promise链中返回promise

```javascript
const p1 = new Promise(function(resolve, reject) {
  resolve(1)
})
const p2 = new Promise(function(resolve, reject) {
  resolve(2)
})

p1.then(function(value) {
  console.log(value)
  return p2
}).then(function(value) {
  console.log(value)
})
//上面第二段等同于
const p3 = p1.then(function(value) {
  console.log(value)
  return p2
})
p3.then(function(value) {
  console.log(value)
})
```

> p1、p2是同时开始异步的

若要在一个`promise`被解决后再触发一个`promise`，如下

```javascript
const p1 = new Promise(function(resolve, reject) {
  resolve(1)
})
p1.then(function(value) {
  console.log(value)
  const p2 = new Promise(function(resolve, reject) {
    reject(2)
  })
  return p2
}).then(function(value) {
  console.log(value)
})
```

### 响应多个promise

#### promise.all

```javascript
const p = Promise.all(p1,p2,p3)
```

- 接受一个可迭代对象，当对象所有promise实例都完成后，才会解决返回的promise
  - 当数组所有`promise`实例都变为`resolve`，p才会变为`resolve`
  - 当数组中有一个为`reject`，p会变为`reject`
- 返回一个`promise`

#### promise.race

和`promise`的区别在于，数组中若有一个变为`resolve`，则返回的`promise`就为`resolve`。

#### promise的缺点

1. `promise`不可以取消
2. 如果不设置回调，`Promise`内部抛出的错误，不会反应到外部。
3. 当处于`pending`状态，无法得知目前进展到哪个阶段了，是刚刚开始还是即将完成。
4. `promise`不够语义化

