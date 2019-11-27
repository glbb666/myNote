## Set

### 基本用法

 `Set`类似于数组，但是**成员的值都是唯一的,可以用来数组去重**。

`Set`函数可以接受一个`Set`函数接受**数组**或**<font color='red'>类似数组的对象</font>**（或者具有 `iterable` 接口的其他数据结构）作为参数，用来初始化。

```javascript
// 例一
const set = new Set([1, 2, 3, 4, 4]);
[...set]// [1, 2, 3, 4]
set.add(/*要增加的值*/) //返回set
set.has(/*要检测的值*/) //返回布尔值
set.delete(/*要删除的值*/)//返回布尔值
set.size //元素数量
set.clear() //删除set中的所有值 无返回
```

向 Set 加入值的时候，**不会发生类型转换**。Set 内部判断两个值是否不同，使用的算法叫做“Same-value-zero equality”，它和（`===`），区别是向 Set 加入值时认为`NaN`等于自身

另外，两个对象总是不相等的。

### 遍历操作

Set 结构的实例有四个遍历方法，可以用于遍历成员。

- `Set.prototype.keys()`：返回键名的遍历器，`keys`方法和`values`方法的行为完全一致
- `Set.prototype.values()`：返回键值的遍历器，是`Set`的默认遍历器生成函数，这意味着，可以省略`values`方法，直接用`for...of`循环遍历 Set。
- `Set.prototype.entries()`：返回键值对的遍历器
- `Set.prototype.forEach()`：使用回调函数遍历每个成员

需要特别指出的是，`Set`的遍历顺序就是插入顺序。这个特性有时非常有用，比如使用 Set 保存一个回调函数列表，调用时就能保证按照添加顺序调用。

**forEach()**

Set 结构的实例与数组一样，也拥有`forEach`方法，差别是第一个参数与第二个参数的值永远都是一样的

另外，`forEach`方法还可以有第二个参数，表示绑定处理函数内部的`this`对象。

### Set和Object/Array的区别

- 在`Set`中，不会对值进行隐式转换，数字5和字符串'5'、{}和{}都是作为两个截然不同并且独立的存在
- Set和数组一样都有`forEach`，唯一的区别就是`forEach`的第一个参数`callback`中的前两个值相同，可以理解成`Set`的`key`和`value`是一个东西。
- 在数组或者对象中，可以直接通过key（数组是index）来访问元素，但是`Set`中不行，如果需要可以将`Set`转换为数组(通过`Array.from(set)`或者`[...set]`)。

## WeakSet

### 含义

与 Set 有两个区别。

- `WeakSet` 的**成员只能是对象**

- `WeakSet` 中的对象**都是弱引用**，即垃圾回收机制不考虑 `WeakSet` 对该对象的引用 
- 不支持`size forEach clear`等属性，只有`add has delete`
- 不可遍历，是因为成员都是弱引用，随时可能消失，遍历机制无法保证成员的存在

这是因为垃圾回收机制依赖引用计数，如果一个值的引用次数不为`0`，垃圾回收机制就不会释放这块内存。结束使用该值之后，有时会忘记取消引用，导致内存无法释放，进而可能会引发内存泄漏。`WeakSet` 里面的引用，都不计入垃圾回收机制，所以就不存在这个问题。因此，`WeakSet` 适合临时存放一组对象，以及存放跟对象绑定的信息。只要这些对象在外部消失，它在 `WeakSet` 里面的引用就会自动消失。

由于上面这个特点，`WeakSet` 的成员是不适合引用的，因为它会随时消失。另外，由于 `WeakSet` 内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行是不可预测的，因此 `ES6` 规定 `WeakSet` 不可遍历。

## Map

### 含义和基本用法

`Map`和`JavaScript` 的对象（`Object`）类似，但是`Object` 结构提供了**“字符串—值”**的对应，`Map` 结构提供了**“值—值”的对应**

作为构造函数，`Map` 可以接受一个数组作为参数。该数组的成员是一个个**表示键值对的数组**。

```javascript
const map = new Map([
  ['name', '张三'],
  ['title', 'Author']
]);
map.set([key, value]) //添加
map.get(key) //获取
map.has(key) //检测
map.delete(key) //删除
map.clear() //清空
map.size //返回元素个数
```

任何`(1)具有 Iterator 接口`、`(2)每个成员都是一个双元素的数组`的数据结构（详见《Iterator》一章）都可以当作`Map`构造函数的参数。这就是说，`Set`和`Map`都可以用来生成新的 `Map`。

**虽然`NaN`不严格相等于自身，但 `Map` 将其视为同一个键**。

## `WeakMap`

注意，

```javascript
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};

wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}
```

上面代码中，键值`obj`是正常引用。所以，即使在 WeakMap 外部消除了`obj`的引用，WeakMap 内部的引用依然存在。

### WeakMap 的语法

`WeakMap` 与 `Map` 在区别

- `key`只能是**对象**，但是`value`没有这个限制。

- 作为`key`的对象同`WeakSet`，是弱引用。

- 不支持`forEach、size、clear`等属性，只支持`set、get、has、delete`。

  🌟注意：`WeakMap` 弱引用的只是键名，而不是键值。键值依然是正常引用。

- `WeakMap`是**不可遍历**对象。

```javascript
const wm = new WeakMap();

// size、forEach、clear 方法都不存在
wm.size // undefined
wm.forEach // undefined
wm.clear // undefined
```

### Set和Map的区别

一般来说，Set集合常被用于检查对象中是否存在某个key，而Map用户获取已存的信息

### 和数组对象相比，优点有哪些？

- 判断是否存在，不需要担心值为false，也不担心in会遍历原型链上的属性等等
- 如果某个元素不存在时，直接返回undefined，而数组、对象还会遍历原型链浪费性能
- key能够是非基本类型的值