### 文字立体投影 / 文字长阴影

`text-shadow`为文字添加阴影。可以为文字与  [`text-decorations`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-decoration)  添加多个阴影，阴影值之间用逗号隔开。每个阴影值由元素在X和Y方向的偏移量、模糊半径和颜色值组成。

正常而言，我们使用 text-shadow 来生成文字阴影，像这样：

```
<div> Txt Shadow</div>
-----
div {
    text-shadow: 6px 6px 3px hsla(14, 100%, 30%, 1);
}
复制代码
```



![image](https://user-gold-cdn.xitu.io/2018/11/6/166e6f02000ea2c7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



嗯，挺好的，就是不够立体。那么要做到立体文字阴影，最常见的方法就是使用多层文字阴影叠加。

> Tips：和 `box-shadow` 一样，`text-shadow` 是可以叠加多层的！但是对于单个元素而言， `drop-shadow` 的话就只能是一层。

好，上面的文字，我们试着叠加个 50 层文字阴影试一下。额，50 层手写，其实很快的~ 

![image](https://user-gold-cdn.xitu.io/2018/11/6/166e6f020737cf32?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



好吧，手写真的太慢了，还容易出错，所以这里我们需要借助一下 SASS/LESS 帮忙，写一个生成 50 层阴影的 `function` 就好，我们每向右和向下偏移 1px，生成一层 text-shadow：

```
@function makeLongShadow($color) {
    $val: 0px 0px $color;

    @for $i from 1 through 50 {
        $val: #{$val}, #{$i}px #{$i}px #{$color};
    }

    @return $val;
}

div {
    text-shadow: makeLongShadow(hsl(14, 100%, 30%));
}
复制代码
```

上面的 SCSS 代码。经过编译后，就会生成如下 CSS：

```
div {
      text-shadow: 0px 0px #992400, 1px 1px #992400, 2px 2px #992400, 3px 3px #992400, 4px 4px #992400, 5px 5px #992400, 6px 6px #992400, 7px 7px #992400, 8px 8px #992400, 9px 9px #992400, 10px 10px #992400, 11px 11px #992400, 12px 12px #992400, 13px 13px #992400, 14px 14px #992400, 15px 15px #992400, 16px 16px #992400, 17px 17px #992400, 18px 18px #992400, 19px 19px #992400, 20px 20px #992400, 21px 21px #992400, 22px 22px #992400, 23px 23px #992400, 24px 24px #992400, 25px 25px #992400, 26px 26px #992400, 27px 27px #992400, 28px 28px #992400, 29px 29px #992400, 30px 30px #992400, 31px 31px #992400, 32px 32px #992400, 33px 33px #992400, 34px 34px #992400, 35px 35px #992400, 36px 36px #992400, 37px 37px #992400, 38px 38px #992400, 39px 39px #992400, 40px 40px #992400, 41px 41px #992400, 42px 42px #992400, 43px 43px #992400, 44px 44px #992400, 45px 45px #992400, 46px 46px #992400, 47px 47px #992400, 48px 48px #992400, 49px 49px #992400, 50px 50px #992400;
}
复制代码
```

看看效果：



![image](https://user-gold-cdn.xitu.io/2018/11/6/166e6f02c7383ef3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



额，很不错，很立体。但是，就是丑，而且说不上来的奇怪。

问题出在哪里呢，阴影其实是存在明暗度和透明度的变化的，所以，对于渐进的每一层文字阴影，明暗度和透明度应该都是不断变化的。这个需求，SASS 可以很好的实现，下面是两个 SASS 颜色函数：

- `fade-out` 改变颜色的透明度，让颜色更加透明
- `desaturate` 改变颜色的饱和度值，让颜色更少的饱和

> 关于 SASS 颜色函数，可以看看这里：[Sass基础—颜色函数](https://link.juejin.im?target=https%3A%2F%2Fwww.w3cplus.com%2Fpreprocessor%2Fsass-color-function.html)

我们使用上面两个 SASS 颜色函数修改一下我们的 CSS 代码，主要是修改上面的 `makeLongShadow` function 函数：

```
@function makelongrightshadow($color) {
    $val: 0px 0px $color;

    @for $i from 1 through 50 {
        $color: fade-out(desaturate($color, 1%), .02);
        $val: #{$val}, #{$i}px #{$i}px #{$color};
    }

    @return $val;
}
复制代码
```

好，看看最终效果：



![image](https://user-gold-cdn.xitu.io/2018/11/6/166e6f02c9bcc1ab?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



嗯，大功告成，这次顺眼了很多~