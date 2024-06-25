# `CSS`基础盒模型详解

`css`盒模型由 `margin`，`border`，`padding`，`content`四个部分组成

## `box-sizing`

盒模型有两种

- `content-box`：W3C盒模型，宽高为 `content`。padding和border的宽高会增加到最后计算出的元素的总宽度和总高度上。
- `border-box`： `IE`盒模型，宽高指的是 `border+padding+content`

另外有两个较少用到，非标准的值：

- `padding-box`：宽高指的是 `padding+content`
- `inherit` ：从父元素继承 `box-sizing`属性

我们在编写页面代码时应使用标准的 `W3C`模型(需在页面中声明 `DOCTYPE`类型)，这样可以避免多浏览器对同一页面的不兼容。

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
