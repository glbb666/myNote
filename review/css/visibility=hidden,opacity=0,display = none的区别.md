### display = none,visibility=hidden,opacity=0的区别
- `display = none`：不占据空间，引起重绘和回流，不会被子元素继承，`transition`无效，无法触发绑定事件
- `visibility = hidden`：占据空间，引起重绘，会继承，子元素可用`visibility:visible`显示，`transition`无效，无法触发绑定事件
- `opacity = 0`：占据空间，引起重绘，会继承，子元素设置`opacity=1`依然不能显示，可以触发绑定事件

