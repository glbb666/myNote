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

### 二叉树先序遍历：根左右

先序中序后序表示的是根的顺序，先序中序后续都是深度优先，用的是栈

```js
function preOrder(root) {
  let result = []
  let stack = []
  let p = root
  while (stack.length != 0 || p != null) {
    if (p != null) {
      stack.push(p)
      result.push(p.data)
      p = p.left
    } else {
      p = stack.pop().right
    }
  }
  return result
}
```

### 二叉树中序遍历：左根右

```js
function midOrder(root) {
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

### 二叉树后序遍历：左右根

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
      p = stack.pop().left
    }
  }
  return res
}
```

### 二叉树广度优先遍历

广度优先用的是队列

```js
function BFS(root) {
  let res = []
  let queue = []
  let p = root
  queue.push(p)
  while (queue.length) {
    p = queue.shift()
    res.push(p.data)
    if (p.left)queue.push(p.left)
    if (p.right) queue.push(p.right)
  }
  return res
}
```

## 排序
### 三种简单排序（冒泡，插入，选择）

- 时间复杂度：`O(n^2)`
- 空间复杂度： `O(1)`
- 稳定性：冒泡和插入稳定，选择不稳定（记忆：人的选择就是不一定的）
### 冒泡

- 外层控制轮次，表示第几轮

- 内层控制交换，若`arr[j]>arr[j+1]`则交换`arr[j]`和`arr[j+1]`，`j`代表被排序的前一个值的下标

  (不写错可以判断临界值有没有越界)

```javascript
function bubble(arr){
	for(let i = 1;i<arr.length;i++){//外层控制轮次,表示第几轮
		for(let j = 0;j<arr.length-i;j++){//内层控制交换
			if(arr[j]>arr[j+1]){
				[arr[j],arr[j+1]] = [arr[j+1],arr[j]]
			}
		}
	}
}
```

### 插入

将左边的序列看作有序序列，每次循环将一个数字插入有序序列

从有序序列最右边开始比较，若比较的数较大，通过交换后移一位

```javascript
function insert(arr){
	for(let i = 1;i<arr.length;i++){
		let target = i;
		for(let j = i-1;j>=0;j--){
			if(arr[target]<arr[j]){
             [arr[target], arr[j]] = [arr[j], arr[target]]//交换后移动
                target = j;
            }else{
                break;
            }
		}
	}
}
```

### 选择

每次循环选取一个最小的数字放到前面的有序序列中

```javascript
function select(arr){
        for(let i = 0;i<arr.length-1;i++){
            let min = i;
            for(let j = i+1;j<arr.length;j++){
                if(arr[min]>arr[j]){
                    min = j;
                }
            }
            [arr[i],arr[min]] = [arr[min],arr[i]];
        }
    }
```

### 快速排序

- 时间复杂度，`O(n^2)`,平均`O(nlogn)`，大多数情况下小于平均值
- 空间复杂度：`O(logn)`
- 稳定性：不稳定

```javascript
function quickSort(arr,start,end){
        if(end-start<1){
            return;
        }
        let l = start;
        let r = end;
        const target = arr[start];
        while(l<r){
            while(l<r&&arr[r]>=target){
                r--;
            }
            arr[l] = arr[r];
            while(l<r&&arr[l]<target){
                l++;
            }
            arr[r] = arr[l];
        }
        arr[l] = target;
        quickSort(arr,start,l-1);
        quickSort(arr,l+1,end)
        return arr
}
```

### 归并排序

归并排序是建立在归并操作上的一种有效的排序算法，该算法是采用**分治法**的一个非常典型的应用。将已有的子序列合并，得到完全有序的序列，即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为二路归并。

- 时间复杂度`O(nlog(n))`
- 空间复杂度`O(n)`
- 稳定性：稳定

```javascript
let arr = [65,54,34,432,213,6,34]
function mergeSortRec(arr){
    if(arr.length===1){
        return arr;
    }
    var mid = Math.floor(arr.length/2);
    var left = arr.slice(0,mid);
    var right = arr.slice(mid);
    return merge(mergeSortRec(left),mergeSortRec(right))
}
function merge(left,right){
    var il = 0;
    var ir = 0;
    var result = [];
    while(il<left.length&&ir<right.length){
        if(left[il]<right[ir]){
            result.push(left[il++]);
        }else{
            result.push(right[ir++]);
        }
    }
    while(il<left.length){
        result.push(left[il++]);
    }
    while(ir<right.length){
        result.push(right[ir++]);
    }
    return result;
}
arr = mergeSortRec(arr) 
```

