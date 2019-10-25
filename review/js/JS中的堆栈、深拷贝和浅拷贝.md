# JavaScript中的浅拷贝与深拷贝
## 一：什么是堆栈？
  我们都知道：在计算机领域中，堆栈是两种数据结构，它们只能在一端(称为栈顶(top))对数据项进行插入和删除。

> 堆：队列优先,先进先出；由操作系统自动分配释放 ，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。
> 栈：先进后出；动态分配的空间 一般由程序员分配释放， 若程序员不释放，程序结束时可能由OS回收，分配方式倒是类似于链表。

以上都属于计算机基础部分，在此都不详细赘述了，下面我们联系JavaScript来剖析一下堆栈。

## 二：JavaScript中的基本类型和引用类型与堆栈有什么联系？

> JavaScript的数据类型分为两大种：

1. 基本类型：Undefined、Null、Boolean、Number 、String、BigInt、Symbol这7种类基本数据类型可以直接访问，他们是按照值进行分配的，存放在栈(stack)内存中的简单数据段，数据大小确定，内存空间大小可以分配。
2. 引用类型：即存放在堆(heap)内存中的对象，变量实际保存的是一个指针，这个指针指向另一个位置。
  以上我们知道了什么是堆栈，和JavaScript的数据类型，下面我们根据js的数据类型来说明一下他们的拷贝情况：
```js
var obj1 = {name:'bangbang',age:18};
var b = obj1;
var c = obj1.age;

console.log(b.name); //bangbang
console.log(c);      //18
//改变b和c的值
b.name = 'yanniu';
c = 22;
console.log(obj1.name);     //yanniu
console.log(obj1.age);       //18
```
  以上看出：当我们改变b的数据的时候，我们看到了obj1.name的数据也在改变,但是我们改变c的数据的时候发现，obj1.age的值没有变化，这说明了：b和obj1变量操作的是同一个对象，c和obj1完全独立的。图示如下：

![这里写图片描述](https://img-blog.csdn.net/20161022234725144)


## 三：什么是浅拷贝？
  创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。**如果属性是基本类型，拷贝的就是基本类型的值**，如果属性**是引用类型**，**拷贝的就是内存地址** ，所以如果其中一个对象改变了这个地址，就会影响到另一个对象。

```js
function cloneShallow(source) {
    var target = {};
    for (var key in source) {
     	//for in 在数组和对象中都能用作遍历
        if (Object.prototype.hasOwnProperty.call(source, key)) {
            target[key] = source[key];
        }
    }
    return target;
}
```

创建一个新的对象，遍历需要克隆的对象，将需要克隆对象的属性依次添加到新对象上，返回

es6的**Object.assign**属性也可以进行浅拷贝

```javascript
var target = {};
Object.assign(target,source);
```



## 四：什么是深度拷贝？
将一个对象从内存中完整的拷贝一份出来,**从堆内存中开辟一个新的区域存放新对象**,且修改新对象不会影响原对象

- 方法一：

```js
    function deepCopy(obj) {
        return JSON.parse(JSON.stringify(obj));
    }
```

缺陷: 不能拷贝其他引用类型、拷贝函数、循环引用等情况。例如数组的元素是个对象

- 方法二：
  - 其实深拷贝可以拆分成 2 步，浅拷贝 + 递归
  - 浅拷贝时判断属性值是否是对象，如果是对象就进行递归操作，两个一结合就实现了深拷贝。

```js
// 木易杨
function cloneDeep1(source) {
    var target = {};
    for(var key in source) {
        //用来检测一个对象是否含有特定的自身属性
        if (Object.prototype.hasOwnProperty.call(source, key)) {
            if (typeof source[key] === 'object') {
                target[key] = cloneDeep1(source[key]); // 注意这里
            } else {
                target[key] = source[key];
            }
        }
    }
    return target;
}
```

一个简单的深拷贝就完成了，但是这个实现还存在很多问题。

- 1、没有对传入参数进行校验，传入 `null` 时应该返回 `null` 而不是 `{}`
- 2、对于对象的判断逻辑不严谨，因为 `typeof null === 'object'`
- 3、没有考虑数组的兼容

## 第二步：拷贝数组

我们来看下对于对象的判断

```js
typeof null //"object"
typeof {} //"object"
typeof [] //"object"
typeof function foo(){} //"function" (特殊情况)
```

因为[]和{}需要放在下面单独判断，所以我们只需要用isObject<font color='red'>筛除基本类型</font>即可

```js
function isObject(obj) {
	return typeof obj === 'object' && obj != null;
}
```

所以兼容数组的写法如下。

```js
// 木易杨
function cloneDeep2(source) {
    if (!isObject(source)) return source;//筛除基本类型
    var target = Array.isArray(source) ? [] : {};
    for(var key in source) {
        if (Object.prototype.hasOwnProperty.call(source, key)) {
            target[key]=isObject(source[key]？cloneDeep2(source[key]):source[key]；
			// 注意这里   
        }
    }
    return target;
}
```

#### 考虑循环引用（如果不解决循环引用问题，会出现栈溢出）

> 循环引用：对象的属性间接或直接的引用了自身的情况
> **解决循环引用**：额外开辟一个存储空间，来存储当前对象和拷贝对象的对应关系，当需要拷贝当前对象时，先去存储空间中找，有没有拷贝过这个对象，如果有的话直接返回，如果没有的话继续拷贝，这样就巧妙化解的循环引用的问题
>
> 假如对象A和B循环应用，那么B会是A的一个属性，B会是A的一个属性，每次拷贝时，看看要拷贝的值在map中存不存在，存在的话，就直接返回，不存在的话，就把要拷贝的值存入map中
```js
function cloneDeep3(source, hash = new WeakMap()) {
    if (!isObject(source)) return source; //筛除基本值
    if (hash.has(source)) return hash.get(source); //看看该对象在不在哈希表中，在表中直接取值返回即可
    //对象不存在，再进行拷贝
    var target = Array.isArray(source) ? [] : {};
    hash.set(source, target); //把source作为键存入map，map的键名不重复
    for(var key in source) {
        if (Object.prototype.hasOwnProperty.call(source, key)) {
          target=isObject(source[key])?cloneDeep3(source[key], hash):source[key];
        }
    }
    return target;
}
```
### 为什么用WeakMap?
- <font color='red'>解决同名属性碰撞问题：</font>WeakMap的键是和内存地址绑定的，只要内存地址不一样，就视为两个键，这样就能
- <font color='red'>解决内存泄漏问题</font>`WeakMap`的键名所指向的对象是弱引用的，不计入垃圾回收机制，不造成对对象的引用。这样，当这个对象被删除，其所对应的`WeakMap`记录就自动被移除，不需要手动删除引用，