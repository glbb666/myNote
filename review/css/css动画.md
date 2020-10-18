## transition（4个属性）

过渡，顾名思义，一个元素的某个属性的值发生改变，这是一个状态的转变，**需要一种条件来触发这种转变**，比如我们平时用到的:hover、:focus、:checked、媒体查询或者JavaScript。

语法：`transition: property duration timing-function delay`

| 值                         | 描述                              |
| -------------------------- | --------------------------------- |
| transition-property        | 规定设置过渡效果的 CSS 属性的名称 |
| transition-duration        | 规定完成过渡效果需要多少秒或毫秒  |
| transition-timing-function | 规定速度效果的速度曲线            |
| transition-delay           | 定义过渡效果何时开始              |
📌`transition-property`：可以把值设置为`opacity`，`width`等属性，也可以把值设置成`transform`，`transform`是变形的意思，可以用`transform`进行用于元素进行**旋转、缩放、移动或倾斜**。

- 旋转(rotate)
- 缩放(scale)
- 移动(translate)
- 倾斜(skew)

### transition的特点

- 需要事件触发，所以没法在网页加载时自动发生
- 是一次性的，不能重复发生，除非一再触发
- 只能定义开始状态和结束状态，不能定义中间状态，也就是说只有两个状态
- 一条`transition`规则，只能定义一个属性的变化，不能涉及多个属性。如果需要设置多个属性，需要用`transition`内部的单个属性进行设置，如下。
### transition怎样监测多个属性?

可以在transition的属性中按顺序设置多个值。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>transition</title>
    <style>
        #box {
            height: 100px;
            width: 100px;
            background: rgb(86, 170, 209);
            transition-property: transform, border-radius, opacity;
            transition-duration: 1s, 2s, 3s;
            transition-timing-function: ease-in, ease-out, ease-in-out;
        }
        #box:hover {
            border-radius: 100px;
            opacity: .5;
            transform: rotate(-180deg) scale(.5, .5);
        }
    </style>
</head>
<body>
    <div id="box"></div>
</body>
</html>
```
> display 这个属性并不能添加 transition 效果,可以考虑使用 visibility 或者后边会提及的 animation

### transition 相关的事件

transitionend 事件会在 transition 动画结束的时候触发. [transitionend](https://developer.mozilla.org/en-US/docs/Web/API/TransitionEvent/TransitionEvent)

## animation（8个属性）

在官方的介绍上介绍这个属性是**transition属性的扩展**，transition只能够控制从一个状态到达另外一个状态，animation 可以控制多个状态的不断变化。

语法：`animation: name duration timing-function delay iteration-count direction play-state fill-mode;`

使用 `animation` 的前提是我们需要先使用 `@keyframes` 关键帧来定义一个动画效果，`@keyframes` 定义的规则可以用来控制动画过程中的各个状态的情况，语法是这个样子：

```css
@keyframes changebox {
  from { left: 0; top: 0; }
  to { left: 100%; top: 100%; }
}
```

`@keyframes` 关键词后跟动画的名字，然后是一个块，块中有动画进度的各个选择器，选择器后的块则依旧是我们常见的各个 CSS 样式属性。 **可以使用 from 或者 0% 来表示起始状态，而 to 或 100% 来表示结束状态。中间的部分你都可以使用百分比来进行表示。**

```css
.box {
    height: 100px;
    width: 100px;
    border: 15px solid black;
    animation: changebox 1s ease-in-out 1s infinite alternate running forwards;
    //动画changebox在1s后以1s/次的速度沿着ease-in-out的速度循环运行，永不停息，动画结束后动画停留在结束状态
}

