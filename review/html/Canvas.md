> 简单介绍属性特征及使用方法。 Canvas、SVG、SVG vs Canvas、Web 存储、服务器发送事件

## Canvas

用于使用 Javascript 在网页上绘制图形。

### 给Canvas设置宽高
因为canvas的默认宽高为300px*150px，在css中设置canvas的宽高，实际上是把canvas在300px*150px的基础上进行了拉伸。

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



### 更多方法

[🐷 canvas 教程 MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial)