

## cookie和session的区别

1. **存放位置不同。**`cookie`数据存放在客户的浏览器上，session数据放在服务器上。
2. **安全性不同。**`cookie`对客户端可见，别人可以分析存放在本地的`COOKIE`并进行`COOKIE`欺骗，考虑到安全应当使用`session`。为了提高安全性，可以把Cookie信息加密，提交到服务器后再进行解密，保证`Cookie`中的信息只要本人能读得懂。
3. **服务器压力不同**。`session`会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用`COOKIE`。
4. 单个`cookie`保存的数据不能超过`4K`，很多浏览器都限制一个站点最多保存20个`cookie`。
5. **<font color='red'>有效期不同</font>**。`session`依赖于`cookie`，`cookie`中存储的`sessionId`的默认值为-1，只要关闭浏览器`session`就会失效，因而`session`不能完成信息永久有效的效果。假如设置`session`的超时时间过长，服务器累计的`session`过多会造成内存溢出。
6. **跨域支持上的不同。Cookie支持跨域名访问**，例如将`domain`属性设置为`“.biaodianfu.com”`，则以`“.biaodianfu.com”`为后缀的一切域名均能够访问该`Cookie`。跨域名`Cookie`如今被普遍用在网络中，例如`Google`、`Baidu`、`Sina`等。而`Session`则不会支持跨域名访问。**`Session`仅在他所在的域名内有效**。

## Cookie

### cookie的作用：

- **保存用户登录状态。**存储用户`id`，下次用户访问该页面时就不需要重新登录了。`cookie`还**可以设置过期时间**，当超过时间期限后，`cookie`就会自动消失。因此，系统往往可以提示用户保持登录状态的时间：常见选项有一个月、三个 月、一年等。
- **跟踪用户行为。**例如一个天气预报网站，能够根据用户选择的地区显示当地的天气情况。如果每次都需要选择所在地是烦琐的，当利用了`cookie`后就会显得很人性化了，系统能够记住上一次访问的地区，当下次再打开该页面时，它就会自动显示上次用户所在地区的天气情况。因为一切都是在后 台完成，所以这样的页面就像为某个用户所定制的一样，使用起来非常方便定制页面。如果网站提供了换肤或更换布局的功能，那么可以使用cookie来记录用户的选项，例如：背景色、分辨率等。当用户下次访问时，仍然可以保存上一次访问的界面风格。
- **标识用户身份。**`HTTP`是一个无状态协议，因此`Cookie`的最大的作用就是存储`sessionId`用来唯一标识用户

### cookie有哪些字段可以设置，怎么设置

`name`：`cookie`的名称。

`value`：`cookie`的值。

`domain`：可以访问此`cookie`的域名。

`domain`只能向上读取和设置。

顶级域名只能获取和设置domain设置为顶级域名的cookie，否则cookie无法生成。二级域名或者三级域名，只能获取和设置domain设置为顶级域名或者二级域名或者三级域名本身，不能设置和读取其他二级域名的cookie。所以要想`cookie`在多个二级域名中共享，需要设置`domain`为顶级域名。

`path`:可以访问此`cookie`的页面路径。  这个属性设置的`url`且带有这个前缀的`url`路径都是有效的 。

🌟路径和域一起构成cookie的作用范围

`expires/Max-Age`:`cookie`超时时间。

若设置其值为一个时间，浏览器就会把cookie保存到硬盘上，当到达此时间后，此`cookie`失效。

不设置的话`cookie`的生命周期为浏览器会话期间。当浏览器关闭(不是浏览器标签页，而是整个浏览器) 后，此`cookie`失效，这种cookie称为**会话cookie**，会话cookie一般不存储在硬盘上而是保存在内存里

`Size`: `cookie`大小。

`httpOnly`：若此属性为`true`，则只有在`http`请求头中会带有此`cookie`的信息，而不能通过`js`来访问此`cookie`，能有效的防止`xss`攻击

> `XSS `攻击的原理是，攻击者插入一段可执行的 `JS` 脚本，该脚本会读出用户浏览器的 `cookies` 并将它传输给攻击者，攻击者得到用户的 `Cookies `后，即可冒充用户。