.box:hover {
    animation-play-state: paused;
    动画停止
}
@keyframes changebox {
  from { left: 0; top: 0; }
  to { left: 100%; top: 100%; }
}
```


不是所有的属性都可以有动画效果，MDN 维护了一份 CSS [动画的属性列表](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animated_properties) 可供参考。



前四个属性和`transition`相同

| 值              | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| name            | 用来调用@keyframes定义好的动画，与@keyframes定义的动画名称一致 |
| duration        | 指定元素播放动画所持续的时间                                 |
| timing-function | 规定速度效果的速度曲线，是针对每一个小动画所在时间范围的变换速率 |
| delay           | 定义在浏览器开始执行动画之前等待的时间，值整个animation执行之前等待的时间 |
| iteration-count | 定义动画的播放次数，可选具体次数或者无限(infinite)           |
| direction       | 设置动画播放方向：normal(按时间轴顺序),reverse(时间轴反方向运行),alternate(轮流，即来回往复进行),alternate-reverse(动画先反运行再正方向运行，并持续交替运行) |
| play-state      | 控制元素动画的播放状态，通过此来控制动画的暂停和继续，两个值：running(继续)，paused(暂停) |
| fill-mode       | 控制动画结束后，元素的样式，有四个值：none(回到动画没开始时的状态)，forwards(动画结束后动画停留在结束状态)，backwords(动画回到第一帧的状态)，both(根据animation-direction轮流应用forwards和backwards规则)，注意与iteration-count不要冲突(动画执行无限次) |
### 优先级

优先级高的属性会覆盖优先级低的属性，当 animation 应用到元素中时，动画运行过程中，@keyframes 声明的 CSS 属性优先级最高，比行内声明 !important 的样式还要高。

### 多个动画的顺序

animation-name 可以指定多个动画效果，后指定的动画会覆盖掉前边的

例如：

```css
#colors {
  animation-name: red, green, blue; /* 假设这些 keyframe 都是修改 color 这个属性 */
  animation-duration: 5s, 4s, 3s;
}
```
上述代码的动画效果会是这样：前 3 秒是 blue，然后接着 1 秒是 green，最后 1 秒是 red。整个覆盖的规则是比较简单的。

> 注意：这个animation-name中的动画，是从后往前执行的，并且如果没有设置delay，这些动画将会同时执行

### display 的影响

如果一个元素的 display 设置为 none，那么在它或者它的子元素上的动画效果便会停止，而重新设置 display 为可见后，动画效果会重新重头开始执行。

### animation 相关事件
我们可以通过绑定事件来监听 animation 的几个状态，这些事件分别是：

- animationstart 动画开始事件，如果有 delay 属性的话，那么等到动画真正开始再触发，如果是没有 delay，那么当动画效果应用到元素时，这个事件会被触发。
- animationend 动画结束的事件，和 transitionend 类似。如果有多个动画，那么这个事件会触发多次，像上边的例子，这个事件会触发三次。如果 animation-iteration-count 设置为 infinite，那么这个事件则不会被触发。
- animationiteration 动画循环一个生命周期结束的事件，和上一个事件不一样的是，这个在每次循环结束一段动画时会触发，而不是整个动画结束时触发。无限循环时，除非 duration 为 0，否则这个事件会无限触发。

## transition和animition的区别

它们大部分属性相同，都是随时间而改变的属性值。但是`transition`需要事件触发才能改变属性，`animition`不需要触发任何事件，并且`transition`只有两帧，`animition`可以一帧一帧的。

## 实例

问题一：实现方形旋转缩放成圆的效果。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>transition</title>
    <style>
        #box {
            height: 100px;
            width: 100px;
            background: rgb(86, 170, 209);
            transition: transform 1s ease-in 0s;
        }
        #box:hover {
            transform: rotate(-180deg) scale(.5, .5);
        }
    </style>
</head>
<body>
    <div id="box"></div>
</body>
</html>
```

问题二：实现如下效果

![img](images/16abfb4e13651589)
即，一个小球从向右匀速移动 200px，然后移动回来，再移动过去，最后停留在 200px 处。

动图效果如下

![img](images/16abfbc339e758eb)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .ball{
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background-color: red;
            animation:move 5s linear 0s 3 alternate both;
            position: relative;
        }
        @keyframes move{
            from {
                transform: translate(0,0);
            }
            to{
                transform: translate(200px,0);
            }
        }
        
    </style>
</head>
<body>
    <div class='ball'></div>
</body>
</html>
```

还可以写成这样

```css
.ball{
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background-color: red;
    animation:move 5s linear both;
    position: relative;
}
@keyframes move{
    0% {
        transform: translate(0,0);
    }
    33% {
        transform: translate(200px,0);
    }
    66% {
        transform: translate(0px,0);
    }
    100% {
        transform: translate(200px,0);
    }
}
```

可以进一步简写

```css
@keyframes move{
            0%,66% {
                transform: translate(0,0);
            }
            33%,100% {
                transform: translate(200px,0);
            }
        }
```

问题三：

小球来回滚动，每次都在两边停留`800ms`再继续滚动

```css
.ball{
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background-color: red;
            animation:move 2s infinite alternate both;
            position: relative;
        }
        @keyframes move{
            0%,40% {
                transform: translate(0,0);
            }
            60%,100% {
                transform:translate(200px,0px);
            }
        }
```

