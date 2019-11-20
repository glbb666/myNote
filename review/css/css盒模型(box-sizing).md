# CSS基础盒模型详解

## 前言

当对一个文档进行布局（lay out）的时候，浏览器的渲染引擎会根据标准之一的 **CSS 基础框盒模型**（**CSS basic box model**），将所有元素表示为一个个矩形的盒子（box）。CSS 决定这些盒子的大小、位置以及属性（例如颜色、背景、边框尺寸…）。

## 区分
- W3C标准盒模型	-content-box（标准盒模型）

  width = 内容的宽度
  
  height = 内容的高度
  
- IE盒模型	-border-box(怪异盒模型)

  width=border+padding+内容的宽度（border内部宽度）

  height=border+padding+内容的高度(border内部高度)
  
- inherit -从父元素继承`box-sizing`属性

> 在ie8+浏览器中使用哪个盒模型可以由**box-sizing**(CSS新增的属性)控制，默认值为content-box，即标准盒模型；如果将box-sizing设为border-box则用的是IE盒模型。如果在ie6,7,8中DOCTYPE缺失会触发IE模式。在当前W3C标准中盒模型是可以通过box-sizing自由的进行切换的。



  

