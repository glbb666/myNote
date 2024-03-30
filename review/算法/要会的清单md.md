## 二叉树

### 二叉树深度遍历（递归）

```js
var deepth = function(root) {
  if (root === null) {
    return 0
  } else {
    let lH = deepth(root.left),
    let rH = deepth(root.right)
    return Math.max(lH, rH) + 1
  }
}
```

### 二叉树先序遍历：根左右

先中后序表示的是根的位置，先序中序后序都是**非递归深度遍历**，用的是栈

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

二叉搜索树一般都是用中序遍历，这样能够给出比较准确的结果。

```js
function midOrder(root) {
  let stack = []
  let res = []
  let p = root
  while (stack.length|| p) {
    if (p) {
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

可以用 `unshift`把结果推进 `result`，从左右根变成，根右左。

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

- 不分行从上到下打印二叉树

```js
function BFS(root) {
  let queue = []
  let result = []
  let p = root
  queue.push(p)
  while (queue.length) {
    p = queue.shift()
    result.push(p.data)
    if (p.left)queue.push(p.left)
    if (p.right) queue.push(p.right)
  }
  return result;
}
```

- 分行从上到下打印二叉树

思想：用两个值记录本行未打印结点，和下一行将打印结点。当本行未打印结点等于0时。需要进行换行，做法是：把本行未打印结点的值设置为下一行将打印结点的值，把下一行将打印结点的值设为0。

```javascript
var levelOrder = function(root) {
    if(root===null){
        return [];
    }
    let queue = [root];
    let result = [];
    let count = 1;
    let nextCount = 0;
    let row = []
    while(queue.length){
        let p = queue.shift();
        count--;
        row.push(p.val);
        if(p.left){
            queue.push(p.left);
            nextCount++;
        }
        if(p.right){
            queue.push(p.right);
            nextCount++;
        }
        if(count===0){
            result.push(row);
            row = [];
            count = nextCount;
            nextCount = 0;
        }
    }
    return result
};
```

- 之字形打印二叉树

思路：用一个odd值标志奇数行和偶数行，如果是奇数行就用 `push`推入值，如果是偶数行，就用 `unshift`推入值，奇数行和偶数行在换行的时候进行切换。

```javascript
var levelOrder = function(root) {
    if(root===null){
        return [];
    }
    let queue = [root];
    let result = [];
    let row = [];
    let count = 1;
    let nextCount = 0;
    let odd = true;
    while(queue.length){
        let p = queue.shift();
        count--;
        if(odd){
            row.push(p.val)
        }else{
            row.unshift(p.val)
        }
        if(p.left){
            queue.push(p.left);
            nextCount++;
        }
        if(p.right){
            queue.push(p.right);
            nextCount++;
        }
        if(count===0){
            result.push(row)
            row = []
            count = nextCount
            nextCount = 0
            odd = !odd;
        }
    }
    return result;
};
```

### 二叉树的创建

#### 递归

根据前序遍历和中序遍历的结果创建二叉树

思想：前序遍历的顺序是，根左右。中序遍历的顺序是：左根右。

因此前序遍历的第一个结点是根节点，中序遍历中根节点两边是左子树和右子树。

而我们又可以根据根节点的序列号，找到前序遍历中的左子树（根节点之后的index个结点）和右子树（左子树后的结点）。

```javascript
var buildTree = function(preorder, inorder) {
    if(preorder.length===0){
        return null;
    }
    let root = preorder.shift();
    let node = new TreeNode(root);
    let rootIndex = inorder.indexOf(root);
    let preLeft = preorder.slice(0,rootIndex);
    let preRight = preorder.slice(rootIndex);
    let inLeft = inorder.slice(0,rootIndex);
    let inRight = inorder.slice(rootIndex+1);
    node.left = buildTree(preLeft,inLeft);
    node.right = buildTree(preRight,inRight);
    return node;
};
```

### 二叉树的实际应用

1. 哈夫曼编码，如果带权路径长度达到最小，这样的二叉树称为最优二叉树，可以用来进行数据压缩。
2. B+树在文件系统中的目录会使用到
3. 红黑树的查找/删除/插入，时间复杂度均为 `O(logn)`

## 排序

### 三种简单排序（冒泡，插入，选择）

- 时间复杂度：`O(n^2)`
- 空间复杂度： `O(1)`
- 稳定性：冒泡和插入稳定，选择不稳定（记忆：人的选择就是不一定的）

### 冒泡

- 外层控制轮次，表示第几轮
- 内层控制交换，若 `arr[j]>arr[j+1]`则交换 `arr[j]`和 `arr[j+1]`，`j`代表被排序的前一个值的下标

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

### 归并排序

归并排序是建立在归并操作上的一种有效的排序算法，该算法是采用**分治法**的一个非常典型的应用。将已有的子序列合并，得到完全有序的序列，即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为二路归并。

- 时间复杂度 `O(nlog(n))`
- 空间复杂度 `O(n)`
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

### 删除链表给定 `val`节点

思路：把头节点当作哑结点后继结点，这样就可以进行删除操作了。注意的是，如果后续结点被删除，则不需要后移，因为后续结点已经不是之前的那一个结点了。

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

- `hash`表，不存在就把结点保存，存在说明对同一个结点存在两次引用，存在环形链表

```javascript
var hasCycle = function(head) {
    var set = new Set();
    var p = head;
    while(p){
        if(set.has(p)){
            return true
        }else{
            set.add(p)
            p = p.next;
        }
    }
    return false;
};
```

- 快慢指针，如果是环形，总是会相遇的

```javascript
var hasCycle = function(head) {
    if(!head||!head.next){
        return false;
    }
    let slow = head;
    let quick = head.next;
    while(quick&&quick.next){
        if(quick===slow){
            return true;
        }
        slow = slow.next;
        quick = quick.next.next;
    }
    return false;
};
```

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

### 回文链表（时间复杂度 `O(n)`，空间复杂度 `O(1)`）

首先通过快慢指针找出中间结点，接着把后半段链表进行翻转，然后把前后两段链表进行比较。

### 插入排序

```js
var insertionSortList = function(head) {
    if(head===null || head.next === null){
        return head;
    }
    var temp = new ListNode(-Infinity);
    temp.next = head;
    var beforeInsert = head;
    while(beforeInsert.next){
        if(beforeInsert.next.val>=beforeInsert.val){
            beforeInsert = beforeInsert.next;
            continue;
        }
        var p = temp;
        while(beforeInsert.next.val>=p.next.val&&p !== beforeInsert){
            p = p.next;
        }
        if(p!==beforeInsert){
            var insertNode = beforeInsert.next;
            beforeInsert.next = insertNode.next;
            insertNode.next = p.next;
            p.next = insertNode;
        }else{
            beforeInsert = beforeInsert.next;
        }
    }
    return temp.next;
};
```

### 两数相加

哑巴结点+记录进位

## 算法

### 两数之和

遍历数组，把数组的值当作 `hash`的键，把数组的下标当作 `hash`的值，边存之前的值，边计算当前数的补，只要 `hash`表中存在当前数的补，就返回数组的下标即可。

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

### call&bind&apply

```js
Function.prototype.call2 = function(context,...args){
	context =context||window;
	context.fn = this;
	var result =context.fn(...args）;
	delete context.fn;
	return result;
}
```

```js
Function.prototype.apply2 = function(context,...args){
	context = context||window;
	context.fn = this;
	var result = context.fn(...args);
	delete context.fn;
	return result;
}
```

```js
Function.prototype.bind2 = function(context,...args){
    	if(typeof this!=='function'){
           throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
        }
        var self = this;
        var fNOP = function(){}
        var fBound = function(){
            var bindArgs = [...args,...arguments];
            var _this = this instanceof self?this:context;
            _this.fn = self;
            var result = _this.fn(...bindArgs);
            delete _this.fn;
            return result;
        }
        fNOP.prototype = this.prototype;
        fBound.prototype = new fNOP();
        return fBound;
}
```

### new（9.8 19:23)

```javascript
function new2(Constructor,...args){
   const obj = new Object();
   obj.__proto__ = Constructor.prototype;
   // 利用apply把构造函数内部的this指向更改为实例
   const ret = Constructor.apply(obj,args);
   // 如果构造函数返回引用类型，则不会new一个新的实例，而是返回该引用类型
   // 如果构造函数返回一个基本类型，则返回new的新实例
   return typeof ret==='object' || typeof ret=== 'function'?ret:obj;
}
```

### 节流&防抖

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

柯里化是一种**将使用多个参数的一个函数转换成一系列使用一个参数的函数**的技术。

换言之，用闭包把参数保存起来，当参数的数量足够执行函数了，就开始执行函数。

它可以减少相同参数的重复传递

```js
function curry(fn, ...args) {
    return function(){
        var _args = [...args,...arguments];
        return _args.length<fn.length?curry.call(this, fn, ..._args):fn.apply(this,_args);
    }
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

### `JSONP`

```js
<script type="text/javascript">
var url = 'http://localhost:8080/';
function jsonp(obj,time,url,success,error){
    //基本类型
    url+=url.includes('?')?'?':'&';
    for(let item in obj){
        url+=encodeURIComponent(item)+'='+encodeURIComponent(obj[item])+'&';
    }
    var callBackName =('_jsonp'+Math.random()).replace('.','')
    url+='callback='+callBackName;
    var scriptEle = document.createElement('script');
    scriptEle.src = url;
    window[callBackName] = function(data){
       	//这里是对数据进行处理
        success()
        window.clearTimeout(timer);//清除定时器
        window[callBackName] = null;//把回调函数解除引用
        document.head.removeChild(scriptEle);
    }
    //出错处理
    scriptEle.onerror = function(){
       error();
       window.clearTimeout(timer);//清除定时器
       window[callBackName] = null;//把回调函数解除引用
       document.head.removeChild(scriptEle);
    }
    //超时处理
    var timer = window.setTimeout(function(){
        error();
        document.head.removeChild(scriptEle);//移除script标签
		window[callBackName] = null;//把回调函数解除引用
    },time)
    document.head.appendChild(scriptEle);
}
jsonp({name:'dd'},5000,url)
</script>
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

### 图片懒加载

```js
~function(){
    var img = document.getElementsByTagName('img');
    var n = 0;
    lazyImg()
    window.addEventListener('scroll', throttle(lazyImg,1000));
    function lazyImg(){
        for(let i = n;i<img.length;i++){
            var {offsetTop} = img[i];
            var {scrollTop,clientHeight} = document.documentElement;
            if(img[i].getAttribute("src")==="../image/default.jpg"&&offsetTop<scrollTop+clientHeight){
                img[i].src = img[i].getAttribute('data-src');
                n = i+1;
            }
        }
    }
}()
function throttle(fn,wait){
    var flag = true;
    return function(...args){
        var context = this;
        if(!flag)return;
        flag = false;
        timer = setTimeout(() => {
            console.log(this===context)
                fn.call(context,...args);
                flag = true;
        }, wait);
    }
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

### sleep

`promise`

```javascript
function sleep(timeout){
	return new Promise(function(resolve){
		setTimeout(resolve,timeout)
	})
}
sleep(1000).then(()=>{
	console.log(1);
})
```

`async`

```javascript
function sleep(timeout){
	return new Promise(function(resolve,reject){
			setTimeout(function(){
				resolve()
			},timeout)
	})
}
async function output(timeout){
	await sleep(timeout);
    console.log(1);
}
```

`ES5`

```javascript
function sleep(callBack,time){
	setTimeout(callBack,time);
}
function output(){
  console.log(1);
}
sleep(output,1000);
```

## 数组操作

### `reduce`实现 `flat`

```js
Array.prototype.flat = function(){
  return this.reduce(
    (p,c)=>(Array.isAr ray(c)?last.concat(c.flat()):last.concat(c)),[])
}
```

### `reduce`实现 `map`

```js
Array.prototype.map1 = function(fn){
    return this.reduce((p,c)=>{
        p.push(fn(c))
        return p;
    },[])
}
```

### `reduce`实现 `flatMap`

```js
Array.prototype.flatMap2 = function(fn){
    return this.reduce(
    (p,c)=>Array.isArray(c)?p.concat(c.flatMap2(fn)):p.concat(fn(c)),[])
}
```

### `reduce`实现 `filter`

```js
Array.prototype.filter1 = function(fn){
    return this.reduce((p,c)=>{
        fn(c)?p.push(c):null;
        return p;
    },[])
}
```

### 数组去重

双重 `for`循环

```javascript
Array.prototype.unique = function(){
    let result = [];
    let repeat = false;
    for(let i = 0;i<this.length;i++){
        for(let j = 0;j<result.length;j++){
         	if(this[i]===result[j]){
				repeat = true;
                break;
            }   
        }
        if(repeat){
        	repeat.push(this.[i]);
        }
    }
    return result;
}
```

`indexOf`

```javascript
Array.prototype.unique = function(){
	let result = [];
	for(let i = 0,len = this.length;i<len;i++){
		if(result.indexOf(this[i])===-1){
			result.push(this[i]);
		}
	}
	return result;
}
```

排序后去重

```javascript
Array.prototype.unique = function(){
	arr.sort();
	let result = [];
	for(let i = 0,len = this.length;i<len;i++){
		if(this[i]!=result[result.length-1]){
			result.push(this[i]);
		}
	}
}
```

`Array.prototype.includes()`

```javascript
Array.prototype.unique = function(){
	let result = [];
	this.forEach(item=>{
    	!result.includes(item)?result.push(item):null;        
    })
}
```

`Array.prototype.reduce()`

```javascript
Array.prototype.unique = function(){
    return this.sort().reduce((p,v)=>{
        if(p.length===0||p[p.length-1]!==v){
            p.push(v);
        }
        return p;
    },[])
}
```

`map`

```javascript
Array.prototype.unique = function(){
	var map = new Map();
	return this.filter(item=>!map.has(item)?map.set(item,1):null)
}
```

`set`

```javascript
Array.prototype.unique = function(){
	return Array.from(new Set(this));
}
```

### 数组乱序

```js
function shuffle(a) {
    for (let i = a.length; i>0; i--) {
        let j = Math.floor(Math.random() * i);
        [a[i - 1], a[j]] = [a[j], a[i - 1]];
    }
    return a;
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
  off(event,fn) {
    //解除
    if (this._eventpool[event]&&this._eventpool[event].length!=0) {
        if(!fn){
            delete this._eventpool[event]
        }else{
            //如果相同就从缓存列表中删掉
            this._eventpool[event].forEach((cb,index)=>{
                if(cb===fn){
                    this._eventpool[event].splice(index,1);
                }
            })
        }
    }
  }
  //发布者用emit执行事件的所有回调
  emit(event, ...args) {
    //执行全部回调
    if (this._eventpool[event]&&this._eventpool[event.length]!==0) {
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
