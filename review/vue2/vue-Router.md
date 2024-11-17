[JS路由指南](<https://juejin.im/post/5b82bcfcf265da4345153343#heading-26>)

路由的本质是建立并管理`url`和相应组件之间的映射关系。

### 两种模式

#### 1. `hash` 模式

**`hash`模式就是通过改变#后面的hash值,实现浏览器渲染指定的组件**。因为hash 值的变化，并不会导致浏览器向服务器发出请求，也就不会刷新页面。所以我们可以通过监听`window.hashchange` 这个事件进行路由跳转，更新页面部分内容。

```javascript
function matchAndUpdate () {
   // todo 匹配 hash 做 dom 更新操作
}
window.addEventListener('hashchange', matchAndUpdate)
```
但是由于`#`不够美观，所以又有了`history`模式。

#### 2. `history `模式

通过`pushState` 和 `replaceState`分别对当前历史记录进行添加和修改，不会发送请求。`popstate`事件，会在用户点击浏览器倒退按钮和前进按钮，或者调用 `history` 的`back`、`forward`、`go`方法时才会触发。

当用户刷新页面之类的操作时，浏览器还是会给服务器发送请求。所以这个`history 模式`实现需要服务器的支持，如果`url`匹配不到任何静态资源，就重定向到根页面。

```javascript
function matchAndUpdate () {
   // todo 匹配路径 做 dom 更新操作
}

window.addEventListener('popstate', matchAndUpdate)
```

### `Vue-router`使用
```html
<v-link to=""></v-link><!--映射路由,to属性指定目标地址，-->
<router-view>写在这里,即router-view里的内容是不会显示在页面上的</router-view>
<!--标签内渲染你路由匹配到的视图组件，没有设置name的视图会获得默认名default-->
<router-view name="a"></router-view>
```

```javascript
import VueRouter from 'vue-router'//引入
Vue.use(VueRouter)//注入
//创建路由器实例
const router = new VueRouter({
  mode: 'history',
  routes: [{
      name:'root'//路由名,
      path:'/',//路由路径
      //component:home//路由映射的组件
      component:{
      	'default':home,
        'a':home1
  	  }
      redirect:'/home'//重定向，拦截path，替换url
      children:[{}]//利用children可以进行嵌套路由，
      //alias:'/house'//别名
      alias:['house','hhome']//有多个别名时可以写成数组形式
  }]
})
new Vue({
  router//挂载
  ...
})
```

####  `$router`和`$route`的区别：

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

这样`/user/foo`和`/user/bar`都将映射到相同的路由，当匹配到一个路由时，参数就会被设置到`this.$route.params`。例如`/user/foo`的`this.$route.params.id`就为`foo`

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

`<router-link to="/user?id=foo">foo</router-link>`，`vue-router`会自动将?后的`id=foo`封装进`this.$route.query`里. 此时,在组件里`this.$route.query.id`值为`'foo'`。query也可以通过后面讲到的编程式导航进行传参。

### 编程式导航

#### `router.push`

**向`history`里添加新记录**，编程式导航一般都是用到`router.push`方法，该方法的参数可以是字符串路径，也可以是描述地址的对象

```javascript
this.$router.push('home')//字符串
this.$ruter.push({path:'home'})//对象
this.$router.push({name:'user',params:{userId:2333}})//命名路由
this.$router.push({path:'register',query:{plan:'private'}})
//带查询参数,变成/register?plan=private
```

💚**`path`和`params`是不能同时生效的！**否则`params`会被忽略掉。所以使用对象写法进行`params`传参时，要么就是`path`加冒号`:`要么就是像上例中的'命名路由'。通过`name`和`params`进行传参。**然而`query`却并不受影响,有没有`path`都可以进行传参**

#### `router.replace`

**直接将当前浏览器`history`记录替换掉**，当你不想让用户回退到之前的页面，常用于权限验证，验证后就不让用户回退到登录页重复验证。

#### `router.go(n)`

控制页面前进或后退多少步

实际上不通过`router`配置，也可以用`router-link`上通过`to`使用编程式导航

```javascript
routes:[
	{name:'shotCat',path:'/shotCat', component:shotCat}
]
```
```html
<router-link :to="{ name:'shotCat',params:{paramId:'hello'},query:{queryId:'world'}}">helloWorld</router-link> <!--此时通过name匹配到路由对象shotCat.-->	 
<router-link :to="{ path:'/shotCat',params:{paramId:'hello'},query:{queryId:'world'}}">helloWorld</router-link>
<!--此时通过path匹配到路由对象shotCat.但是!!!!!此时`paramId`并不能添加到`$route.params`里,只有`queryId`成功添加到`$route.query`-->
```

🧡刷新页面后，`url`是不变，所以`query`参数可以正常显示在`url`地址栏中，刷新页面后也不会丢失。**而`params`参数不会显示在`url`地址栏中**，所以除了在路由中通过`routes`进行配置的，其他的页面刷新后都会丢失

### 路由组件传参

`params`和`query`进行传参的本质上都是把参数放在`url`上，会造成参数和组件的高度耦合。这时就可以使用route的props进行解耦，提高组件的复用，同时不改变`url`。

```javascript
const Hello = {
  props: ['name'], //使用route的props传参的时候,对应的组件一定要添加props进行接收,否则根本拿不到传参
  template: '<div>Hello {{ $route.params}}和{{this.name}}</div>'
}
const router = new VueRouter({
mode: 'history',
  routes: [
    { path: '/hello/:name', component: Hello, props: true }, //布尔模式: props 被设置为 true，此时route.params (即此处的name)将会被设置为组件属性。
    { path: '/static', component: Hello, props: { name: 'world' }}, // 对象模式: 此时就和params没什么关系了,此时的name将直接传给Hello组件.注意:此时的props需为静态!
    { path: '/dynamic/:years', component: Hello, props: dynamicPropsFn }, // 函数模式: 1.这个函数默认接受当前路由对象为参数2.这个函数返回的是一个对象3.在这个函数里你可以将静态值与路由相关值进行处理
  ]
})
function dynamicPropsFn (route) {
  return {
    name: (new Date().getFullYear() + parseInt(route.params.years)) + '!'
  }
}
new Vue({
  router,
  el: '#app'
})
```

### 路由懒加载

`vue`主要用于单页面应用，此时`webpack`会打包大量文件，这样就会造成首页需要加载资源过多，首屏时间过长,用户体验差。 如果使用路由懒加载，仅在你路由跳转的时候才加载相关页面，这样首页加载的东西少了，首屏时间也减少了。`vueRouter`的懒加载主要是靠**`Vue` 的异步组件**和**` Webpack` 的代码分割功能**，只需要将组件以`promise`形式引入即可.

```javascript
routes:[
      path:'/',
      name:'HelloWorld',
      component:resolve=>require(['@/component/HelloWorld'],resolve)
  ]
  //此时HelloWorld组件则不需要在第一步import进来
```

### 导航守卫

#### 全局守卫

一般在`main.js`和组件内部`this.$router`进行配置

- `router.beforeEach` 全局前置守卫

- `router.beforeResolve` 全局解析守卫(2.5.0+) 在`beforeRouteEnter`调用之后调用.

- `router.afterEach` 全局后置钩子，进入路由之后 **注意:不支持next()**

每个守卫方法接收三个参数：

- **to: Route**: 即将要进入的目标 [路由对象](https://router.vuejs.org/zh/api/#路由对象)
- **from: Route**: 当前导航正要离开的路由对象
- **next: Function**: 一定要调用该方法来 **resolve** 这个钩子。执行效果依赖 `next` 方法的调用参数。
  - **next()**: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 **confirmed** (确认的)。
  - **next(false)**: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。
  - **next('/') 或者 next({ path: '/' })**: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 `next` 传递任意位置对象，且允许设置诸如 `replace: true`、`name: 'home'` 之类的选项以及任何用在 [`router-link` 的 `to` prop](https://router.vuejs.org/zh/api/#to) 或 [`router.push`](https://router.vuejs.org/zh/api/#router-push) 中的选项。
  - **next(error)**: (2.4.0+) 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 [`router.onError()`](https://router.vuejs.org/zh/api/#router-onerror) 注册过的回调。

#### 路由独享守卫

- `beforeEnter`：路由只独享这一个钩子，一般在`Router`实例对象的`routes`中进行配置

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // 使用方法和上面的beforeEach一毛一样
      }
    }
  ]
})
```

#### **组件内的守卫**

- `beforeRouteEnter` 进入路由前,此时实例还没创建,无法获取到`this`
- `beforeRouteUpdate`路由复用同一个组件时
- `beforeRouteLeave` 离开当前路由,此时可以用来保存数据,或数据初始化,或关闭定时器等等

```javascript
//在组件内部进行配置,这里的函数用法也是和beforeEach一毛一样
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
```

