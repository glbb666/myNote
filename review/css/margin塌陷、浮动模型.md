## 一、margin塌陷（外边距折叠）问题的分类
#### 类型一：父子嵌套
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

**结论：当一个块级元素包含在另一个块级元素之中时，子元素与父元素之间也会产生重叠现象，重叠后的外边距取其中最大者。**

`margin`重叠时`margin`值计算方法：

- 正值取大
- 负值取小
- 正负取和

针对类型一：触发父级`BFC`（`Block Formatting Context` ），改变父级的渲染规则

* `float`的属性不是`none`; 
* `overflow`的属性不是`visible`; 
* `position:absolute / fixed`; 
* `display：inline-block / table-cells / table-caption`

**解决方案各有优缺点，`例如：overflow:hidden; 溢出部分隐藏 但溢出部分有用时，不能用这个方法`   在解决这个问题时要根据实际情况进行选取。哪个不影响用哪个***

#### 类型二  
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
  &nbsp; &nbsp; &nbsp;   在css中对垂直相邻块级元素作如上样式设置，我们期望的间距是60px,实际上，它们之间的距离只有40px。

**结论：两个垂直相邻的块级元素，当上下两个边距相遇时，起外边距会产生重叠现象，且重叠后的外边距，等于其中较大者。**

针对类型二：

*   我们可以在设置边距时只设置一个边距来解决。

