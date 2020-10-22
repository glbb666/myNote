思想：`Promise.all()`方法用于将多个 `Promise` 实例，包装成一个新的 `Promise` 实例。

```javascript
const p = Promise.all([p1, p2, p3]);
```

上面代码中，`p1`、`p2`、`p3`都是 `Promise` 实例，如果不是，就会用`Promise.resolve`方法转为 `Promise` 实例。另外，`Promise.all()`方法的参数可以不是数组，但必须具有 `Iterator` 接口，且返回的每个成员都是 `Promise` 实例。

`p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。

（1）只有`p1`、`p2`、`p3`的状态都变成`resolved`，`p`的状态才会变成`resolved`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。

（2）只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。

```javascript
function promiseAll(promises){
    return new Promise((resolve, reject)=>{
        // 参数判断
        if(!Array.isArray(promises)){
            throw new TypeError("promises must be an array")
        }
        let result = [] // 存放结果
        let count = 0 // 记录有几个resolved
        promises.forEach((promise, index) => {
            promise.then((res)=>{
                result[index] = res
                count++
                count === promises.length && resolve(result) // 判断是否已经完成
            }, (err)=>{
                reject(err)
            })
        })
    })
}

```

