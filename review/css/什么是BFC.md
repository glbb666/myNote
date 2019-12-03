### 块级格式化上下文(`Block Formatting Context`)

`BFC`是`块级格式化上下文`，是一个**独立**的渲染区域，并且有一定的布局规则

### `BFC`创建方法

- **根元素**或**包含根元素的元素**
- **浮动**元素 (元素的 `float` 不为 `none`)
- **绝对定位**的元素`position` 为 `absolute` 或 `fixed`
- `overflow` 不为 `visible` 的元素
- **弹性元素**（`display`为 `flex` 或 `inline-flex`元素的直接子元素）
- 非块盒的块容器`display:inline-block / table-cells / table-caption.`
- 匿名表格单元格元素（元素的 `display`为 `table`、`table-row`、` table-row-group`、`table-header-group`、`table-footer-group`（分别是`table`、`row`、`tbody`、`thead`、`tfoot`的默认属性）或 `inline-table`）
- `display 的值为 flow-root的元素(`该元素会生成一个块级容器框，并且使用的是流布局。为里面内容创建新的块级格式化上下文。`)
- 网格元素（`display`为` grid` 或` inline-grid` 元素的直接子元素）
- 多列容器（元素的 `column-count` 或 `column-width` 不为 `auto`，包括 `column-count`为1）

#### display:flow-root的生效条件

- `float` 的值`不是none`。
-  `overflow`的使用值`不是 visible`
- `display`的值为`table-cell`，`table-caption`（见 [CSS3TBL]），`inline-- block`或`inline-table`。
- `position`的值既`不是static`也`不relative`（参见[CSS3POS]）。
-  `block-progression`值为`lr`或 `rl`，其父框的`block-progression`的值为`tb`
-  `block-progression`的 `tb`的值为' '，其父框的`block-progression`的值为`lr`或`rl`。

### `BFC`范围

`BFC`包含创建该上下文元素的所有子元素，但不包括创建了新`BFC`的子元素的内部元素。

```html
<div id='div_1' class='BFC'>
    <div id='div_2'>
        <div id='div_3'></div>
        <div id='div_4'></div>
    </div>
    <div id='div_5' class='BFC'>
        <div id='div_6'></div>
        <div id='div_7'></div>
    </div>
</div>
```

根据定义，`#div_1`创建了一个块格式上下文，这个上下文包括了`#div_2、#div_3、#div_4、#div_5`。由于`#div_5`创建了新的`BFC`，所以`#div_6`和`#div_7`就被排除在外层的`BFC`之外。

也就是说，**一个元素不能同时存在于两个`BFC`中。**

这也是为什么`BFC`可以解决浮动触发的问题，因为起到了隔离作用。

### 布局规则

- 垂直方向：`BFC`内部的盒子依次放置，相邻盒子的`margin`会折叠
- 水平方向：`每个元素的`margin`的左边，与`包含块`border`的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。
- `BFC`就是页面上的一个**隔离的独立容器**，容器里面的子元素不会影响到外面的元素。（解决margin塌陷）
- `BFC`的区域不会与浮动元素重叠。
- 计算`BFC`的高度时，**浮动元素也参与计算**(解决浮动元素父级高度塌陷)

### 应用

- 解决父元素高度塌陷
- 清除浮动

- 自适应两栏布局
- 分属于不同的`BFC`时可以解决`margin`塌陷

### `BFC`和`IFC`的区别

- `BFC`在浮动，绝对定位，`overflow`不为`visible`的情况下都会产生，`IFC`只在一个块级元素仅包含内联元素时产生。

- `BFC`在垂直方向依次放置，在同一个`BFC`内的垂直块级盒子会发生`margin`重叠，`IFC`在水平方向依次放置
- `BFC`的每个元素`margin`左边和包含块`border`左边相接触，`IFC`的对齐起点是包含块的顶部，垂直对齐方式由`vertical-align`决定
- `BFC`的高度会计算浮动元素，`IFC`的行框宽度需要减去浮动元素
- `BFC`是一个隔离的独立容器，容器里面的子元素不会影响到外面的元素