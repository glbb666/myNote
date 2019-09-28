# 1px问题产生的原因

从移动端的角度说个具体的场景，以iphone6为例。

iphone6的屏幕宽度为375px，设计师做的`视觉稿`一般是750px，也就是2x，这个时候设计师在`视觉稿`上画了1px的边框，于是你就写了“`border-width:1px`”，**so...1px边框问题产生了**。

对设计师来说它的1px是相对于750px的（物理像素），对你来说你的1px是相对于375px的（css像素）“**实际上你应该是border-width:0.5px**”。

#  解决方案

## 用小数写边框
 由于retina 屏的浏览器可能不认识0.5px的边框，将会把它解释成0px，没有边框。包括 iOS 7 和之前版本，OS X Mavericks 及以前版本，还有 Android 设备。
**通过 JavaScript 检测浏览器能否处理0.5px的边框，如果可以，给html标签元素添加个class。**
以下是检测方法

```js
if (window.devicePixelRatio && devicePixelRatio >= 2) {
//当dpr>=2时1px问题才要进行考虑
  var testElem = document.createElement('div');
  testElem.style.border = '.5px solid transparent';
  document.body.appendChild(testElem);
}//创建一个检测元素，添加到body中
if (testElem.offsetHeight == 1) {
//offsetHeight = height+padding+border
//检测元素如果被设置0.5px的边框，height=0,border-top+border-bottom=1，offsetHeight=1
  document.querySelector('html').classList.add('hairlines');
}
  document.body.removeChild(testElem);
}
```
然后，极细的边框样式就容易了：
```css
div {
  border: 1px solid #bbb;
}
.hairlines div {
  border-width: 0.5px;
}
```
优点：
- 简单，不需要过多代码。
缺点：
- 无法兼容安卓设备、 iOS 8 以下设备。

## 使用boreder-image实现

准备一张符合你要求的border-image：

