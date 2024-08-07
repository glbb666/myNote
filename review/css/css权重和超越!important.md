## `css`样式优先级

```
！important>行间样式>id选择器>class选择器| 属性选择器|伪类选择器>标签选择器|伪元素>通配符
```

## `css`样式的权重值

| 项目                                                                    | 权重   |
| ----------------------------------------------------------------------- | ------ |
| ！important                                                             | 正无穷 |
| 行间样式（style）                                                       | 1000   |
| id选择器                                                                | 100    |
| class选择器/属性选择器(eg:input[type="radio"])/伪类选择器（eg :hover ） | 10     |
| 标签选择器（div）/伪元素（eg ::before）                                 | 1      |
| 通配符选择器（*）                                                       | 0      |

💥注意：`!important`权重值是虽然是正无穷，但是在计算机中表示的数字是有穷的，所以可以被权重高的 `!important`覆盖

## 优先级的基本规则

1. `!important `优先级最高，但也会被权重高的 `!important`所覆盖
2. 不同权重，权重值高则生效；
3. 相同权重：后面出现的选择器生效；
4. 就近原则：如果在 `CSS`文件中定义的样式 `A`，在 `html`文件定义了 `B`，则应用 `B`；
5. `CSS`否定伪类，`:not(x)`可以选择除某个元素之外的所有元素。
   `：not`否定伪类在优先级计算中不参加计算，但是，计算选择器数量时会把 `：not `当做普通选择器进行计数.

## 题目

1. 说出`<p>`的颜色

   ```html
   <style>
           .b{
               color: blue;
           }
           .a{
               color: red;
           }
   </style>
   <body>
       <p class="a b">1232</p>
   </body>
   ```
   p为红色

   ```html
   <style>
           .a{
               color: blue;
           }
           .b{
               color: red;
           }     
       </style>
   </head>
   <body>
       <p class="b a">1232</p>
   </body>
   ```
   p依然为红色，因为在style中，.a和.b的权重相同，所以后面的会覆盖掉前面的。和在 `html`中使用 `a`和 `b`的顺序无关。
