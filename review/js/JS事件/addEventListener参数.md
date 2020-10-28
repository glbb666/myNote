### 捕获和冒泡

捕获和冒泡的概念都是为了解决事件发生顺序的问题。

- 捕获：从最外层开始发生（`Document`），知道最具体的元素。

- 冒泡：从最底层元素开始发生，然后逐级向上传播到`Document`节点。


事件冒泡和事件捕获的过程图（先捕获再冒泡）

![img](../images/16a2654b0dd928ef)

### `addEventListener`的第三个参数
`addEventListener(event,function,useCapture)`

`addEventListener(event,function,options)`

- `event `指定事件名，不要使用 "on" 前缀。 例如，使用 "click" ,而不是使用 "onclick"。

- `function`指定事件发生后触发的函数

- `useCapture `：指定事件在捕获接断还是冒泡阶段执行

  - `true `在捕获阶段执行

  - `false` 在冒泡阶段执行

    默认值`false`

- `options` 指定有关 `listener `属性的可选参数**对象** 

  可用的选项如下：

  - `capture`:  [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)，表示 `listener` 会在该类型的事件捕获阶段传播到该 `EventTarget` 时触发。
  - `once`:  [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)，表示 `listener 在添加之后最多只调用一次。如果是` `true，` `listener` 会在其被调用之后自动移除。
  - `passive`: [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)，设置为true时，表示 `listener` 永远不会调用 `preventDefault()。`

### 事件委托

事件委托，就是把一个元素响应事件的函数，委托到另外一个元素，一般是它的父元素。

这样，当事件响应到需要绑定的元素上时，会通过事件冒泡机制而触发到它的外层元素的绑定事件上，然后在外层函数上去执行函数。

```html
<ul class="color_list">        
    <li>red</li>        
    <li>orange</li>        
    <li>yellow</li>        
    <li>green</li>        
    <li>blue</li>        
    <li>purple</li>    
</ul>
<div class="box"></div>
```

```css
.color_list{            
    display: flex;            
    display: -webkit-flex;        
}        
.color_list li{            
    width: 100px;            
    height: 100px;            
    list-style: none;            
    text-align: center;            
    line-height: 100px;        
}
/* 每个li加上对应的颜色，此处省略 */
.box{            
    width: 600px;            
    height: 150px;            
    background-color: #cccccc;            
    line-height: 150px;            
    text-align: center;        
}
```

![img](../images/16a264146329fa93)

```javascript
var color_list=document.querySelector(".color_list");
var box=document.querySelector(".box");            
function colorChange(e){
    var e=e||window.event;//兼容性的处理
    //无论是冒泡还是捕获，e.target都是正在点击的最深层的元素
    //e.target.nodeName是正在点击的元素名称的大写，如'LI'
    if(e.target.nodeName.toLowerCase()==="li"){                    
        box.innerHTML="该颜色为 "+e.target.innerHTML;                
    }                            
}            
color_list.addEventListener("click",colorChange,true)
```

#### 优点

**减少内存消耗**

如果我们有一个列表，列表之中有大量的列表项，需要在点击列表项的时候响应事件。给每个列表项都绑定一个函数，那对于内存消耗是非常大的，效率上需要消耗很多性能。比较好的方法就是把这个点击事件绑定到他的父层，也就是 `ul` 上，然后在执行事件的时候再去匹配判断目标元素。

**为什么性能消耗大呢**

每次鼠标移动/离开，都会触发`mousemove/mouseout`事件，这意味着可能有多个事件在短时间内触发，且此时触发的是`DOM`事件。在浏览器中，`dom`事件的实现和`js`的实现是分离的，因此，操作`dom`，就是通过`js`代码调用`dom`的接口，这是两个独立的模块的交互，会造成较大的性能损耗。访问`dom`的次数越多，引起浏览器的重绘和重排的次数就越多，就会延长整个页面的交互就绪事件。

如果处理器做任何重大的处理，或者如果该事件存在多个处理函数，这可能造成浏览器的严重的性能问题。使用事件委托就可以避免这些问题。