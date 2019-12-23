##### 类型一：父子嵌套（`margin`合并）
```css
.father{
    width:400px;
    height:400px;
    background: #00FFFF;
    margin-top: 20px;
  }  
.son{
    width:200px;
    height:200px;
    background: #FE2EF7;
    margin-top: 40px;
}
```
 我们对一对嵌套的盒子作如上样式设置，我们期望父级上边距为`20px`;子元素上边距为`40px`;实际上，父元素和子元素的上边距都变成了`40px`。

针对类型一：触发父级`BFC`，因为`BFC`容器内部的子元素是不会影响到外部元素的。

- `float`的属性不是`none`; 
- `overflow`的属性不是`visible`; 
- `position:absolute / fixed`; 
- `display：inline-block / table-cells / table-caption`

##### 类型二 :垂直相邻（`margin`合并）
```css
.box1{
    width:200px;
    height:200px;
    background: #00FFFF;
    margin-bottom: 40px;
  }  
.box2{
    width:200px;
    height:200px;
    background: #FE2EF7;
    margin-top: 20px;
}
```
在`css`中对垂直相邻块级元素作如上样式设置，我们期望的间距是`60px`,实际上，它们之间的距离只有`40px`。

针对类型二：

*   我们可以在设置边距时只设置一个边距来解决。
*   只有属于同一个`BFC`的相邻垂直盒才会发生`margin`折叠，所以我们可以把一个盒子用`div`包起来，触发这个`div`的`BFC`，这样这两个盒子就不属于一个`BFC`了

##### 类型三 :空块级元素自身的`margin-top`和`margin-bottom`合并

针对类型三：

- 设置`float`为非`none`，浮动元素不会发生外边距合并。

- 设置`border`或`padding`或文字内容阻隔`margin`

#### 💛`margin`重叠时`margin`值计算方法：

- 正值取大
- 负值取小
- 正负取和