`secure` : 设置是否只能通过`https`来传递此条`cookie`

#### 客户端设置

document.cookie可以对cookie进行读写

```javascript
document.cookie = "name=xiaoming; age=12 "
```

客户端可以设置cookie的一下选项: expires, domain, path, secure(只有在`https`协议的网页中, 客户端设置secure类型cookie才能生效)，但无法设置`httpOnly`选项

#### 服务端设置

- response header中有一项叫`set-cookie`, 是服务端专门用来设置cookie的;
  - 一个set-cookie只能设置一个cookie, 当你想设置多个, 需要添加同样多的`set-cookie`
  - 服务端可以设置cookie的所有选项: expires, domain, path, secure, HttpOnly

### cookie有哪些编码方式？
`encodeURI（）`

## session

1.用户向服务器发送用户名和密码

2.服务器验证通过后，在当前对话(session)里面保存相关数据,比如用户角色, 登陆时间等;

3.服务器向用户返回一个`session_id`, 写入用户的`cookie`

4.用户随后的每一次请求, 都会通过`cookie`, 将`session_id`传回服务器

5.服务端收到 `session_id`, 找到前期保存的数据, 由此得知用户的身份

#### 存在的问题

- 扩展性不好，就比如实现单点登陆（ A网站和B网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录）
  - `nginx ip_hash`策略，服务端使用 `Nginx `代理，每个请求按访问 `IP `的 `hash` 分配，这样来自同一 IP 固定访问一个后台服务器，避免了在服务器 A 创建 Session，第二次分发到服务器 B 的现象。
  - `Session`复制：任何一个服务器上的 Session 发生改变（增删改），该节点会把这个 Session 的所有内容序列化，然后广播给所有其它节点。
  - 共享`Session`：将`Session_Id `集中存储到一个地方，所有的机器都来访问这个地方的数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败

另一种方案是服务器索性不保存session数据了，所有数据就保存在客户端，每次请求都发回服务器。这种方案就是接下来要介绍的基于Token的验证

## Token

#### 验证流程

1. 登录时候，客户端通过用户名与密码请求登录
2. 服务端收到请求后验证用户名与密码，服务端会签发一个Token,再把这个Token发给客户端
4. 客户端收到Token，存储到本地，如`Cookie`，`SessionStorage`，`LocalStorage`（如果需要跨域的话就用`cookie`进行存储，如把`domain`设置为`.a.com`，这样以`.a.com`结束的网站都可以对该token进行访问）

>这里的cookie只是一个存储机制，不是一个认证机制，所以没有`CSRF`攻击的风险

4. 客户端每次像服务器发起请求，都要带上Token。

5. 服务端收到请求，做解密和签名认证，判断其有效性

#### 为什么使用token

- token可以避开同源策略

- token可以避免[CSRF](https://www.cnblogs.com/shanyou/p/5038794.html)攻击

  >`CSRF`（Cross-site request forgery）跨站请求伪造：登陆受信任网站A并在本地生成cookie，在不登出A的情况下访问危险网站B。要抵御 `CSRF`，关键在于在请求中放入黑客所不能伪造的信息，并且该信息不存在于 cookie 之中。可以在 HTTP 请求中以参数的形式加入 `token`，并在服务器端建立一个拦截器来验证这个` token`，如果请求中没有` token` 或者` token `内容不正确，则认为可能是 `CSRF` 攻击而拒绝该请求。

- token可以是无状态的，可以在多个服务间共享

#### Token有效期

- 把token存在cookie里，就可以和cookie一样设置有效期

- 登陆成功后服务端给客户端发送一个 `Refresh Token`和`token`。服务端收到客户端请求时发现` token `过期，就反馈给前端，前端使用 `Refresh Token` 申请一个全新` token `继续使用。当然 `Refresh Token `也是有有效期的，但是这个有效期就可以长一点了，比如，以天为单位的时间。

  

![img](images/161375750d33b4cd)

![img](images/161375750d060f97)