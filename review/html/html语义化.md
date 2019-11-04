# 对HTML语义化的理解（语义化标签有哪些）
### 什么是html语义化

> 语义化是指根据**内容的结构化**（内容语义化），选择**合适的标签**（代码语义化），**便于开发者阅读**和写出更**优雅的代码**的同时，让浏览器的**爬虫**和**机器**很好的**解析**。

### 语义化的优点

- 有利于SEO，有助于爬虫抓取更多的有效信息。（爬虫根据标签来确定上下文和各个关键字的权重）
- 有利于构建清晰的代码结构，利于团队开发维护。
- 有利于构建清晰的内容结构，html 在没有 css 的情况下，能呈现 **较好的** 内容结构
- 方便其他设备的解析，比如盲人阅读器，有利于区分哪些优先阅读

### 语义化标签都有哪些

- `<h1>~<h6>`，做标题，依据重要性递减，`<h1>` 最高
- `<p>`段落标记
- `<ul>、<ol>、<li>`，`<ul>` 无序列表，这个被大家广泛的使用，`<ol>` 有序列表不常用。
- `<dl>、<dt>、<dd>`，`<dl>` 就是“定义列表”。`<dl>`常与`<dt>`和`<dd>`一起使用。`<dl>`开启一个定义列表，`<dt>`表示要定义的项目名称，`<dd>`表示对`<dt>`的项目的描述。
- `<em>、<strong>`，`<em>` 是用作强调，`<strong>` 是用作重点强调。
- `<table>、<thead>、<tbody>、<td>、<th>、<caption>`， 用来做表格

### h5 新增语义化标签

![HTML5 语义化](<https://github.com/glbb666/myNote/blob/master/review/html/images/html5-layout.jpg>)

### header

> 定义 section 或 page 的页眉

```html
<header>
  <h1>Welcome to my homepage</h1>
  <p>My name is Donald Duck</p>
</header>
<main>
  ...
</main>
<footer>
  <p>end</p>
</footer>
```

### nav

定义导航链接的部分。

> 如果文档中有“前后”按钮，则应该把它放到 <nav> 元素中。

```html
<nav>
  <a href="index.asp">Home</a>
  <a href="html5_meter.asp">Previous</a>
  <a href="html5_noscript.asp">Next</a>
</nav>
```

### main

规定文档的主要内容。

<main> 元素中的内容对于文档来说应当是唯一的。它不应包含在文档中重复出现的内容，比如侧栏、导航栏、版权信息、站点标志或搜索表单。 ️

> ⚠️️ 注意：在一个文档中，不能出现一个以上的 <main> 元素。<main> 元素不能是以下元素的后代：<article>、<aside>、<footer>、<header> 或 <nav>。

```html
<main>
  <h1>Web Browsers</h1>
  <p>Google Chrome、Firefox 以及 Internet Explorer 是目前最流行的浏览器。</p>

  <article>
    <h1>Google Chrome</h1>
    <p>Google Chrome 是由 Google 开发的一款免费的开源 web 浏览器，于 2008 年发布。</p>
  </article>

  <article>
    <h1>Internet Explorer</h1>
    <p>Internet Explorer 由微软开发的一款免费的 web 浏览器，发布于 1995 年。</p>
  </article>

  <article>
    <h1>Mozilla Firefox</h1>
    <p>Firefox 是一款来自 Mozilla 的免费开源 web 浏览器，发布于 2004 年。</p>
  </article>
</main>
```

### section

定义文档中的节（section、区段)

```html
<section>
  <h1>PRC</h1>
  <p>The People's Republic of China was born in 1949...</p>
</section>
```

### article

规定独立的自包含内容（常用于文章）。

```html
<article>
  <h1>Internet Explorer 9</h1>
  <p>Windows Internet Explorer 9（简称 IE9）于 2011 年 3 月 14 日发布.....</p>
</article>
```

### aside

标签定义其所处内容之外的内容

```html
<p>Me and my family visited The Epcot center this summer.</p>
<aside>
<h4>Epcot Center</h4>
The Epcot Center is a theme park in Disney World, Florida.
</aside>
```

> <aside> 的内容可用作文章的侧栏。

### footer

定义文档或节的页脚。

> <footer> 元素应当含有其包含元素的信息。页脚通常包含文档的作者、版权信息、使用条款链接、联系信息等等。可以在一个文档中使用多个 <footer> 元素。

```html
<footer>
  <p>Posted by: W3School</p>
  <p>Contact information:
    <a href="mailto:someone@example.com">someone@example.com</a>
  </p>
</footer>
```

# div布局是否符合html语义化（为什么要用div布局）

[div+ css布局和table布局的比较](https://blog.csdn.net/qq_40128682/article/details/89883319)
不符合。div布局和table布局比较是有优势的

- **速度快**，增强用户体验。DIV+CSS布局较Table布局（`为什么`）**减少了页面代码**，加载速度得到很大的提高。
-  **符合w3c标准**，（`符合怎样的标准`）在w3c标准中，**1.table是用来存储数据的。2.结构、样式和行为分离，带来足够好的可维护性。**
- **布局灵活**，能够通过样式选择来实现界面设计要求。
- **改版方便**，通常只要更换相应的css样式就可改变网页风格。
- **SEO**，table有表格嵌套问题，搜索引擎一般不抓取三层以上的表格嵌套。CSS的设计有助于SEO**让搜寻机器人更加轻易辨识网页内容**

