### 捕获和冒泡

捕获和冒泡的概念都是为了解决事件发生顺序的问题。

- 捕获：从最外层开始发生（`Document`），知道最具体的元素。

- 冒泡：从最底层元素开始发生，然后逐级向上传播到`Document`节点。


事件冒泡和事件捕获的过程图（先捕获再冒泡）

![img](../images/16a2654b0dd928ef)

### addEventListener的第三个参数
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

### 事件代理

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

