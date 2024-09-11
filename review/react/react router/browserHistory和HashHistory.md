# 是什么

## **`BrowserRouter`**

`BrowserRouter` 是基于 HTML5 的历史 API（`history.pushState` 和 `history.replaceState`）的路由器，使用标准的 URL 结构进行导航。这意味着 URL 是 "干净的"，不会包含 `#` 符号。

* URL 示例：`http://example.com/about`
* `BrowserRouter` 直接修改浏览器的历史记录，而不会重新加载页面。
* 支持更复杂的路由配置，如嵌套路由、动态路由等。

## **`HashRouter`**

`HashRouter` 使用 URL 的哈希部分（`#`）来管理路由，所有的路径都在 `#` 之后。例如，访问 `/about` 页面时，实际的 URL 会变成：`http://example.com/#/about`。

* URL 示例：`http://example.com/#/about`
* `HashRouter` 不依赖浏览器的历史 API，因此即使浏览器不支持 HTML5 历史 API，它也能工作良好。

# 为什么

## **`BrowserRouter` 的优点和适用场景**

1. **干净的 URL** ：`BrowserRouter` 使用标准的 URL 结构，没有 `#` 符号，URL 更加简洁和美观，用户体验较好。
2. **SEO 友好** ：因为使用的是标准 URL 结构，`BrowserRouter` 可以更好地与服务器端渲染配合（如 `Next.js`），对于 SEO 和社交媒体分享非常有利。
3. **浏览器的前进/后退导航支持** ：`BrowserRouter` 能直接与浏览器的历史记录进行交互，支持前进、后退、刷新等操作，用户体验较自然。

 **适用场景** ：

* 适用于有服务器支持的应用。服务器需要配置，使得当用户直接访问某个非根路径时，服务器也能正确地返回 `index.html`，否则用户刷新页面时会遇到 404 错误。
* 适合需要 SEO 或在 URL 上做更高级操作的场景。

## **`HashRouter` 的优点和适用场景**

1. **无需服务器配置** ：`HashRouter` 不需要任何服务器端的额外配置，因为它依赖于 URL 的哈希部分，浏览器不会向服务器请求哈希路径。这意味着即使你直接访问某个页面或刷新页面，也不会导致 404 错误。
2. **兼容性好** ：对于不支持 HTML5 历史 API 的老浏览器（特别是一些移动设备上的旧浏览器），`HashRouter` 能正常工作，因为它只依赖于哈希部分。

 **适用场景** ：

* 适用于没有服务器端支持的应用，或是部署在静态文件服务器上（如 GitHub Pages），这样可以避免刷新页面时出现 404 错误。
* 更适合开发时不需要复杂服务器配置的简单应用，或应用对 SEO 不敏感。

# 如何选择和使用

* **选择 `BrowserRouter`** ：如果你的应用有服务器支持，且你希望 URL 美观，SEO 友好，那么 `BrowserRouter` 是更好的选择。前提是服务器要能处理所有路由并返回主页面（即适当的服务器配置，使得所有请求都指向 `index.html`，以便 React 来处理路由）。
* **选择 `HashRouter`** ：如果你不想配置服务器，或者你的应用是一个纯静态站点（例如部署在 GitHub Pages），那么 `HashRouter` 是更简单的解决方案。它在路由过程中不会和服务器交互，避免了因刷新或直接访问子路径时出现的 404 问题。

# 服务端为什么要配合BrowserRouter

`BrowserRouter` 可以修改浏览器地址栏的 URL，而不重新加载页面。

浏览器检测到url变化，就会向服务端请求。

默认情况下服务器会寻找 /xxx 路径对应的文件。由于这个路径通常并不存在（因为应用是单页的，只有一个 index.html以及它引用的静态资源）服务器会返回 404 错误。

所以需要在服务器上配置路由，将所有的请求（除了静态资源如 CSS、JS 文件）重定向到 index.html 文件。

# 服务端如何配合BrowserRouter

```js
const express = require('express');
const path = require('path');
const app = express();

app.use(express.static(path.join(__dirname, 'build')));

app.get('/*', function (req, res) {
  res.sendFile(path.join(__dirname, 'build', 'index.html'));
});

app.listen(3000);

```
