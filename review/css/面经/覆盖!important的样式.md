##### 已知如下代码，如何修改才能让图片宽度为 300px ？注意下面代码不可修改。

```html
<img src="1.jpg" style="width:480px!important;”>
```

方法一：`max-width: 300px`

方法二：`transform: scale(0.625,0.625)`

方法三：

```
box-sizing: border-box;
padding: 0 90px;
```

方法四：给图片设置动画，`from{width:300px;}to{width:300px;}`，动画时间为`0s`，原理是`CSS`动画的样式优先级高于`!important`的特性

