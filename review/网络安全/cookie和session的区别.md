

### Cookie和session的区别

1. **存放位置不同。**`cookie`数据存放在客户的浏览器上，session数据放在服务器上。
2. **安全性不同。**`cookie`对客户端可见，别人可以分析存放在本地的`COOKIE`并进行`COOKIE`欺骗，考虑到安全应当使用`session`。为了提高安全性，可以把Cookie信息加密，提交到服务器后再进行解密，保证`Cookie`中的信息只要本人能读得懂。
3. **服务器压力不同**。`session`会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用`COOKIE`。
4. 单个`cookie`保存的数据不能超过`4K`，很多浏览器都限制一个站点最多保存20个`cookie`。
5. **<font color='red'>有效期不同</font>**。`session`依赖于`cookie`，`cookie`中存储的`sessionId`的默认值为-1，只要关闭浏览器`session`就会失效，因而`session`不能完成信息永久有效的效果。假如设置`session`的超时时间过长，服务器累计的`session`过多会造成内存溢出。
6. **跨域支持上的不同**。**Cookie支持跨域名访问**，例如将`domain`属性设置为`“.biaodianfu.com”`，则以`“.biaodianfu.com”`为后缀的一切域名均能够访问该`Cookie`。跨域名`Cookie`如今被普遍用在网络中，例如`Google`、`Baidu`、`Sina`等。而`Session`则不会支持跨域名访问。**`Session`仅在他所在的域名内有效**。

### cookie的作用：

- **保存用户登录状态。**例如将用户`id`存储于一个`cookie`内，这样当用户下次访问该页面时就不需要重新登录了，现在很多论坛和社区都提供这样的功能。` cookie`还**可以设置过期时间**，当超过时间期限后，`cookie`就会自动消失。因此，系统往往可以提示用户保持登录状态的时间：常见选项有一个月、三个 月、一年等。
- **跟踪用户行为。**例如一个天气预报网站，能够根据用户选择的地区显示当地的天气情况。如果每次都需要选择所在地是烦琐的，当利用了`cookie`后就会显得很人性化了，系统能够记住上一次访问的地区，当下次再打开该页面时，它就会自动显示上次用户所在地区的天气情况。因为一切都是在后 台完成，所以这样的页面就像为某个用户所定制的一样，使用起来非常方便定制页面。如果网站提供了换肤或更换布局的功能，那么可以使用cookie来记录用户的选项，例如：背景色、分辨率等。当用户下次访问时，仍然可以保存上一次访问的界面风格。
- **标识用户身份。**`HTTP`是一个无状态协议，因此`Cookie`的最大的作用就是存储`sessionId`用来唯一标识用户

### cookie有哪些字段可以设置
##### name

`name`字段为一个`cookie`的名称。

##### value

`value`字段为一个`cookie`的值。

##### domain

`domain`字段为可以访问此`cookie`的域名。

非顶级域名，如二级域名或者三级域名，设置的cookie的domain只能为顶级域名或者二级域名或者三级域名本身，不能设置其他二级域名的cookie，否则cookie无法生成。

顶级域名只能设置`domain`为顶级域名，不能设置为二级域名或者三级域名，否则cookie无法生成。

二级域名能读取设置了`domain`为顶级域名或者自身的`cookie`，不能读取其他二级域名`domain`的`cookie`。所以要想`cookie`在多个二级域名中共享，需要设置`domain`为顶级域名，这样就可以在所有二级域名里面或者到这个cookie的值了。
顶级域名只能获取到domain设置为顶级域名的cookie，其他domain设置为二级域名的无法获取。

##### path

`path`字段为可以访问此`cookie`的页面路径。 比如`domain`是![img](file:///C:\Users\Admin\AppData\Local\Temp\%W@GJ$ACOF(TYDYECOKVDYB.png)abc.com,path是/test，那么只有/test路径下的页面可以读取此cookie。

🌟路径和域一起构成cookie的作用范围

##### expires/Max-Age

`expires/Max-Age` 字段为此cookie超时时间。

若设置其值为一个时间，浏览器就会把cookie保存到硬盘上，当到达此时间后，此`cookie`失效。

不设置的话`cookie`的生命周期为浏览器会话期间。当浏览器关闭(不是浏览器标签页，而是整个浏览器) 后，此`cookie`失效，这种cookie称为**会话cookie**，会话cookie一般不存储在硬盘上而是保存在内存里

`Size`字段 此`cookie`大小。

`http`字段  `cookie`的`httponly`属性。若此属性为`true`，则只有在`http`请求头中会带有此`cookie`的信息，而不能通过`document.cookie`来访问此`cookie`。

`secure` 字段 设置是否只能通过`https`来传递此条`cookie`

### cookie有哪些编码方式？
参考回答：
encodeURI（）
