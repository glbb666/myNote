[TOC]



## 二叉树

### 二叉树深度遍历

```js
var deepth = function(root) {
  if (root === null) {
    return 0
  } else {
    let lH = deepth(root.left),
      rH = deepth(root.right)
    return Math.max(lH, rH) + 1
  }
}
```

### 二叉树前序遍历

```js
function perOder(root) {
  let res = []
  let stack = []
  let p = root
  while (stack.length != 0 || p != null) {
    if (p != null) {
      stack.push(p)
      res.push(p.data)
      p = p.left
    } else {
      p = stack.pop().right
    }
  }
  return res
}
```

### 二叉树中序遍历

```js
function midOder(root) {
  let stack = []
  let res = []
  let p = root
  while (stack.length != 0 || p != null) {
    if (p != null) {
      stack.push(p)
      p = p.left
    } else {
      let node = stack.pop()
      res.push(node.data)
      p = node.right
    }
  }
  return res
}
```



### 二叉树后序遍历

```js
function laterOrder(root) {
  let stack = []
  let res = []
  let p = root
  while (stack.length != 0 || p != null) {
    if (p != null) {
      stack.push(p)
      res.unshift(p.data)
      p = p.right
    } else {
      let node = stack.pop()
      p = node.left
    }
  }
  return res
}
```

### 二叉树深度优先遍历

```js
function BFS(root) {
  let res = []
  let queue = []
  let p = root
  queue.push(p)
  while (queue.length) {
    p = queue.shift()
    res.push(p.data)
    if (p.left) {
      queue.push(p.left)
    }
    if (p.right) {
      queue.push(p.right)
    }
  }
  return res
}
```

## 排序

### 快排

```js
function quickSort(arr, start, end) {
  if (end - start < 1) {
    return
  }
  let l = start
  let r = end
  const target = arr[start]
  while (l < r) {
    while (l < r && arr[r] >= target) {
      r--
    }
    arr[l] = arr[r]
    while (l < r && arr[l] < target) {
      l++
    }
    arr[r] = arr[l]
  }
  arr[l] = target
  quickSort(arr, start, l - 1)
  quickSort(arr, l + 1, end)
  return arr
}
```

### 选择

```js
function selectSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    let minIdex = i
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[minIdex] > arr[j]) {
        minIdex = j
      }
    }
    [arr[minIdex], arr[i]] = [arr[i], arr[minIdex]]
  }
  return arr
}
```

### 冒泡

```js
function bubbleSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    for (let j = 0; j < arr.length - i; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]
      }
    }
  }
  return arr
}
```

## 链表

### 原地翻转递归

```js
function printListFromTailToHead(h) {
  if (h && h.next) {
    printListFromTailToHead(h.next)
  }
  console.log(h.val)
}
```

### 原地翻转非递归

```js
function reserveList(head) {
  let newHead = head,
    cur = null
  while (head && head.next) {
    //第二个条件很重要
    cur = head.next
    head.next = cur.next
    cur.next = newHead
    newHead = cur
  }
  return newHead
}
```

### 反序输出

```js
function printListFromTailToHead(h) {
  let stack = []
  while (h) {
    stack.push(h)
    h = h.next
  }
  while (stack.length != 0) {
    console.log(stack.pop().val)
  }
}
```

### 删除链表节点

```js
function deleteListNode(head, val) {
  if (head === null) return null
  let temp = { val: -1, next: head }
  let p = temp
  while (p.next.val != val) {
    p = p.next
  }
  p.next = p.next.next
  return temp.next
}
```

### 两个有序链表合并

```js
function mergeTowList(h1, h2) {
  if (!h1) return h2
  if (!h2) return h1
  let p = { val: -1, next: null },
    head = p
  while (h1 && h2) {
    if (h1.val < h2.val) {
      p.next = h1
      h1 = h1.next
    } else {
      p.next = h2
      h2 = h2.next
    }
    p = p.next
  }
  p.next = h1 || h2
  return head.next
}
```

### 找出链表倒数第K个节点

```js
function FindKthToTail(head, k) {
  if (!head || !k) return null
  let front = head
  let behind = head
  let index = 1
  while (front.next) {
    index++
    front = front.next
    if (index > k) {
      behind = behind.next
    }
  }
  return k <= index && behind
}
```

### 删除链表倒数第K个节点

```js
var removeNthFromEnd = function(head, k) {
  let first = head,
    second = head
  while (k > 0) {
    first = first.next
    k--
  }
  if (!first) return head.next // 删除的是头节点
  while (first.next) {
    first = first.next
    second = second.next
  }
  second.next = second.next.next
  return head
}
```



## 算法

### 两个栈实现队列

