## 属性值都有哪些(5种)
- static（静态定位）

对象遵循标准文档流中，top, right, bottom, left 等属性失效。
- relative(相对定位)

对象遵循标准文档流中，依赖top, right, bottom, left 等属性相对于该对象在标准文档流中的位置进行偏移，同时可通过z-index定义层叠关系。position:relative 对 table-*-group, table-row, table-column, table-cell, table-caption 元素无效。
- absolute(绝对定位)

对象脱离标准文档流，相对于static定位以外的第一个父元素进行绝对定位。绝对定位的元素可以设置外边距（margins），且不会与其他边距合并。可通过z-index定义层叠关系。**当元素祖先的 transform  属性非 none 时，容器由视口改为该祖先。**
- fixed(固定定位)

对象脱离标准文档流，通过指定元素相对于屏幕视口（viewport）的位置来指定元素位置。元素位置不随屏幕滚动改变。fixed 属性会创建新的层叠上下文。**当元素祖先的 transform  属性非 none 时，容器由视口改为该祖先。**
- sticky(粘性定位)
可以被认为是相对定位和固定定位的混合。元素在跨越特定阈值前为相对定位，之后为固定定位。position: sticky 对 table 元素的效果与 position: relative 相同。


填充规则：

- `height`和`width`被设定为`auto`的绝对定位元素，按其内容大小调整尺寸。但，被绝对定位的元素可以通过指定`top`和`bottom` ，保留`height`未指定（即`auto`），来填充可用的垂直空间。它们同样可以通过指定`left `和 `right`并将`width `指定为`auto`来填充可用的水平空间。
- `top`,`bottom`同时设定时`top`优先。`left`,` right`同时设置，当 `direction`设置为 `ltr`时 `left` 优先， 当`direction`设置为` rtl`时` right `优先。

### relative
```html
<div class="first"></div>
<div class="second"></div>
```
```css
实例1：relative是相对该对象处于标准文档流中的位置依据left,top（right,bottom）进行定位，left,top 并不改变该对象原本在文档流中的占位空间。
div {
    width: 100px;
    height: 100px;
}

.first {
    border: 1px solid black;
    position: relative;
    top: 10px;
    left: 10px;
}
.second {
    border: 1px solid red;
}
```
实例2：当设置了margin属性时，该对象在标准文档流中的占位空间会改变。padding也会改变占位空间。
```html
<div class="first"></div>
<div class="second"></div>
```
```css
div {
    width: 100px;
    height: 100px;
}

.first {
    border: 1px solid black;
    position: relative;
    top: 10px;
    left: 10px;
    margin: 0 0 10px 10px;
    padding: 10px;
}

.second {
    border: 1px solid red;
}
```
### absolute
实例：可看出absolute绝对定位根据最近一个非static定位的父级进行定位。**值得一提的是在使用absolute定位时，必须指定left,top,right,bottom中的至少一个（否则left/right/top/bottom属性会使用它们的默认值 auto ，这将导致对象遵从标准文档流，在前一个对象之后立即被呈递，简单讲就是都变成relative，会占用文档空间）。**
```html
<div class="grabdp">
    <div class="parent">
        <div class="first"></div>
        <div class="second"></div>
    </div>
</div>
```
```css
.grabdp {
    width: 300px;
    height: 300px;
    position: absolute;
    border: 1px solid rgb(12, 153, 42);
}

.parent {
    width: 200px;
    height: 200px;
    position: static;
    border: 1px solid blue;
    margin-left: 20px;
}

.first {
    width: 100px;
    height: 100px;
    border: 1px solid black;
    position: absolute;
    top: 10px;
    left: 10px;
    padding: 10px;
}

.second {
    width: 100px;
    height: 100px;
    border: 1px solid red;
}
```
### sticky
实例
```html
<div>
    <dl>
        <dt>A</dt>
        <dd>Andrew W.K.</dd>
        <dd>Apparat</dd>
        <dd>Arcade Fire</dd>
        <dd>At The Drive-In</dd>
        <dd>Aziz Ansari</dd>
    </dl>
    <dl>
        <dt>C</dt>
        <dd>Chromeo</dd>
        <dd>Common</dd>
        <dd>Converge</dd>
        <dd>Crystal Castles</dd>
        <dd>Cursive</dd>
    </dl>
    <dl>
        <dt>E</dt>
        <dd>Explosions In The Sky</dd>
    </dl>
    <dl>
        <dt>T</dt>
        <dd>Ted Leo & The Pharmacists</dd>
        <dd>T-Pain</dd>
        <dd>Thrice</dd>
        <dd>TV On The Radio</dd>
        <dd>Two Gallants</dd>
    </dl>
</div>
```
```css
* {
    box-sizing: border-box;
}

dl {
    margin: 0;
    padding: 24px 0 0 0;
}

dt {
    background: #B8C1C8;
    border-bottom: 1px solid #989EA4;
    border-top: 1px solid #717D85;
    color: #FFF;
    font: bold 18px/21px Helvetica, Arial, sans-serif;
    margin: 0;
    padding: 2px 0 0 12px;
    position: sticky;
    top: -1px;
}

dd {
    font: bold 20px/45px Helvetica, Arial, sans-serif;
    margin: 0;
    padding: 0 0 0 12px;
    white-space: nowrap;
}

dd+dd {
    border-top: 1px solid #CCC
}
```
### sticky生效以及失效
- 须**指定 top, right, bottom 或 left 四个阈值其中之一，才可使粘性定位生效**。否则其行为与**相对定位**相同。
- 元素自身在文档流中的位置
上面已经讲过了，如果设置了**top: 50px**，那么元素在达到距离顶部50px时才会发生定位，否则并不会发生定位。
- 该元素的父容器的边缘
sticky元素在到达父容器的底部时，则不会再发生定位，如果父容器高度并没有比sticky元素高，那么sticky元素一开始就达到了底部，并不会有定位的效果。
- 父元素的**overflow**属性，如果父元素的**overflow**属性并不是默认的**visible**属性，那么**sticky**元素则相对于该父元素定位。也就是如果要定位在顶部的话，此时这个效果就无效了。
### z-index

同级元素的此属性具有同样的值，依据它们在HTML文档中流的顺序层叠，写在后面的将会覆盖前面的。父子关系时子级在父级上面。

