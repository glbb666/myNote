## 方法一、`props`/`$emit`

### 1.父组件向子组件传值

父组件把数据作为一个自定义特性传递给子组件

`v-bind`可以动态传递`prop`

```javascript
//App.vue父组件
<template>
    <div id="app">
        <users v-bind:users="users"></users>//前者自定义名称便于子组件调用，后者要传递数据名
        <users2 title="myTitle"></users2>//前者自定义名称便于子组件调用，后者要传递数据值
	</div>
</template>
<script>
export default {
  ...
  data(){
    return{
      users:["Henry","Bucky","Emily"]
    }
  },
}
</script>
```
```javascript
//users子组件
<template>
  <div class="hello" v-for="user in users">{{title+user}}</div>
</template>
<script>
export default {
  ...
  props:{
    users:{           //这个就是父组件中子标签自定义名字
      type:Array,
      required:true
    }，
    title:{
      type:String,
      required:true
    }
  }
}
</script>
```

**总结：父组件通过props向下传递数据给子组件。**

### 2.子组件向父组件传值（通过事件形式）

子组件通过`$emit`触发事件`A`，并传递数据，父组件通过`v-on`监听事件A，并且指定事件A发生之后触发的函数B，我们可以从函数B的 `$event`中获取到`$emit`抛出的值（子组件传递给父组件的数据）。

```javascript
// 子组件
<template>
  <header>
    <h1 @click="this.$emit("titleChanged","子向父组件传值")">{{title}}</h1>
  </header>
</template>
```
```javascript
// 父组件
<template>
  <div id="app">
    <app-header v-on:titleChanged="title = $event" ></app-header>//与子组件titleChanged自定义事件保持一致
    <h2>{{title}}</h2>
  </div>
</template>
<script>
export default {
  ...
  data(){
    return{
      title:"传递的是一个值"
    }
  }
}
</script>
```
## 方法二、`$emit`/`$on`

创建一个空的`vue`实例作为中介，用$emit来触发事件和用$on监听事件，实现了任何组件间的通信，当我们的项目比较大时可以用`vuex`

#### 1.具体实现方式：

```javascript
var Event=new Vue();
Event.$emit(事件名,数据);
Event.$on(事件名,data => {});
```

#### 2.举个例子

假设兄弟组件有三个，分别是A、B、C组件，C组件如何获取A或者B组件的数据

```javascript
<div id="itany">
	<my-a></my-a>
	<my-b></my-b>
	<my-c></my-c>
</div>
<template id="a">
  <div>
    <h3>A组件：{{name}}</h3>
    <button @click="send">将数据发送给C组件</button>
  </div>
</template>
<template id="b">
  <div>
    <h3>B组件：{{age}}</h3>
    <button @click="send">将数组发送给C组件</button>
  </div>
</template>
<template id="c">
  <div>
    <h3>C组件：{{name}}，{{age}}</h3>
  </div>
</template>
<script>
var Event = new Vue();//定义一个空的Vue实例
var A = {
	template: '#a',
	data() {
	  return {
	    name: 'tom'
	  }
	},
	methods: {
	  send() {
	    Event.$emit('data-a', this.name);
	  }
	}
}
var B = {
	template: '#b',
	data() {
	  return {
	    age: 20
	  }
	},
	methods: {
	  send() {
	    Event.$emit('data-b', this.age);
	  }
	}
}
var C = {
	template: '#c',
	data() {
	  return {
	    name: '',
	    age: ""
	  }
	},
	mounted() {//在模板编译完成后执行
	 Event.$on('data-a',name => {
	     this.name = name;//箭头函数内部不会产生新的this，这边如果不用=>,this指代Event
	 })
	 Event.$on('data-b',age => {
	     this.age = age;
	 })
	}
}
var vm = new Vue({
	el: '#itany',
	components: {
	  'my-a': A,
	  'my-b': B,
	  'my-c': C
	}
});	
</script>

```

## 方法三、`vuex`

### 1.简要介绍`Vuex`原理

`Vuex`实现了一个单向数据流，在全局拥有一个`State`存放数据，当组件必须通过`Mutation`更改`State`中的数据，`Mutation`同时提供了订阅者模式供外部插件调用获取`State`数据的更新。而当所有异步操作(常见于调用后端接口异步获取更新数据)或批量的同步操作需要走`Action`，但`Action`也是无法直接修改`State`的，还是需要通过`Mutation`来修改`State`的数据。最后，根据`State`的变化，渲染到视图上。

### 2.简要介绍各模块在流程中的功能：