```js
const stack1 = []
const stack2 = []
function push(node) {
  stack1.push(node)
}
function pop() {
  while (stack2.length === 0) {
    //第二个里面没有东西时才能保证顺序
    while (stack1.lenght > 0) {
      //第一个栈中有东西可出栈
      stack2.push(stack1.pop())
    }
  }
  return stack2.pop() || null
}
```

### 搜索二维矩阵

```js
//思路：先找行，再找列
var searchMatrix = function(matrix, target) {
  if (matrix.length == 0 || target === null) return false
  let raw = matrix.length
  let col = matrix[0].length
  let tari = 0,
    i = 0,
    j = 0
  while (i < raw) {
    if (matrix[i][col - 1] >= target) {
      tari = i
      break
    }
    i++
  }
  while (tari >= 0 && j < col) {
    if (matrix[tari][j] === target) return true
    j++
  }
  return false
}
```

### 从尾到头打印链表

```js
function printListFromTailToHead(h) {
  var arr = []
  while (h) {
    arr.unshift(head.val)
    h = h.next
  }
  return arr
}
```

## 模拟

### call

```js
Function.prototype.myCall = function(context = window,...args){
    if(typeof this !== 'function'){
        throw TypeError('必须使用函数调用')
    }
    const fn = Symbol()
    context[fn]=this
    const res = context[fn](...args)
    delete context[fn]
    return res
}
```

### apply

```js
Function.prototype.myApply = function (context = window, args) {
  if (typeof this !== 'function') {
     throw TypeError('必须使用函数调用')
  }
  const fn = Symbol();
  context[fn] = this;
  let result;
  if (Array.isArray(args)) {
    result = context[fn](...args);
  } else {
    result = context[fn]();
  }
  delete context[fn];
  return result;
}
```



### bind

```js
Function.prototype.myBind = function(context = window, ...args1) {
  if (typeof this !== 'function') throw TypeError('必须使用函数调用')
  let _this = this
  let ft = function() {}
  let fn = function(...args2) {
    return _this.apply(this instanceof fn ? this : context, args1.concat(args2))
  }
  this.prototype ? (ft.prototype = this.prototype) : null
  fn.prototype = new ft()
  return fn
}
```

### EventEmit

```js
class EventEmit {
  constructor() {
    this._eventpool = {}
  }
  on(event, callback) {
    //绑定
    if (this._eventpool[event]) {
      this._eventpool[event].push(callback)
    } else {
      this._eventpool[event] = [callback]
    }
  }
  off(event) {
    //解除
    if (this._eventpool[event]) {
      delete this._eventpool[event]
    }
  }
  emit(event, ...args) {
    //执行全部回调
    if (this._eventpool[event]) {
      this._eventpool[event].forEach(fn => {
        fn(...args)
      })
    }
  }
  once(event, callback) {
    //绑定＋执行 一次
    this.on(event, (...args) => {
      callback(args)
      this.off(event)
    })
  }
}
```

### 节流

```js
function jieliu2(fn, delay) {
  let timer = null
  return (...args) => {
    if (!timer) {
      timer = setTimeout(() => {
        fn.apply(this, args)
      }, delay)
    }
  }
}
```

### 防抖

```js
function fangdou(fn, delay) {
  let timer = null
  return (...args) => {
    if (timer) {
      clearTimeout(timer)
      timer = setTimeout(() => {
        fn.apply(this, args)
      }, delay)
    }
  }
}
```

### 类型判断函数

```js
function testType(target) {
  if (target === null) {
    return 'null'
  }
  const typeOfT = typeof target
  if (typeOfT !== 'object') return typeOfT

  return Object.prototype.toString.call(target)
}
```

### 深拷贝

```js
function deepclone(target, map = new WeakMap()) {
  if (typeof target === 'object') {
    const cloneT = Array.isArray(target) ? [] : {}
    if (map.has(target)) return map.get(target)
    map.set(target)

    for (let i in target) {
      cloneT[i] = deepclone(target[i])
    }

    return cloneT
  }
  return target
}
```

### 函数柯里化

```js
function curry(fn, ...args) {
    return function(){
        var _args = [...args,...arguments];
        return _args.length<fn.length?curry.call(this, fn, ..._args):fn.apply(this,_args);
    }
}
```

### 数组扁平

```js
function flat(arr) {
  return arr.reduce(
    (last, v) => (Array.isArray(v) ? last.concat(flat(v)) : last.concat(v)),
    []
  )
}
```

### instanceOf

```js
function myInstanceOf(target, father) {
  let proto = target.__proto__
  if (proto) {
    if (proto === father.prototype) {
      return true
    } else {
      return myInstanceOf(proto, father)
    }
  } else {
    return false
  }
}
```

### JSONP

