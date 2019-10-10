> 简单介绍属性特征及使用方法。 Canvas、SVG、SVG vs Canvas、Web 存储、服务器发送事件

## Canvas

用于使用 Javascript 在网页上绘制图形。

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

![在这里插入图片描述](<https://github.com/glbb666/myNote/blob/master/review/html/images/canvas_1.png>)

### 给Canvas设置宽高

我们先试试用css设置

```css
 canvas{
            width: 200px;
            height: 200px;
        }
```

![在这里插入图片描述](<https://github.com/glbb666/myNote/blob/master/review/html/images/canvas_2.png>)

矩形的宽高比例发生了改变！！！

因为canvas的默认宽高为300px*150px，在css中设置canvas的宽高，`实际上是把canvas在300px*150px的基础上进行了拉伸`。



### 给Canvas设置宽高

因为canvas的默认宽高为300px*150px，在css中设置canvas的宽高，实际上是**把canvas在300px * 150px的基础上进行了拉伸**。

![在这里插入图片描述](<https://github.com/glbb666/myNote/blob/master/review/html/images/canvas_1.png>)

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
>注意：设置宽高一定要在Canvas结点添加之后，在设置填充颜色，边框线条以及绘制图形之前，在设置宽高之后，填充颜色会被初始化为黑色。

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



### 更多方法

[🐷 canvas 教程 MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial)