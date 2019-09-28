### viewport
  浏览器的 [viewport](https://developer.mozilla.org/en-US/docs/Glossary/viewport) 是可以看到Web内容的窗口区域，通常与渲染出的页面的大小不同，这种情况下，浏览器会提供滚动条以滚动访问所有内容。

窄屏幕设备（如移动设备）在一个虚拟窗口或视口中渲染页面，这个窗口或视口通常比屏幕宽；然后缩小渲染的结果，以便在一屏内显示所有内容。然后用户可以移动、缩放以查看页面的不同区域。例如，如果移动屏幕的宽度为640px，则可能会用980px的虚拟视口渲染页面，然后缩小页面以适应640px的窗口大小。

这样做是因为许多页面没有做移动端优化，在小窗口渲染时会乱掉（或看起来乱）。所以，这种虚拟视口是一种让未做移动端优化的网站在窄屏设备上看起来更好的办法。  

### viewport分类
**layout viewport 布局视口**

CSS布局是相对于布局视口计算的，布局视口大小一般为浏览器**默认的viewport**，比如Safari iPhone使用980px，Opera 850px，Android WebKit 800px和IE 974px。

**visual viewport 可视视口**
当前屏幕显示的页面的一部分，可以通过缩放来更改可视视口的大小

**ideal viewport 理想视口**
宽度等于移动设备的屏幕宽度，不需要用户手动缩放和横向滚动条，都可以完美的呈现给用户

### 利用meta标签对viewport进行控制
移动设备默认的是layout viewpoint，可以使用meta标签来达到ideal viewpoint,使当前设备的viewpoint的宽度等于屏幕的宽度，同时禁止用户缩放
```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1,user-scalable=no">
```
- width控制视口的宽度
  
  + 直接指定确切的数字比如width=600
  
  + 指定width=device-width，代表缩放为100%时以`CSS像素`计量的宽度
- initial-scale：页面最初加载时的缩放值，为一个数字，可为浮点数
- maxinum-scale：允许用户的最大缩放值，为一个数字，可为浮点数
- mininum-scale：允许用户的最小缩放值，为一个数字，可为浮点数
- user-scalable：允许用户进行缩放，值为“no”或者"yes"
> 只设置<meta name="viewport" content="width=device-width">时，iphone和ipad上，无论是竖屏还是横屏，宽度都是竖屏时ideal viewport的宽度

> 只设置<meta name="viewport" content="initial-scale=1">时也能达到ideal viewpoint的效果，注意IE无论是竖屏还是横屏都把宽度设为竖屏时ideal viewport的宽度。

> 当width设置了数值并且使用了initial-scale=1，取值是两者中较大的数，注意在uc9浏览器中viewport的宽度永远都是ideal viewport的宽度

### 补充
**物理像素**

又被称为设备像素，是显示设备中一个最微小的物理部件（显示器上一个个的点）

**设备独立像素**

又称为密度无关像素，可以认为是计算机坐标系统上的一个点，这个点代表一个虚拟像素（比如说CSS像素），然后由相关系统转换为物理像素

**屏幕密度**

指一个设备屏幕存在的像素数量，通常以每英寸有多少像素来计算（PPI）

**CSS像素**

一般被称为与设备无关的像素(device-independent pixel)，简称DIPS，默认情况下等于一个物理像素，是一个相对值

当将页面放大一倍，其值代表的物理像素也会增加一倍

**devicePixelRatio**

设备像素比，简称dpr。值为设备物理像素/设备独立像素