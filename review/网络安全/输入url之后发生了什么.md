## 大致流程

1. URL 解析
2. DNS 查询
3. TCP 连接
4. 服务器处理请求
5. 浏览器接受响应
6. 渲染页面

## 一、URL 解析

**地址解析：**

首先判断你输入的是一个合法的 URL 还是一个待搜索的关键词，并且根据你输入的内容进行自动补全、字符编码等操作。

**HSTS**

由于安全隐患，会使用 HSTS 强制客户端使用 HTTPS 访问页面。详见：[你所不知道的 HSTS](https://www.barretlee.com/blog/2015/10/22/hsts-intro/)。

**其他操作**

浏览器还会进行一些额外的操作，比如安全检查、访问限制（之前国产浏览器限制 996.icu）。

**检查缓存**

先检查本地有没有缓存，如果没有缓存则向服务器请求数据，如果有缓存，检查缓存有没有过期，如果缓存过期了，就携带标识(这一块涉及到强制缓存和对比缓存)向服务器发起请求，服务器会

https://github.com/glbb666/myNote/blob/master/review/%E7%BD%91%E7%BB%9C%E5%AE%89%E5%85%A8/image/url.png](https://github.com/glbb666/myNote/blob/master/review/网络安全/image/url.png)

## 二、DNS 查询