```js
function jsonp(url, obj, timeout) {
  url += url.indexOf('?') === -1 ? '?' : '&'

  for (let i in obj) {
    url += encodeURIComponent(i) + '=' + encodeURIComponent(obj[i]) + '&'
  }

  let callBack =
    'callback' +
    Math.random()
      .toString()
      .replace('.', '')
  url += 'callback=' + callBack

  let script = document.createElement('script')
  script.src = url
  document.head.appendChild(script)

  let timer = setTimeout(() => {
    document.head.removeChild(script)
    window[callBack] = null
  }, timeout)

  window[callBack] = function(data) {
    //处理数据
    callBack(data)
    document.head.removeChild(script)
    window[callBack] = null
    clearTimeout(timer)
  }
}
```

### Ajax封装

```js
function getXhr() {
  if (window.XMLHttpRequest) {
    return new XMLHttpRequest()
  } else {
    return new ActiveXObject('Microsoft.XMLHTTP')
  }
}

function getParams(data) {
  let arr = []
  for (let i in data) {
    arr.push(encodeURIComponent(i) + '=' + encodeURIComponent(data[i]))
  }
  return arr.join('&')
}

function Ajax(options) {
  options = options || {}
  options.type = (options.type || 'GET').toUpperCase()
  options.async = options.async || true
  options.timeout = options.timeout || 8000

  const xhr = getXhr()
  xhr.onreadystatechange = function(e) {
    if (xhr.readyState === 4) {
      const s = xhr.status
      if ((s >= 200 && s < 300) || s == 304) {
        console.log(xhr.responseText)
      }
    }
  }
  xhr.timeout = options.timeout
  xhr.ontimeout = function(e) {
    console.log('timeout')
    xhr.abort()
  }
  if (options.type === 'GET') {
    xhr.open(
      'GET',
      options.url + '?' + getParams(options.data),
      options.async
    )
    xhr.send()
  } else if (options.type === 'POST') {
    xhr.open('POST', options.url, options.async)
    xhr.setRequestHeader(
      'Content-Type',
      'application/x-www-form-urlencoded'
    )
    xhr.send(options.data)
  }
}


```

### reduce实现map

```js
Array.prototype.reduceToMap = function(callback) {
  return this.reduce((last, v, i) => {
    last.push(callback.call(this, v, i))
    return last
  }, [])
}
```

### reduce实现filter

```js
Array.prototype.myFilter = function(cb){
    return this.reduce((last,v,i)=>{
        if(cb.call(this,v,i))
            last.push(v)
        return last
    },[])
}
```

### 图片懒加载

```js
if (IntersectionObserver) {
  let lazyImageObserver = new IntersectionObserver((entries, observer) => {
    entries.forEach((entry, index) => {
      let lazyImage = entry.target;
      // 如果元素可见            
      if (entry.intersectionRatio > 0) {
        if (lazyImage.getAttribute("src") == "loading.gif") {
          lazyImage.src = lazyImage.getAttribute("data-src");
        }
        lazyImageObserver.unobserve(lazyImage)
      }
    })
  })
  for (let i = 0; i < img.length; i++) {
    lazyImageObserver.observe(img[i]);
  }
}
```

### 单例模式

```js
var Singleton = function(name) {
    this.name = name;
};

Singleton.prototype.getName = function() {
    alert(this.name);
};

Singleton.getInstance = (function(name) {
    var instance;
    return function(name){
        if (!instance) {
            instance = new Singleton(name);
        }
        return instance;
    }
})();

```

### 数组乱序

```js
function shuffle(a) {
    for (let i = a.length; i; i--) {
        let j = Math.floor(Math.random() * i);
        [a[i - 1], a[j]] = [a[j], a[i - 1]];
    }
    return a;
}
```

## 获取URL参数

```js
function getUrlParam(sUrl, sKey) {
  const reg = /(\?|&)(\w+)\=(\w+)/g
  let res = {}
  while (reg.exec(sUrl) != null) {
    if (res[RegExp.$2]) {
      const temp = res[RegExp.$2]
      res[RegExp.$2] = [].concat(temp, RegExp.$3)
    } else {
      res[RegExp.$2] = RegExp.$3
    }
  }
  if (sKey) {
    return res[sKey] || ''
  }
  return res
}
```

### 实现Promise.all

```js
function promiseAll(promises) {
  return new Promise(function(resolve, reject) {
    if(isArray(promises)) {
        return reject(new Error('Promises must be an array'))
    }
    var resolvedCount = 0;
    var promiseNum = promises.length;
    var resloveValue = [];
    for(let i = 0; i < promiseNum; i++) {
        Promise.resolve(promises[i]).then((value) => {
            resloveValue[i] = value;
            resolvedCount++;
            if(resolvedCount === promiseNum) {
                return resloveValue;
            }
        }, (reason) => {
            return reject(reason);
        })
    }
  })
}
```

