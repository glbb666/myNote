## `CSRF` 跨站请求伪造

原理就是攻击者诱导用户在`B`网站对`A`网站通过点击或者其他途径发起请求，黑客利用**用户的登录态**对`A`网站的后端进行欺骗，网站`A`的后台就以为是用户在操作，从而进行相应的逻辑。 [![原理](https://github.com/zyg1999/Note/raw/master/review/web%E5%AE%89%E5%85%A8/pic/csrf.png)](https://github.com/zyg1999/Note/blob/master/review/web安全/pic/csrf.png)

## `CSRF`防御

- `SameSite`
  - 可以对 `Cookie` 设置 `SameSite` 属性。该属性表示 `Cookie` 不随着跨域请求发送，该属性兼容不好
- `Token`验证
  - `cookie`是发送时自动带上的，而不会主动带上Token，所以在每次发送时主动发送`Token`
- `Referer`验证
  - `Referer`是`header`的一部分，我们可以通过验证` Referer `来判断该请求是否为第三方网站发起的。
- 隐藏令牌
  - 主动在**HTTP头部中添加令牌信息**

## `CSRF`蠕虫

如果某个用户打开了被攻击网页，并且用户同时访问了攻击者的网页。 那么攻击者的网页就会使用用户的身份发送一些请求，并且常用用户的身份发布一些评论或文章，里面包含攻击者的网页链接。如果其他用户看到了这个用户的这条评论，都甚至可以不点击，其他用户也会被盗用身份发送一些恶意请求。这样病毒的传播就会越来越快，影响越来越大。

## `CSRF`攻击危害

- 利用用户登录态
- 用户不知情
- 完成业务请求
- 盗取用户资金
- 冒充用户发帖背锅
- 损坏网站名誉