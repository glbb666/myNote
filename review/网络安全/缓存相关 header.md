[面试精选之http缓存(轻松易读梳理)](https://juejin.im/post/5b3c87386fb9a04f9a5cb037)

# Expries

**`Expires`响应头**包含日期/时间， 即在此时候之后，响应过期。

无效的日期，比如 0, 代表资源已经过期。

如果在`Cache-Control`响应头设置了`max-age`或者`s-max-age`指令，那么 `Expires` 头会被忽略。

> http-equiv顾名思义，相当于http的文件头作用，它可以向浏览器传回一些有用的信息，以帮助正确和精确地显示网页内容，与之对应的属性值为content，content中的内容其实就是各个参数的变量值。

```html
//也可以设置在在html文件头中
<meta http-equiv="expires" content="Thu, 30 Nov 2017 11:17:26 GMT">
```

缺点：Expires 过期控制不稳定，因为浏览器端可以随意修改时间，导致缓存使用不精准。

为了增加相对时间的控制，引入了Cache-Control

# Cache-control

## 语法
##### 缓存请求指令

```
Cache-Control: max-age=<seconds>
Cache-Control: max-stale[=<seconds>]
Cache-Control: min-fresh=<seconds>
Cache-control: no-cache 
Cache-control: no-store
Cache-control: no-transform
Cache-control: only-if-cached
```

##### 缓存响应指令

```
Cache-control: must-revalidate
Cache-control: no-cache
Cache-control: no-store
Cache-control: no-transform
Cache-control: public
Cache-control: private
Cache-control: proxy-revalidate
Cache-Control: max-age=<seconds>
Cache-control: s-maxage=<seconds>
```

##### 拓展指令

拓展缓存指令不是核心HTTP缓存标准文档的一部分，使用前请注意检查[兼容性](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control#Browser_compatibility)！

```
Cache-control: immutable 
Cache-control: stale-while-revalidate=<seconds>
Cache-control: stale-if-error=<seconds>
```

## 指令

##### 可缓存性

```
public
```

表明响应可以被任何对象（包括：代理服务器）缓存，即使是通常不可缓存的内容（例如，该响应没有`max-age`指令或`Expires`消息头）。

```
private
```

表明响应只能被客户端缓存，不能被代理服务器缓。私有缓存可以缓存响应内容。

```
no-cache
```

在发布缓存副本之前，强制要求缓存把请求提交给原始服务器进行验证。

```
no-store
```

缓存不应存储有关客户端请求或服务器响应的任何内容。

##### 到期

- `max-age=<seconds>`

  **设置缓存存储的最大周期**，超过这个时间缓存被认为过期(单位秒)。时间是<font color='red'>相对于请求的时间</font>

  ✨特别：某些浏览器（比如Firefox）中如果设定为永不缓存，那么其发出的请求中，请求头会包含`max-age=0`。

- `s-maxage=<seconds>`

  覆盖`max-age`或者`Expires`头，但是仅适用于共享缓存(比如各个代理)，私有缓存会忽略它。

- `max-stale[=<seconds>]`

  表明客户端愿意接收一个已经过期的资源。可以设置一个可选的秒数，表示响应不能已经过时超过该给定的时间。是从代理服务器获取资源。

- `min-fresh=<seconds>`

  表示客户端希望获取一个能在指定的秒数内保持其最新状态的响应。

- `stale-while-revalidate=<seconds>` 

  表明客户端愿意接受陈旧的响应，同时在后台异步检查新的响应。秒值指示客户愿意接受陈旧响应的时间长度。

- `stale-if-error=<seconds>` 

  表示如果新的检查失败，则客户愿意接受陈旧的响应。秒数值表示客户在初始到期后愿意接受陈旧响应的时间。

##### 重新验证和重新加载

- `must-revalidate`

  一旦资源过期（比如已经超过`max-age`），在成功向原始服务器验证之前，缓存不能用该资源响应后续请求。

- `proxy-revalidate`

  与must-revalidate作用相同，但它仅适用于共享缓存（例如代理），并被私有缓存忽略。

- `immutable` 

  表示响应正文不会随时间而改变。资源（如果未过期）在服务器上不发生改变，因此客户端不应发送重新验证请求头（例如`If-None-Match`或I`f-Modified-Since`）来检查更新，即使用户显式地刷新页面。

##### 其他

- `no-transform`

  不得对资源进行转换或转变。`Content-Encoding`、`Content-Range`、`Content-Type`等HTTP头不能由代理修改。例如，非透明代理可能对图像格式进行转换，以便节省缓存空间或者减少缓慢链路上的流量。`no-transform`指令不允许这样做。

- `only-if-cached`

  表明客户端只接受已缓存的响应，并且不要向原始服务器检查是否有更新的拷贝。cache要么用缓存的内容给出响应，要么给出一个504（GateWay Timeout）响应码。如果一组cache被作为一个内部相连的系统，那么其中的某个成员可以向这个缓存组里请求响应。
# Last-modified和If-modified-since和If-Unmodified-Since

### Last-Modified(HTTP/1.0)

响应头，包含源服务器认定的资源修改的日期及时间。 通常被用作一个验证器来判断接收到的和存储的资源是否彼此一致。由于精确度比`ETag`要低，经常作为一个备用机制。包含有`If-Modified-Since`或`If-Unmodified-Since`首部的条件请求会使用这个字段。
### If-Modified-Since(HTTP/1.0)

条件式请求头，等于上一次请求的Last-Modified。服务器比较请求头里的 Last-Modified 时间和服务器上 a.js的上次修改时间：

- 如果一致，则告诉浏览器：你可以继续用本地缓存。返回一个不带有消息主体的`304` 响应
- 如果不一致，返回一个带有最近修改时间Last-Modified以及过期时间Expries以及消息主体的`200`响应。

不同于`If-Unmodified-Since`, `If-Modified-Since` <font color='red'>只可以用在`GET`或`HEAD`请求中</font>。

当与`If-None-Match`一同出现时，会被忽略掉，除非服务器不支持 `If-None-Match`。

最常见的应用场景是来更新没有特定`ETag`标签的缓存实体。

### If-Unmodified-Since

条件式请求头：只有当资源在指定的时间之后没有进行过修改的情况下，服务器才会返回请求的资源。或是接受`POST`或其他`non-safe`方法的请求。如果所请求的资源在指定的时间之后发生了修改，那么会返回`412`(Precondition Failed/先决条件失败）错误。

常见的应用场景有两种：

- 与`non-safe`方法如`POST`搭配使用，可以用来优化并发控制，例如在某些wiki应用中的做法：假如在原始副本获取之后，服务器上所存储的文档已经被修改，那么对其作出的编辑会被拒绝提交。
- 与含有`If-Range`消息头的范围请求搭配使用，用来确保新的请求片段来自于未经修改的文档。

### 缺点

Last-Modified 过期时间只能精确到秒。

精确到秒存在两个问题：

- 浏览器给资源设置无缓存，资源在一秒内修改，浏览器拿不到最新资源。
- 资源打开并保存，但没有修改也会更新`last-modified`的时间，服务器会因为时间匹配不上重新返回资源主体。

为了增加文件内容对比，引入了Etag

# Etag和If-None-Match和If-Match

### Etag

`Etag`响应头，是资源的特定版本的标识符。资源内容不改变，`Etag`不变。使用`ETag`和`If-match`有助于防止资源的同时更新相互覆盖（“空中碰撞”）。

**语法**

```javascript
//弱Etag值：根本发生变化，Etag值才会改变
ETag: W/"<etag_value>"
//例如：ETag: W/"0815"
//强Etag值：无论内容发生多么细微的改变都会改变其值
ETag: "<etag_value>"
//例如：ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

**指令**

- `W/` 可选

  `'W/'`(大小写敏感) 表示使用弱验证器。 弱验证器很容易生成，但不利于比较。 强验证器是比较的理想选择，但很难有效地生成。 相同资源的两个弱`Etag`值可能语义等同，但不是每个字节都相同。

- "<etag_value>"

  实体标签，唯一地表示所请求的资源。 它们是位于双引号之间的ASCII字符串（如“675af34563dc-tr34”）。没有明确指定生成ETag值的方法，通常，把内容通过散列算法得到它们。最后修改时间戳的哈希值，或简单地使用版本号。例如，MDN使用wiki内容的十六进制数字的哈希值。

### If-none-match(配合Etag缓存未更改的资源)

请求头。上次请求同一资源时，服务器返回给浏览器的Etag。

对于`GET`和`HEAD`请求方法来说

- 当且仅当服务器上没有任何资源的`ETag`属性值与If-none-match中列出的相匹配的时候，服务器端会才返回所请求的资源，响应码为`200` 。

- 当验证失败的时候，服务器端必须返回响应码 304（Not Modified，未改变）。对于能够引发服务器状态改变的方法，则返回 412 （Precondition Failed，前置条件失败）。

对于其他方法来说，当且仅当最终确认没有已存在的资源的`ETag`属性值与这个首部中所列出的相匹配的时候，才会对请求进行相应的处理。

##### Notes

服务器端在生成状态码为 304 的响应的时候，<font color='red'>必须同时生成以下会存在于对应的 200 响应中的首部</font>：`Cache-Control`、`Content-Location`、`Date`、`ETag`、`Expires` 和 `Vary`。

`ETag`属性之间的比较采用的是<font color='red'>**弱比较算法**</font>，即两个文件除了每个比特都相同外，内容一致也可以认为是相同的。例如，如果两个页面仅仅在页脚的生成时间有所不同，就可以认为二者是相同的。

当与`If-Modified-Since`一同使用的时候，`If-None-Match`优先级更高（假如服务器支持的话）。

##### 两个常见的应用场景：

- 采用`GET`或`HEAD`方法，来**更新**拥有特定的`ETag`属性值的**缓存**。
- 采用其他方法，尤其是`PUT`,将 `If-None-Match` used 的值设置为 * ，用来生成事先并不知道是否存在的文件，可以确保先前并没有进行过类似的上传操作，防止之前操作数据的丢失。这个问题属于更新丢失问题的一种。

### If-match（配合Etag避免“空中碰撞”）

请求首部。在请求方法为`GET`和`HEAD`的情况下，服务器仅在请求的资源满足此首部列出的 `ETag `之一时才会返回资源。而对于`PUT`或其他非安全方法来说，只有在满足条件的情况下才可以将资源上传。

##### Notes

`ETag`存储比较使用的是**强比较算法**，即只有在每一个比特都相同的情况下，才认为两个文件是相同的。采用相对宽松的算法可以在`ETag`前面添加`W/`。

##### 两个常见的应用场景：

- 对于`GET`和`HEAD`方法，搭配`Range`请求头使用，可以用来**保证**新请求的范围与之前请求的范围**是对同一份资源的请求**。如果 ETag 无法匹配，那么需要返回`416` (Not Satisfiable，范围请求无法满足) 响应。
- 对于其他方法来说，尤其是`PUT`, `If-Match` 首部可以**用来避免更新丢失问题**。它可以用来检测用户想要上传的不会覆盖获取原始资源之后做出的更新。如果请求的条件不满足，那么需要返回 `412`(Precondition Failed，先决条件失败) 响应。

