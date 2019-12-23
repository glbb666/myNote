# `CSS`基础盒模型详解

可以认为每个`html`标签都是一个方块，然后这个方块又包着几个小方块，如同盒子一层一层的包裹着，这就是盒模型。

`css`盒模型由`margin`，`border`，`padding`，`content`四个部分组成

- `IE5-`采用`IE`盒模型
- `IE6、7`已经开始使用`W3C`盒模型
- `IE8+`借助`box-sizing`，又重新提供了对`IE`盒模型的支持

## `box-sizing`
- `content-box`：标准盒模型，宽高为`content`

- `border-box`：怪异盒模型，又称`IE`盒模型，宽高指的是`border+padding+content`

- `padding-box`：宽高指的是`padding+content`

- `inherit` ：从父元素继承`box-sizing`属性

我们在编写页面代码时应使用标准的`W3C`模型(需在页面中声明`DOCTYPE`类型)，这样可以避免多浏览器对同一页面的不兼容。

```css
由于content-box在计算宽度的时候不包含border pading很烦人，而且又是默认值，业内一般采用以下代码重置样式：
html {
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
}
*, *:before, *:after {
    -webkit-box-sizing: inherit;
    -moz-box-sizing: inherit;
    box-sizing: inherit;
}
```





