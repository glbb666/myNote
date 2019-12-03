## 属性值都有哪些(5种)
- static（静态定位）

对象遵循标准文档流中，没有定位，top, right, bottom, left,z-index等属性失效。

- inherit

  规定应该从父元素继承 position 属性的值。

- relative(相对定位)

对象遵循标准文档流中，依赖top, right, bottom, left 等属性**相对于该对象在标准文档流中的位置进行偏移**，其占据的空间仍然保留。同时可通过z-index定义层叠关系。相对定位元素会创建一个包含块，用于作为内部子元素定位的基点。

💛 `static` 或 `relative` ，包含块就是由它的最近的祖先**块元素**或**格式化上下文**的内容边界

- absolute(绝对定位)

对象脱离标准文档流，相对于**包含块**进行定位。此时其原有空间为0，附近元素也会重新排列。同时，该元素会产生格式化上下文（BFC）。上下外边距不会合并，不会因为内部浮动元素产生高度塌陷。

💛`absolute`的包含块就是由它的最近的 position 的值不是 `static`的祖先元素的内边距区的边缘。如果绝对定位的祖先元素position属性都是static，那么会相对于初始包含块————body定位。如果存在position属性为其他值的祖先元素，则相对于其创建的包含块定位。

**在使用absolute定位时，必须指定left,top,right,bottom中的至少一个（否则left/right/top/bottom属性会使用默认值 auto ，导致对象遵从标准文档流，简单讲就是都变成relative，会占用文档空间）。**

- fixed(固定定位)

对象脱离标准文档流，相对于屏幕视口定位，元素位置不随屏幕滚动改变。同时，该元素也会产生格式化上下文（BFC）。旧版本IE不支持。

💛如果`position`属性是`fixed`和`absolute`，**当元素祖先的 transform  属性非 none 时，容器会相会于该祖先定位**，这是因为设置transform属性为非none的会导致该祖先元素形成一个新的包含块，元素的包含块由视口变成了该祖先。

- sticky(粘性定位)
可以被认为是相对定位和固定定位的混合。元素在进入特定阈值前为相对定位，之后为固定定位。举个例子：当top=20px时，在元素往上移动的过程中，一旦进入top=20px的范围，就会固定定位。position: sticky 对 table 元素的效果与 position: relative 相同。


填充规则：

- `height`和`width`被设定为`auto`的绝对定位元素，按其内容大小调整尺寸。但，被绝对定位的元素可以通过指定`top`和`bottom` ，保留`height`未指定（即`auto`），来填充可用的垂直空间。它们同样可以通过指定`left `和 `right`并将`width `指定为`auto`来填充可用的水平空间。
- `top`,`bottom`同时设定时`top`优先。`left`,` right`同时设置，当 `direction`设置为 `ltr`时 `left` 优先， 当`direction`设置为` rtl`时` right `优先。

### sticky生效以及失效
- 须**指定 top, right, bottom 或 left 四个阈值其中之一，才可使粘性定位生效**。否则其行为与**相对定位**相同。
- 元素自身在文档流中的位置
上面已经讲过了，如果设置了**top: 50px**，那么元素在达到距离顶部50px时才会发生定位，否则并不会发生定位。
- 该元素的父容器的边缘
sticky元素在到达父容器的底部时，则不会再发生定位，如果父容器高度并没有比sticky元素高，那么sticky元素一开始就达到了底部，并不会有定位的效果。
- 父元素的**overflow**属性，如果父元素的**overflow**属性并不是默认的**visible**属性，那么**sticky**元素则相对于该父元素定位。也就是如果要定位在顶部的话，此时这个效果就无效了。

![image](images/bbcdaa41c5d4338434c3dfc2e5b5c47f_articlex.png)

