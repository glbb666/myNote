### 单行文本截断

单行文本的话，除非用空格做单词拆分，否则是不会换行的。

```css
.ellip{
	text-overflow:ellip;
	overflow:hidden;
	white-space:nowrap;
}
```

### 多行文本截断

#### `-webkit-line-clamp` 实现

```css
div {
  display: -webkit-box;/*将对象作为弹性伸缩盒子模型显示*/
  overflow: hidden;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;/*设置或检索伸缩盒对象的子元素的排列方式*/
}
```

1. 响应式截断，根据不同宽度做出调整
2. 文本超出范围才显示省略号，否则不显示省略号
3. 浏览器原生实现，所以省略号位置显示刚好

只有` webkit `内核的浏览器才支持这个属性，浏览器兼容性不好，多用于移动端页面，因为移动设备浏览器更多是基于 `webkit` 内核。

#### 定位元素实现多行文本截断

通过伪元素绝对定位到行尾并遮住文字，行数用高度和`line-height`实现再通过 overflow: hidden 隐藏多余文字。

```css
p {
    position: relative;
    line-height: 18px;
    height: 36px;
    overflow: hidden;
}
p::after {
    content:"...";
    font-weight:bold;
    position:absolute;
    bottom:0;
    right:0;
    padding:0 20px 1px 45px;
}
```

