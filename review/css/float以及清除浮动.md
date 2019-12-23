#### 一、浮动特性

`float`属性的实现初衷是为了实现文字环绕图片的效果。它有三个值`none`，`left`，`right`，`inline-start`，`inline-end`，后两个值表示元素必须浮动在所在块的开始/结束一侧。

它有以下特性：

- 块布局，会修改`display`的计算值，大多数是修改为`block`，`flex`的时候是`flex`，`inline-flex`的时候是`inline-flex`。

- 包裹性：即此时元素`width`会像`height`一样由子元素决定，而不是默认撑满父元素。

- `BFC`：元素`float`值不为`none`之后，会在元素内部创建一个`BFC`。

- 没有任何`margin`合并；

- 脱离文档流：`float`设计的初衷就是为了“文字环绕”效果，为了让文字环绕图片，就需要具备两个条件。第一是元素高度坍塌，第二是行框盒子不可与浮动元素重叠。而元素高度坍塌就导致元素后面的非浮动块状元素会和其重叠，于是他就像脱离文档流了。

#### 二、浮动引起父元素塌陷

浮动：当盒子设置`float：left/right`就会脱离文档流，变成浮动元素。允许文本和内联元素环绕它。

```css
.father{
    border: 5px solid black;
    width: 300px;
}  
.son1{
    border: 5px solid #f66;
    width: 100px;
    height: 100px;
    float: left;
}
.son2{
    border: 5px solid #f66;
    width:100px;
    height: 100px;
    float: left;
}
```

效果图：![这里写图片描述](images/20180624160209473.png)

#### 三、清除浮动

##### clear属性清除浮动

- 块级元素清除浮动

最后一个子元素添加

```css
p{
	clear:both;
}
```

- 伪元素清除浮动：

```css
.father::after {
    content: '';
    display: block;//clear 很特殊，想让他生效，必须是块级元素才可以，而::after 是行级元素
    clear: both;
    width: 0;
    height: 0;
    visibility:hidden;//允许浏览器渲染它，但是不显示出来，这样才能实现清楚浮动。
}
.father{
     zoom:1;  /*==for IE6/7 Maxthon2==*/
}
```

##### `BFC`清除浮动

- 触发父级`BFC `：`BFC`在计算的时候会把浮动元素的高度计算在内。