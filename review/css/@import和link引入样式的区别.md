# link

## 定义和用法

<link> 标签定义文档与外部资源的关系。

<link> 标签最常见的用途是链接样式表。还能被用来创建站点图标(比如PC端的“favicon”图标和移动设备上用以显示在主屏幕的图标)甚至一些其他事情。

# @import

**@import** [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)[@规则](https://developer.mozilla.org/en-US/docs/Web/CSS/At-rule)，用于从其他样式表导入样式规则。这些规则必须先于所有其他类型的规则，[`@charset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@charset) 规则除外; 因为它不是一个[嵌套语句](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Syntax#nested_statements)，@import不能在[条件组的规则](https://developer.mozilla.org/zh-CN/docs/Web/CSS/At-rule#Conditional_Group_Rules)中使用。

### 1.从属关系区别

`@import`是 CSS 提供的语法规则，只有导入样式表的作用；`link`是HTML提供的标签，不仅可以加载 CSS 文件，还可以定义 RSS、rel 连接属性等。

### 2.加载顺序区别

加载页面时，`link`标签引入的 CSS 被同时加载；`@import`引入的 CSS 将在页面加载完毕后被加载。

### 3.兼容性区别

`@import`是 CSS2.1 才有的语法，故只可在 IE5+ 才能识别；`link`标签作为 HTML 元素，不存在兼容性问题。

### 4.DOM可控性区别

可以通过 JS 操作 DOM ，插入`link`标签来改变样式；由于DOM方法是基于文档的，无法使用`@import`的方式插入样式。

作者：Caoyp_Let_It

链接：https://juejin.im/post/5ab36d99f265da23866fccd1

来源：掘金

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。