## 链表

### 反序输出递归
```js
function printListFromTailToHead(h) {
  if (h && h.next) {
    printListFromTailToHead(h.next)
  }
  console.log(h.val)
}
```
### 反序输出非递归（开辟空间）

思路：用栈存储，pop输出

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
### 原地翻转非递归

思想：一个旧头一个新头，利用cur把旧头之后的结点放在最前面（指向新头）作为头节点，新头始终指向头节点，当旧头后面没有东西了，即head.next = null，说明遍历结束。

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
### 删除链表给定val节点

思路：把头节点当作节点temp的后继结点，进行操作，这样就可以删除头节点了。最后以temp.next的形式返回头节点。注意的是，如果后续结点被删除，则不需要后移，因为后续结点已经不是之前的那一个结点了。

```js
var removeElements = function(head, val) {
    var temp = new ListNode(-1);
    temp.next = head;
    var p = temp;
    while(p.next){
        if(p.next.val==val){
            p.next = p.next.next;
        }else{
            p = p.next;
        }
    }
    return temp.next;
};
```

### 两个有序链表合并

思路：创建一个新的链表，把两个链表依次往后遍历，比较小的拼接到新链表尾部。

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
    k--;
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

### 判断是不是环形链表

- hash表，不存在就把结点保存，存在说明对同一个结点存在两次引用，存在环形链表
- 快慢指针，如果是环形，总是会相遇的

### 链表的中间结点

快慢指针

```javascript
var middleNode = function(head) {
    var fast = head;
    var slow = head;
    while(fast&&fast.next){
        fast = fast.next.next;
        slow = slow.next
    }
    return slow;
};
```

### 回文链表（时间复杂度`O(n)`，空间复杂度`O(1)`）

首先通过快慢指针找出中间结点，接着把后半段链表进行翻转，然后把前后两段链表进行比较。

## 算法

### 两数之和

遍历数组，把数组的值当作`hash`的键，把数组的下标当作`hash`的值，边存之前的值，边计算当前数的补，只要`hash`表中存在当前数的补，就返回数组的下标即可。

### 两个栈实现队列

一个栈（`stack1`）负责入队，一个栈(`stack2`)负责出队

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
function isObject(obj){
    return typeof obj==='object'&&obj!=null
}
function deepClone(obj,map = new WeakMap()){
    if(!isObject(obj)){
        return obj;
    }
    if(map.has(obj)){
        return map.get(obj);
    }
    var target = Array.isArray(obj)?[]:{};
    map.set(obj,target)
    for(let item in obj){
        target[item] = isObject(obj[item])?deepClone(obj[item],map):obj[item];
    }
    return target;
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

### `instanceOf`

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
function getParams(data) {
  let arr = []
  for (let i in data) {
    arr.push(encodeURIComponent(i) + '=' + encodeURIComponent(data[i]))
  }
  return arr.join('&')
}
function Ajax(options = {}) {
  let {type='GET',async=true,timeout=8000,data,url} = options;
  const xhr = new XMLHttpRequest();
  xhr.onreadystatechange = function(e) {
    if (xhr.readyState === 4) {
      if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
        console.log(xhr.responseText)
      }
    }
  }
  xhr.timeout = timeout
  xhr.ontimeout = function(e) {
    xhr.abort()
  }
  if (type === 'GET') {
    xhr.open(
      'GET',
      url + '?' + getParams(data),
      async
    )
    xhr.send(null)
  } else if (type === 'POST') {
    xhr.open('POST', url, async)
    xhr.setRequestHeader(
      'Content-Type',
      'application/x-www-form-urlencoded'
    )
    xhr.send(options.data)
  }else{
      return;
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

### 获取URL参数

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

## 设计模式

### `EventEmit`

发布订阅者模式是一个消息范式，发布者把所有的消息分成类别，不需要了解订阅者的存在，订阅者只需要表达自己对某些消息的兴趣，也不需要去了解发布者的存在。

```js
class EventEmit {
  //中介对象，以键值对的形式存储存事件以及事件的回调
  constructor() {
    this._eventpool = {}
  }
  //订阅者用on来订阅事件的回调
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
  //发布者用emit执行事件的所有回调
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

### 单例模式

一个类就只有一个实例对象

- 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于重量级的对象来说很重要
- 因为不需要频繁创建对象，减轻了`GC`的压力

缺点：复杂的单例模式需要考虑线程安全等并发问题

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