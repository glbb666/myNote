1. border

```css
.demo {
  border: 2px solid #000000;
}
```

2. box-shadow

- box-shadow不占据布局空间，它的宽度不会影响到元素的总体尺寸。

```css
.demo {
  box-shadow: 0 0 0 2px #000000;
}
```

3. outline

- outline 不占据布局空间，它的宽度不会影响到元素的总体尺寸。
- outline 是在 border 的外侧绘制。
- outline 是矩形的，不能指定单独的边，也不跟随元素的圆角。
- outline 可以是非矩形线型（如虚线等），且可以通过 outline-offset 属性设置偏移量。

```css
.demo {
  outline: outline-color outline-style outline-width;
}
```

4. background

- `background-image` 属性可以接受一个或多个以逗号分隔的图像值。这种设计允许你在同一个元素上应用多层背景图像。
- `background-size`也可以接受一个或多个值，来设置多个背景图层的宽高
- `background-position` 属性在 CSS 中用来指定一个或多个背景图像的起始位置
- `padding`：确保文本或元素内容不会与“边框”接触，从而在“边框”周围形成足够的空间

```css
.demo {
   background-image: 
        linear-gradient(red, black), /* top border */
        linear-gradient(red, black), /* right border */
        linear-gradient(red, black), /* bottom border */
        linear-gradient(red, black); /* left border */
    background-repeat: no-repeat;
    background-size: 
        100% 1px,    /* top border */
        1px 100%,    /* right border */
        100% 1px,    /* bottom border */
        1px 100%;    /* left border */
    background-position: 
        0 0,          /* top border */
        100% 0,       /* right border */
        0 100%,       /* bottom border */
        0 0;          /* left border */
    padding: 1px; 
}
```
