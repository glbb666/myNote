### 拖放
拖放是一种常见的特性，即抓取对象以后拖到另一个位置。

在 `HTML5` 中，拖放是标准的一部分，任何元素都能够拖放。
### 拖放分析

#### 设置被拖放元素和目标元素

##### 设置元素为可拖放
`<div draggable="true"></div>`

##### 设置元素为目标元素

默认无法将数据/元素放置到其他元素中。如果需要设置允许放置，我们必须阻止对元素的默认处理方式。

```javascript
var dragtarget = document.getElementById("dropstart");
dragtarget.ondragover = function(event){
	event.preventDefault()//一定要，否则ondrop事件不会被触发(因为拖不进容器内部了)
}
```

#### 拖放事件

- 被拖拽元素上触发的事件：
  - `dragstart` - 用户开始拖动元素时触发
  - `drag` - 元素正在拖动时持续触发
  - `dragend` - 拖动停止时（无论是把元素放置到有效的目标位置还是无效的目标位置，都会触发）
- 目标元素上触发的事件：
  - `dragenter` - 元素被拖动到放置目标上触发
  - `dragover` - 被拖动的对象在放置目标的范围内移动时持续触发
  - `dragleave`/`drop ` - 当被鼠标拖动的对象离开其容器范围内时触发此事件/如果元素被放到了放置目标中，则会触发`drop`事件而不是`dragleave`事件

> 如果是从其他应用软件或是文件中拖东西进来，尤其是图片的时候，默认的动作是显示这个图片或是相关信息，并不是真的执行drop。 
> 此时需要用用document的`ondragover`事件把它直接干掉。

#### `dataTransfer`对象

为了能在拖放操作时能实现数据交换而引入，有两个主要方法
##### `setData()`

一般在`dragstart`事件触发时调用

```js
function drag(ev)
{
    ev.dataTransfer.setData("Text",ev.target.id);//dataTransfer.setData() 方法设置被拖数据的数据类型和值
}
```
##### `getData()`
保存在`dataTransfer`对象中的数据只能在`ondrop`事件处理程序中读取

```js
function drop(ev)
{
    ev.preventDefault();
    var data=ev.dataTransfer.getData("Text");
    ev.target.appendChild(document.getElementById(data));
}
```
### 拖拽例子
```html
<!DOCTYPE HTML>
<html>
<head>
<style type="text/css">
#div1,#div2 {
    width:198px; 
    height:66px;
    padding:10px;
    border:1px solid #aaaaaa;
}
#div1 div,#div2 div{
  float: left;
}
</style>
<script type="text/javascript">


function drag(ev)
{
    console.log('start'+ev);
    ev.dataTransfer.setData("Text",ev.target.id);
}
function draging(ev){
    console.log('元素正在被拖动');
}
function draged(ev){
  ev.preventDefault();
  console.log('用户完成元素拖动'+ev);
}
function enter(ev){
  console.log('进入矩形'+ev.target.id);
  ev.preventDefault();
}
function over(ev)
{
  console.log('在矩形'+ev.target.id+'的范围内被拖动');
  ev.preventDefault();
}
function leave(ev){
  console.log('离开容器'+ev.target.id+'范围')
}
function drop(ev)
{
    console.log('在目标容器'+ev.target.id+'中安置');
    // ev.preventDefault();
    var data=ev.dataTransfer.getData("Text");
    //如果想把被拖动元素的放置到目标对象中，需要加这个
    ev.target.appendChild(document.getElementById(data));
}
</script>
</head>
<body>

<p>请把方块拖放到矩形中：</p>

<div id="div1" 
    ondragenter="enter(event)"
    ondragover="over(event)"
    ondragleave="leave(event)"
    ondrop="drop(event)"
></div>
<div id="div2"
    ondragenter="enter(event)"
    ondragover="over(event)"
    ondragleave="leave(event)"
    ondrop="drop(event)"
></div>

<div id="drag1" style="width:50px;height:50px;background-color:red" draggable="true" ondragstart="drag(event)" ondragend="draged(event)" ondrag="draging(event)"></div>
<div id="drag2" style="width:50px;height:50px;background-color:green" draggable="true" ondragstart="drag(event)" ondragend="draged(event)"
ondrag="draging(event)"></div>

</body>

</html>
```
### 原生拖拽
```js
var div = document.getElementsByTagName('div')[0];
var disX,
    disY;
//mousedown事件在鼠标按下时触发
div.onmousedown = function(e){
    disX = e.pageX - parseInt(div.style.left);
    disY = e.pageY - parseInt(div.style.top);
    document.onmousemove = function(e){  //要想快速移动不脱离鼠标则用document.  否则div
        div.style.left = e.pageX - disX + "px";
        div.style.top = e.pageY - disY +"px";
    }   
    document.onmouseup = function(){
        document.onmousemove = null;
   }
}
```