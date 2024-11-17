### method

在`vue`中`method`就是**一个`function`域**，可以定义方法来进行属性的修改，或者返回。

```vue
<template>
    <p>{{msg}}</p>
    <button v-on:click="reverseMessage">逆转消息</button>
</template>
export default {
  name: 'Home',
  data: function () {
    return {
      msg: 'this msg for vue.js!!!',
    }
  },
  methods: {
    reverseMessage: function () {
      this.msg = this.msg.split('').reverse().join('')
      return this.msg
    }
  },
 ......
```

### computed

`methods`和`computed`都具备执行函数，返回值的功能，但是`computed`是基于它的依赖进行缓存的。如果依赖没有发生改变，`computed`会立即返回之前的计算结果，而不会再次执行函数。

而且`computed`中的并不是方法，而是属性的封装，所以我们不需要用()进行执行，只要属性的值不发生改变，`computed`的值就不会发生调用。

在界面中使用大量的逻辑表达式处理数据时，会影响页面的可维护性，所以`computed`出现了。`computed`是计算属性，它会监听相关依赖的变化，**只有依赖发生变化他才会重新求值**。且**不能在组件的`props`和`data`中定义**，否则会报错。

```vue
<template>
  <p>{{fullName}}</p>
</template>
export default {
  name: 'Home',
  data: {
        firstName: 'Xiao',
        lastName: 'Ming'
    },
    computed: {
        fullName: function () {
            return this.firstName + ' ' + this.lastName
        }
    }
 ......

```
### watch

`watch`是用来监听 `Vue` 上数据的变化，当需要在数据变化时**执行异步或开销较大的操作时**使用。`watch`还有`immediate`和`deep`两个属性，`immediate`表示初始化的时候`watch`可以立即执行，`deep`设置为`true`可以对对象进行一个**深度监听**，监听对象内属性的变化，`deep`属性会对对象进行一个深度的遍历，**在对象的每一层部署一个监听器，从而达到深度监听的效果**

```javascript
watch: {
    obj: {
      handler(val) {
       console.log('obj.a changed')
      },
      immediate: true，
      deep: true
    }
  }
```

但是给每一层都加上监听器，其实会加大性能开销，所以我们可以用字符串的形式来优化，比如我们要监听`obj.a`

```javascript
  watch: {
    'obj.a': {
      handler(val) {
       console.log('obj.a changed')
      },
      immediate: true
      // deep: true
    }
  }
```

直到遇到`'obj.a'`属性，才会给该属性设置监听函数，提高性能。

### `computed`和`watch`用法异同

它们其实都是`vue`对监听器的实现，只不过**`computed`具有缓存性，只有当依赖的数据发生变化才会进行重新的计算，`watch`主要用来观测某个值的变化，调用`handle`去完成一段开销较大的复杂业务逻辑**。

`computed` 是计算一个新的属性，并将该属性挂载到`Vue`实例上，而 `watch` 是监听已经存在且已挂载到 `Vue`实例上的数据，所以用 `watch` 同样可以监听 `computed` 计算属性的变化（其它还有 `data`、`props`）

从使用场景上说，`computed` 适用一个数据被多个数据影响，而 `watch` 适用一个数据影响多个数据

举个具体的使用例子：

- `computed`：一个属性上需要进行许多的操作，操作后返回。

- `watch`：一个属性上