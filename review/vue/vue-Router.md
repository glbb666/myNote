路由的本质是建立并管理`url`和相应组件之间的映射关系。

### 两种模式

#### 1. hash 模式

**Hash模式就是通过改变#后面的hash值,实现浏览器渲染指定的组件**。因为hash 值的变化，并不会导致浏览器向服务器发出请求，也就不会刷新页面。所以我们可以通过监听`window.hashchange` 这个事件进行路由跳转，更新页面部分内容。

```javascript
function matchAndUpdate () {
   // todo 匹配 hash 做 dom 更新操作
}
window.addEventListener('hashchange', matchAndUpdate)
```
但是由于`#`不够美观，所以又有了`history`模式。

#### 2. history 模式

通过`pushState` 和 `replaceState`分别对当前历史记录进行添加和修改，不会发送请求。`popstate`事件，会在用户点击浏览器倒退按钮和前进按钮，或者调用 `history` 的`back`、`forward`、`go`方法时才会触发。

当用户刷新页面之类的操作时，浏览器还是会给服务器发送请求。所以这个`history 模式`实现需要服务器的支持，如果`url`匹配不到任何静态资源，就重定向到根页面。

```javascript
function matchAndUpdate () {
   // todo 匹配路径 做 dom 更新操作
}

window.addEventListener('popstate', matchAndUpdate)
```

### `Vue-router`使用
模板上

```html
<v-link to=""></v-link><!--映射路由,to属性指定目标地址.-->
<router-view>写在这里,即router-view里的内容是不会显示在页面上的</router-view>
<!--标签内渲染你路由匹配到的视图组件，支持嵌套router-view，支持多个router-view-->
```

这个是在`vue-cli`脚手架中
```javascript
import VueRouter from 'vue-router'//引入
Vue.use(VueRouter)//注入
//创建路由器实例
const router = new VueRouter({
  mode: 'history',
  routes: [{
      name:'root'//路由名,
      path:'/',//路由路径
      component:home//路由映射的组件
      redirect:'/home'//重定向
      children:[{}]//利用children可以进行嵌套路由
  }]
})
new Vue({
  router//挂载
  ...
})
```

#### 🧡 `$router`和`$route`的区别：

`$router`是指整个路由实例，你可以操纵整个路由，通过`$router.push`往其中添加任意的路由对象

`$route`：是指路由实例`$router`跳转到的当前路由对象

`$router`可以包含多个`$route`，它们是父子包含关系

### `Vue-router`传参（动态路由匹配）

动态路由匹配本质上就是通过`url`进行传参。

#### 属性

**$route.path** 类型: `string` 字符串，对应当前路由的路径，总是解析为绝对路径，如 `"/foo/bar"`。

**$route.params** 类型: `Object` 一个 key/value 对象，包含了动态片段和全匹配片段，如果没有路由参数，就是一个空对象。

**$route.query** 类型: `Object` 一个 key/value 对象，表示 URL 查询参数。例如，对于路径 `/foo?user=1`，则有 `$route.query.user == 1`，如果没有查询参数，则是个空对象。

**$route .name** 当前路由的名称，如果有的话。**这里建议最好给每个路由对象命名,方便以后编程式导航.不过记住name必须唯一!**

**$route.hash** 类型: `string` 当前路由的 hash 值 (带 `#`) ，如果没有 hash 值，则为空字符串。

**$route.fullPath** 类型: `string` 完成解析后的 URL，包含查询参数和 hash 的完整路径。

**$route.matched** 类型: `Array<RouteRecord>` 一个数组，包含当前路由的所有嵌套路径片段的**路由记录** 。路由记录就是 `routes` 配置数组中的对象副本 (还有在 `children` 数组)。

**$route.redirectedFrom** 如果存在重定向，即为重定向来源的路由的名字。

#### `params`

例如我们有一个组件，对于所有`ID`各不相同的用户，都要使用这个组件来渲染。那么，我们可以使用动态路径参数来达到这个效果

```javascript
const router = new VueRouter({
  routes: [{
      //动态路径参数 以冒号开头
        path: '/user/:id', 
     	component: User 
    }]
})
```

这样`/user/foo`和`/user/bar`都将映射到相同的路由

当匹配到一个路由时，参数就会被设置到`this.$route.params`。例如`/user/foo`的`this.$route.params.id`就为`foo`

这里以官方的表格示例进行展示

| 模式                          | 匹配路径            | $route.params                        |
| ----------------------------- | ------------------- | ------------------------------------ |
| /user/:username               | /user/evan          | `{ username: 'evan' }`               |
| /user/:username/post/:post_id | /user/evan/post/123 | `{ username: 'evan', post_id: 123 }` |

有时候，同一个路径可以匹配多个路由，此时匹配的优先级就按照路由的定义顺序，越先定义的优先级越高。

##### 组件更新

由于路由参数对组件实例是复用的，例如：`/user/foo` 和 `/user/bar`在使用路由参数时，复用了`User`组件。此时组件的生命周期钩子不会再被调用。如果你在想路径切换时，进行一些初始化操作，可以用以下三种解决办法

- `key`：给`router-view`设置key值为当前时间戳，缺点是如果`router-view`存在嵌套路由，点击子路由会造成组件刷新，且子路由不会携带参数，就会出现获取不到初始化数据的情况。
- 在组件内`watch`路由：
- 导航守卫

#### `query`

### 命名视图

模板上

```

```



这个是在`vue-cli`脚手架中

```javascript
const router = new VueRouter({
  ...
  routes: [{
      ...
      component:{
      	'default':home,
      	'a':home1,
      	'b':home2
      }
  }]
})
```



### 路由守卫

### `Vue-router`的实现原理