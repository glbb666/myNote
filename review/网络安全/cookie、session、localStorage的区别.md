### Cookie、sessionStorage、localStorage的区别

共同点：都是保存在浏览器端，并且是同源的

**Cookie**：cookie数据始终在同源的http请求中携带（即使不需要），即cookie在浏览器和服务器间来回传递。而`sessionStorage`和`localStorage`不会自动把数据发给服务器，仅在本地保存。`cookie`数据还有路径（path）的概念，可以限制`cookie`只属于某个路径下,存储的大小很小只有4K左右。 （key：可以在浏览器和服务器端来回传递，存储容量小，只有大约4K左右）

**sessionStorage**：仅在当前浏览器窗口关闭前有效，localStorage：始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据；cookie只在设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭。（key：本身就是一个回话过程，关闭浏览器后消失，session为一个会话，当页面不同即使是同一页面打开两次，也被视为同一次会话）

**localStorage**：localStorage 在所有**同源窗口中都是共享**的；cookie也是在所有**同源窗口中都是共享**的。（key：同源窗口都会共享，并且不会失效，不管窗口或者浏览器关闭与否都会始终生效）

