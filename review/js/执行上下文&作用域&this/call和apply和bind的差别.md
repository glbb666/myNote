### `call`

* **作用** ：立即调用函数，并指定 `this` 的值。
* **语法** ：`func.call(thisArg, arg1, arg2, ...)`
* 立即执行函数。
* 比 `apply`的性能要好，`call`传入参数的格式正是函数内部所需要的格式

### `apply`

* **作用** ：立即调用函数，并指定 `this` 的值，但参数是以数组的形式传递。
* **语法** ：`func.apply(thisArg, [arg1, arg2, ...])`
* 立即执行函数。

### `bind`

* **作用** ：创建一个新的函数，该函数将 `this` 绑定到指定的值，并且可以预先设置部分参数。
* **语法** ：`func.bind(thisArg, arg1, arg2, ...)`
* **特点** ：
  * 后面的参数列表是预先设置的参数。
  * 返回一个新的函数，该函数的 `this` 已绑定，并且可以使用预设的参数。
  * 函数不会立即执行，而是可以在之后的时间点调用。

#### 返回的函数使用new

通过bind方法反悔的函数内部存在一个绑定记录，包含绑定的this和预设的参数。

但是 `bind`方法返回的函数如果使用 `new`，通过 `bind`绑定的 `this`失效，因为 `new`会将 `this`指向初始化为构造函数创建出的实例。

#### 为什么函数中的 `this` 只会由第一次 `bind` 的对象决定

根据 ECMAScript 规范，`bind` 方法返回的函数是一个拥有固定 `this` 的函数，这个固定的 `this` 无法被重新绑定，即使使用 `bind` 方法再次绑定也是无效的。

```javascript
let a = {}
let fn = function () { console.log(this) }
fn.bind().bind(a)()//window
```

因此结果永远为 `window`
