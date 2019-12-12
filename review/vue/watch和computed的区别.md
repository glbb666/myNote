在界面中使用大量的逻辑表达式处理数据时，会影响页面的可维护性，所以`computed`出现了。`computed`是计算属性，它会监听相关依赖的变化，**只有依赖发生变化他才会重新求值**。且**不能在组件的`props`和`data`中定义**，否则会报错。

```html
<<div id="app">
    <p>{{fullName}}</p>
</div>
```
```javascript
new Vue({
    data: {
        firstName: 'Xiao',
        lastName: 'Ming'
    },
    computed: {
        fullName: function () {
            return this.firstName + ' ' + this.lastName
        }
    }
})
```
`watch`是用来监听 `Vue` 上数据的变化，当需要在数据变化时**执行异步或开销较大的操作时**使用。`watch`还有`immediate`和`deep`两个属性，`immediate`表示初始化的时候`watch`可以立即执行，`deep`设置为`true`可以对对象进行一个深度监听，监听对象内属性的变化，`deep`属性会对对象进行一个深度的遍历，在对象的每一层部署一个监听器，从而达到深度监听的效果

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

直到遇到'obj.a'属性，才会给该属性设置监听函数，提高性能。

## `computed`和`watch`用法异同

它们其实都是`vue`对监听器的实现，只不过**computed主要用来处理同步数据，只有依赖的数据发生变化才会进行重新的计算，watch主要用来观测某个值的变化，调用`handle`去完成一段开销较大的复杂业务逻辑**。能用computed的时候优先用computed，避免了多个数据影响其中某个数据时多次调用watch的尴尬情况。

`computed` 是计算一个新的属性，并将该属性挂载到`Vue`实例上，而 `watch` 是监听已经存在且已挂载到 `Vue`实例上的数据，所以用 `watch` 同样可以监听 `computed` 计算属性的变化（其它还有 `data`、`props`）

`computed` 本质是一个惰性求值的观察者，具有缓存性，只有当依赖变化后，第一次访问  `computed`  属性，才会计算新的值，而 `watch` 则是当数据发生变化便会调用执行函数

从使用场景上说，`computed` 适用一个数据被多个数据影响，而 `watch` 适用一个数据影响多个数据；

