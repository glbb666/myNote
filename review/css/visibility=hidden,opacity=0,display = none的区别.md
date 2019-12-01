### visibility=hidden,opacity=0,display = none的区别

- `opacity = 0`：该元素隐藏起来了，但不会改变页面布局，并且，如果该元素已经绑定一些事件，如`click`事件，那么点击该区域，还是能触发该事件
- `visibility = hidden`：该元素隐藏起来了，但不会改变页面布局，也不会触发该元素已经绑定的事件
- `display = none`：把元素隐藏起来，并且会改变页面布局，可以理解成在页面中把该元素删除掉一样

