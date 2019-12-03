> 简单介绍属性特征及使用方法。 Canvas、SVG、SVG vs Canvas、Web 存储、服务器发送事件

## Canvas是什么？

Canvas 元素创造了一块固定大小的画布，用于使用 Javascript 在网页上绘制图形。

### 绘制过程

1. 创建 canvas 元素

```js
  let canvas = document.getElementById('#myCanvas');
  // 或者
  let myCanvas = document.createElement('canvas');
  document.body.append(myCanvas);
```

2. 创建 context 对象

```js
  let cxt = myCanvas.getContext('2d');
```

> getContext("2d") 对象是内建的 HTML5 对象，拥有多种绘制路径、矩形、圆形、字符以及添加图像的方法。

3. 开始绘制(矩形)

```js
  cxt.fillStyle="#FF0000";//填充
  cxt.fillRect(0,0,150,75); //(x坐标，y坐标，宽度，高度)
```

![canvas_1](images/canvas_1.png)

### 给Canvas画布设置宽高

我们先试试用css设置

```css
 canvas{
            width: 200px;
            height: 200px;

}
```

矩形的宽高比例发生了改变！！！

因为`canvas`画布默认为`300px * 150px`，在`css`中设置`canvas`的宽高，实际上是在一个容器内把`canvas`画布进行压缩和拉伸，但是`canvas`宽度表示的还是`300px x 150px`

![canvas_2](images/canvas_2.png)

所以给Canvas设置宽高有**两种方法**

- html设置行内样式

```html
<canvas  width="200px" height="200px"id="myCanvas"></canvas>
```

- 在js中设置canvas的宽高

```js
myCanvas.width = 200;
myCanvas.height = 200;
```
>注意：在设置宽高之后，填充和线条颜色会被初始化为黑色。所以要先设置宽高。

完整代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        canvas { border: 1px solid black; }
    </style>
</head>
<body>
    <canvas id="myCanvas"></canvas>
</body>
<script>
    var  myCanvas= document.getElementById('myCanvas');
    if(myCanvas.getContext){
        
        var context = myCanvas.getContext('2d');
        myCanvas.width = 200;
	    myCanvas.height = 200;
        context.fillStyle="green";
        
        context.fillRect(10,10,150,100);
       
    }
</script>
</html>
```

### canvas进行图片压缩

##### drawImage(image, x, y, width, height)

其中 `image` 是 `image` 或者 `canvas` 对象，`x` 和 `y` 是其在目标 canvas 里的起始坐标，`width` 和 `height`，这用来控制缩放的图片大小

用`canvas`进行压缩的时候，图像的`dpi`不会发生变化，在把`canvas`转换为`Blob`的时候，可以设置第三个参数，这个参数可以改变`dpr`的大小。

> dpi：一英寸中像素点的数量

```javascript
var canvas = document.createElement('canvas');
var context = canvas.getContext('2d');
//获取图片的原始宽高
var originW = oImg.width;
var originH = oImg.height;
//设置图片压缩后的最大宽高
var maxW = 120, maxH = 120;
var targW = originW, targH = originH;
if(originW > maxW || originH > maxH) {
    if(originH/originW > maxH/maxW) {
        targH = maxH;
        targW = Math.round(maxH * (originW / originH));
    }else {
        targW = maxW;
        targH = Math.round(maxW * (originH / originW));
    }
}
//对图片进行缩放canvas.toblob
canvas.width = targW;
canvas.height = targH;
//清除画布，在画之前清是因为会存在多次绘制的情况
context.clearRect(0,0,targW,targH);
//图片压缩
context.drawImage(oImg,0,0,targW,targH);
canvas.toBlob(function(){},files.type || 'image/png'，quality)
```

### 更多方法

[🐷 canvas 教程 MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial)

### 用Canvas压缩图片模糊的问题

`canvas`压缩图片，假如把`10000 * 10000`的图片压缩成`120 * 120`的，图片的dpr是不会发生变化的，但是比较图片是否清晰，是建立在图片大小相等的基础上进行比较，如果我需要的是`200 * 200`的图片，那我这个`120 * 120`的图片就肯定需要进行放大，这时就会出现模糊的问题。所以我们解决的方案是，让用户自己选择压缩图片的大小，这样可以确保缩小的图片不因为放大出现模糊的问题，用户还可以传入`quality`参数设定`dpr`（`quality`参数的默认值是`0.92`），这样就可以根据自己的要求获得清晰或者模糊的图片了。

### Canvas适配高倍屏

因为canvas会依赖像素，所以在高倍屏(`dpr>1`)canvas图片会出现模糊问题，我们需要将**canvas放大到设备像素比来绘制，最后将canvas压缩成一倍的物理大小来显示。**

将canvas放在高倍屏绘制，最后将canvas压缩成一倍的物理大小来显示

用canvas.width和canvas.height设置canvas的画布大小

用canvas.style.width和canvas.style.height设置画布外面的容器大小

```javascript
//把容器大小设置成画布大小
myCanvas.style.width = myCanvas.width
myCanvas.style.height = myCanvas.height
//把画布放大为设备像素比
myCanvas.width = myCanvas.width*2;
myCanvas.height = myCanvas.height*2;
context.scale(2, 2);//渲染区域放大,等同于把drawImage的width*2和height*2
context.drawImage(image,0,0,width,height)
```

## SVG

### 什么是SVG?

SVG 指可伸缩矢量图形，它用` XML` 格式定义图形，放大或改变尺寸的情况下其图形质量不会有损失，SVG 是万维网联盟的标准

### SVG 的优势

与其他图像格式相比（比如 JPEG 和 GIF），使用 SVG 的优势在于：

- SVG 图像可通过文本编辑器来创建和修改
- SVG 图像可被搜索、索引、脚本化或压缩
- SVG 是可伸缩的
- SVG 图像可在任何的分辨率下被高质量地打印
- SVG 可在图像质量不下降的情况下被放大

### 更多

[🌟 SVG 深入浅出](https://github.com/luoping1998/Frontend-Nodes/blob/master/坑路记载/svg.md)

## SVG VS Canvas

### SVG

> SVG 是一种使用 XML 描述 2D 图形的语言。

SVG 基于 XML，这意味着 SVG DOM 中的每个元素都是可用的。您可以为某个元素附加 JavaScript 事件处理器。

在 SVG 中，每个被绘制的图形均被视为对象。如果 SVG 对象的属性发生变化，那么浏览器能够自动重现图形。

### Canvas

> Canvas 通过 JavaScript 来绘制 2D 图形。

Canvas 是逐像素进行渲染的。

在 canvas 中，一旦图形被绘制完成，它就不会继续得到浏览器的关注。如果其位置发生变化，那么整个场景也需要重新绘制，包括任何或许已被图形覆盖的对象。

### Canvas 与 SVG 的比较

#### Canvas

- 依赖分辨率
- 不支持事件处理器
- 弱的文本渲染能力
- 能够以 .png 或 .jpg 格式保存结果图像
- 最适合图像密集型的游戏，其中的许多对象会被频繁重绘

#### SVG

- 不依赖分辨率
- 支持事件处理器
- 最适合带有大型渲染区域的应用程序（比如谷歌地图）
- 复杂度高会减慢渲染速度（任何过度使用 DOM 的应用都不快）
- 不适合游戏应用