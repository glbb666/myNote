### 简介

`em` 和` rem`都是灵活、 可扩展的单位，由浏览器转换为像素值，具体取决于你的设计中的字体大小设置。 

### 重点理解

区别：浏览器根据谁来转化成`px`值，理解这种差异是决定何时使用哪个单元的关键。

### `em`基本规则

 有一个普遍的误解，即：`em`是相对父级的字体大小。而根据`W3C`标准 ，它是相对于**使用em单位的元素的字体大小。**
 举个例子： 定义一个盒子，字体为`14px`,根据上述规则，`padding: 2x14=28px`; 如图

```css   
div{
       font-size:14px;
       padding:2em;
}
```

![在这里插入图片描述](https://img-blog.csdn.net/20181014202748183?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dhb3NoYW55YW5nemhpXzE5OTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### `em`父级影响

父级字体大小会对em有影响，实际上，这是由于其单位继承

- 只写一个div，样式只写padding:2em; div会继承根元素字体大小，即16px;此时padding为32px;如图

```css   
  div{
       padding:2em;
   }
```

   ![在这里插入图片描述](https://img-blog.csdn.net/20181014203744469?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dhb3NoYW55YW5nemhpXzE5OTk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)  

- 给上面的div再加一个父级div,设置其字体大小为1.25em，则父级div字体大小为1.25x16=20px;再看内层div，此时padding继承父级字体大小，为2x20=40px; 如图

```css
.father{           
    font-size: 1.25em; /*20px*/    
    width: 1em;        /*20px*/  
}             
.son{    
   font-size:1.5em;   /* 30px*/                 
   padding-top: 2em;  /*60px*/    
   height: 1em;       /*30px*/  
}  
```

- 给原始内层div加上字体大小设置：font-size:1.5em;内层font-size值为20x1.5=30px;padding-top值为30x2=60px;可见此时内层div的padding变成自身字体大小的2倍，**除高度宽度，不再受父级影响。**

```css
.father{           
     font-size: 1.25em;          
 }            
 .son{   
        font-size:1.5em;                 
        padding-top: 2em;            
  }
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181125202151901.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dhb3NoYW55YW5nemhpXzE5OTk=,size_16,color_FFFFFF,t_70)

### `rem `单位如何转换

- 取决于页根元素的字体大小，即 **`html `元素的字体大小**。 根元素字体大小乘以你 rem 值。
  例如，根元素的字体大小 `16px`，`10rem `将等同于` 160px`，即 10 x 16 = 160。

### 浏览器影响

- 根` html `元素将继承浏览器中设置的字体大小，除非显式设置固定值去覆盖。因此浏览器的字体大小设置可以影响每个使用 `rem` 单元以及每个通过 `em` 单位继承的值。
- 没有设置 `HTML` 字体大小时，浏览器设置起作用
- 如果网站的` html `元素的字体大小属性设置为` 1.25em`，根元素字体大小将为 `1.25` 倍的浏览器的字体大小设置。例：`20x1.25=25px`; 此时，若有一个div为`2rem`，则其像素值为`50px`;
- <font color='red'>谷歌浏览器的`font-size`不能比`12px`小，如果字体比`12px`小，就要用`css`的`scale`属性，用`scale`的缺点就是，虽然字体缩小了，但是依然占据着原来的位置。</font>
- <font color='red'>如果想把`html`设为`10px`方便计算，但是又想兼容`chrome`，可以尝试设置`html`字体为`100px`，body 修正为`16px`，这样 `0.1rem` 就是 `10px`，而body的字体仍然是默认大小，不影响未设置大小的元素的默认字体的大小。</font>

```css
div{
	font-size:12px;
	transform:scale(.5);
}
```

### rem和em差异

- `rem` 单位翻译为像素值是由 `html` 元素的字体大小决定的。 此字体大小会被浏览器中字体大小的设置影响，除非显式重写一个具体单位。 
- `em` 单位转为像素值，取决于他们使用的字体大小。 此字体大小受从父元素继承过来的字体大小，除非显式重写与一个具体单位。

### 用户改变浏览器字体大小

不能给`html`设置固定`px`值，这重写了继承用户设置的浏览器字体大小。需要更改` html `元素的字体大小时，那么就使用`em`，`rem`单位，这样根元素的值还会是用户浏览器字体大小的乘积。这将允许您通过更改您的` html `元素的字体大小，调整你的设计，但仍会保留用户的浏览器设置的效果。

### `vh` `vw`

`100vw`就是布局视口的宽度

`100vh`就是布局视口的高度