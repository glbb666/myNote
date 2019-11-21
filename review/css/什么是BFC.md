### 视觉格式模型（FC）

`FC`是用来**处理文档并将它显示在视觉媒体上的机制**，它定义了盒的生成，盒主要包括了块盒、行内盒、匿名盒（没有名字不能被选择器选中的盒）以及一些实验性的盒（未来可能添加到规范中）。盒的类型由display属性决定。

### 块盒（block box）

特性：

- `display`为`block`，`list-item`或 `table`时，它是块级元素；
- 视觉上呈现为块，竖直排列；
- 块级盒参与(块格式化上下文)；
- 每个块级元素至少生成一个块级盒，称为主要块级盒(`principal block-level box`)。一些元素，比如
- 生成额外的盒来放置项目符号，不过多数元素只生成一个主要块级盒。
- 块状元素排斥其他元素与其位于同一行，可以设定元素的宽（`width`）和高（`height`），块级元素一般是其他元素的容器，可容纳块级元素和行内元素

### 块级格式化上下文(`Block Formatting Context`)

`BFC`是`块级格式化上下文`，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域，包含创建它的元素内部的所有内容

### `BFC`创建方法

- **根元素**或**包含根元素的元素**
- **浮动**元素 (元素的 `float` 不为 `none`)
- `position` 不为 `relative` 或 `static`的元素
- `overflow` 的值不为 `visible` 的元素
- **弹性元素**（`display`为 `flex` 或 `inline-flex`元素的直接子元素）
- `display:inline-block / table-cells / table-caption.`
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
- 计算`BFC`的高度时，**浮动元素也参与计算**(解决父级高度塌陷)

### 应用

- 解决父元素高度塌陷
- 清除浮动

- 自适应两栏布局
- 分属于不同的`BFC`时可以解决`margin`塌陷


### `BFC`和`IFC`的区别