![img](https://upload-images.jianshu.io/upload_images/831991-4a025d5c907d26f0.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/167/format/webp)

样式设置：

```css
.border-bottom-1px {
  border-width: 0 0 1px 0;
  -webkit-border-image: url(linenew.png) 0 0 2 0 stretch;
  border-image: url(linenew.png) 0 0 2 0 stretch;
}
```

上文是把border设置在边框的底部，所以使用的图片是2px高，上部的1px颜色为透明，下部的1px使用视觉规定的border的颜色。如果边框底部和顶部同时需要border，可以使用下面的border-image：

![img](https://upload-images.jianshu.io/upload_images/831991-d1788f786488e080.png?imageMogr2/auto-orient/strip|imageView2/2/w/180/format/webp)

样式设置：

```css
.border-image-1px {
  border-width: 1px 0;
  -webkit-border-image: url(linenew.png) 2 0 stretch;
  border-image: url(linenew.png) 2 0 stretch;
}
```
到目前为止，我们已经能在iphone上展现1px border的效果了。但是我们发现这样的方法在非视网膜屏上会出现border显示不出来的现象，于是使用Media Query做了一些兼容，样式设置如下
```css
.border-image-1px {
  border-bottom: 1px solid #666;
}
@media only screen and (-webkit-min-device-pixel-ratio: 2) {
  .border-image-1px {
    border-bottom: none;
    border-width: 0 0 1px 0;
    -webkit-border-image: url(../img/linenew.png) 0 0 2 0 stretch;
    border-image: url(../img/linenew.png) 0 0 2 0 stretch;
  }
}
```

优点：

- 可以设置单条,多条边框
- 没有性能瓶颈的问题

缺点：
-  修改颜色麻烦, 需要替换图片
-  圆角需要特殊处理，并且边缘会模糊

## 使用background-image实现

background-image 跟border-image的方法一样，你要先准备一张符合你要求的图片。然后将边框模拟在背景上。
样式设置：

```css
.background-image-1px {
  background: url(../img/line.png) repeat-x left bottom;
  -webkit-background-size: 100% 1px;
  background-size: 100% 1px;
}
```

**优点：**

- 可以设置单条,多条边框
- 没有性能瓶颈的问题

**缺点：**

- 修改颜色麻烦, 需要替换图片
- 圆角需要特殊处理，并且边缘会模糊

## 多背景渐变实现

与background-image方案类似，只是将图片替换为css3渐变。设置1px的渐变背景，50%有颜色，50%透明。

```css
.background-gradient-1px {
  background:
    linear-gradient(#000, #000 100%, transparent 100%) left / 1px 100% no-repeat,
    linear-gradient(#000, #000 100%, transparent 100%) right / 1px 100% no-repeat,
    linear-gradient(#000,#000 100%, transparent 100%) top / 100% 1px no-repeat,
    linear-gradient(#000,#000 100%, transparent 100%) bottom / 100% 1px no-repeat
}
/* 或者 */
.background-gradient-1px{
  background:
    -webkit-gradient(linear, left top, right bottom, color-stop(0, transparent), color-stop(0, #000), to(#000)) left / 1px 100% no-repeat,
    -webkit-gradient(linear, left top, right bottom, color-stop(0, transparent), color-stop(0, #000), to(#000)) right / 1px 100% no-repeat,
    -webkit-gradient(linear, left top, right bottom, color-stop(0, transparent), color-stop(0, #000), to(#000)) top / 100% 1px no-repeat,
    -webkit-gradient(linear, left top, right bottom, color-stop(0, transparent), color-stop(0, #000), to(#000)) bottom / 100% 1px no-repeat
}
```

**优点：**

- 可以实现单条、多条边框
- 边框的颜色随意设置

**缺点：**

- 代码量不少
- 圆角没法实现
- 多背景图片有兼容性问题

## 使用box-shadow模拟边框
利用css 对阴影处理的方式实现0.5px的效果
样式设置：
```css
.box-shadow-1px {
  box-shadow: inset 0px -1px 1px -1px #c8c7cc;
}
```
**优点：**

- 代码量少
- 可以满足所有场景

**缺点：**

- 边框有阴影，颜色变浅

## viewport + rem

在devicePixelRatio = 2 时，输出viewport：

```html
<meta name="viewport" content="initial-scale=0.5, maximum-scale=0.5, minimum-scale=0.5, user-scalable=no">

```
在devicePixelRatio = 3 时，输出viewport：

```html
<meta name="viewport" content="initial-scale=0.3333333333333333, maximum-scale=0.3333333333333333, minimum-scale=0.3333333333333333, user-scalable=no">

```

**优点：**

- 所有场景都能满足
- 一套代码，可以兼容基本所有布局

**缺点：**

- 老项目修改代价过大，只适用于新项目

## transform: scale()+伪类标签

原理是把原先元素的 border 去掉，然后利用 :before 或者 :after 重做 border ，并 transform 的 scale 缩小一半，原先的元素相对定位，新做的 border 绝对定位

**单条border**

```css
.scale-1px{
  position: relative;
  border:none;
}
.scale-1px:after{
  content: '';
  position: absolute;
  bottom: 0;
  background: #000;
  width: 100%;
  height: 1px;
  -webkit-transform: scaleY(0.5);
  transform: scaleY(0.5);
  -webkit-transform-origin: 0 0;
  transform-origin: 0 0;
}
```

**四条**

```css
.scale-1px{
  position: relative;
  margin-bottom: 20px;
  border:none;
}
.scale-1px:after{
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  border: 1px solid #000;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  width: 200%;
  height: 200%;
  -webkit-transform: scale(0.5);
  transform: scale(0.5);
  -webkit-transform-origin: left top;
  transform-origin: left top;
}
```
最好在使用前也判断一下，结合 JS 代码，判断是否 Retina 屏：

```js
if(window.devicePixelRatio && devicePixelRatio >= 2){
  document.querySelector('ul').className = 'scale-1px';
}
```

**优点：**

- 所有场景都能满足
- 支持圆角(伪类和本体类都需要加border-radius)

**缺点：**

- 对于已经使用伪类的元素(例如clearfix)，可能需要多层嵌套

