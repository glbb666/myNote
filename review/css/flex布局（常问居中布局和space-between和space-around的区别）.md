## flex 基本概念

使用 flex 布局首先要设置父容器 `display: flex`，然后再设置 `justify-content: center` 实现水平居中，最后设置 `align-items: center` 实现垂直居中。

```css
#dad {
    display: flex;
    justify-content: center;
    align-items: center
}
```



![img](images/933e6f0857399ccf7e83.png)



就是这么简单，大功告成。等等，好像哪里不对，`justify-content` 和 `align-items` 是啥？哪里可以看出横向、竖向的语义？是的，flex 的确没有那么简单，这就要从两个基本概念说起了。

![img](images/221bb6de73e54f4104a1.png)



说来也不难，flex 的核心的概念就是 **容器** 和 **轴**。容器包括外层的 **父容器** 和内层的 **子容器**，轴包括 **主轴** 和 **交叉轴**，可以说 flex 布局的全部特性都构建在这两个概念上。flex 布局涉及到 12 个 `CSS` 属性（不含 `display: flex`），其中父容器、子容器各 6 个。不过常用的属性只有 4 个，父容器、子容器各 2 个，我们就先从常用的说起吧。

### 1. 容器

> 容器具有这样的特点：父容器可以统一设置子容器的排列方式，子容器也可以单独设置自身的排列方式，如果两者同时设置，以子容器的设置为准。



![img](images/f443b657dbc39d361f68.png)



#### 1.1 父容器

##### 设置子容器沿主轴排列：**justify-content**

`justify-content` 属性用于定义如何沿着主轴方向排列子容器。



![img](images/be5b7f0e022a8da60ed8.png)



> **flex-start**：起始端对齐



![img](images/ac1d8c5e7b4a2ba51ca7.png)



> **flex-end**：末尾段对齐



![img](images/9ec9245881c2882a35a6.png)



> **center**：居中对齐

![img](images/476461f1b9604a985046.png)

> **space-around**：子容器沿主轴均匀分布，位于首尾两端的子容器到父容器的距离是子容器间距的一半。

![img](images/63119c88aa64853107a9.png)





> **space-between**：子容器沿主轴均匀分布，位于首尾两端的子容器与父容器相切。



![img](images/495f46fc9c5c0c6d1e65.png)



##### 设置子容器如何沿交叉轴排列：**align-items**

`align-items` 属性用于定义如何沿着交叉轴方向分配子容器的间距。



![img](images/e7e6aa079f5333828c58.png)



> **flex-start**：起始端对齐



![img](images/56622862c7831a4d61be.png)



> **flex-end**：末尾段对齐



![img](images/33519955a141be1e713a.png)



> **center**：居中对齐

![img](images/f10513a47130d52f2aa8.png)





> **baseline**：基线对齐，这里的 `baseline` 默认是指首行文字，即 `first baseline`，所有子容器向基线对齐，交叉轴起点到元素基线距离最大的子容器将会与交叉轴起始端相切以确定基线。



![img](images/f78e9f42be9a3f165f8f.png)



> **stretch**：子容器沿交叉轴方向的尺寸拉伸至与父容器一致。



![img](images/160170b3d2022800ffea.png)



#### 1.2 子容器

##### 在主轴上如何伸缩：**flex**



![img](images/089d48122453e9fc372c.png)



  **子容器是有弹性的（flex 即弹性），它们会自动填充剩余空间，子容器的伸缩比例由 `flex` 属性确定。**

  `flex` 的值可以是无单位数字（如：1, 2, 3），也可以是有单位数字（如：`15px，30px，60px`），还可以是 `none` 关键字。子容器会按照 `flex` 定义的尺寸比例自动伸缩，如果取值为 `none` 则不伸缩。

  虽然 `flex` 是多个属性的缩写，允许 1 - 3 个值连用，但通常用 1 个值就可以满足需求，它的全部写法可参考下图。

![img](images/78e9030183f686e0b6ed.png)

flex的取值：

- `initial(flex: 0 1 auto)`元素不会伸长会缩短

- `auto(flex: 1 1 auto)`元素会伸长缩短

- `none(flex: 0 0 auto)`元素不会伸长缩短

🌟注意：

-  [`flex-basis`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex-basis) 属性默认值为 `auto`。若值为`0`，**则必须加上单位**，以免被视作伸缩性。指定了 `flex-basis` 后，`width` 属性被忽略、不再起作用。但`flex-basis`会受到`min-width`和`max-width`的制约。当使用一个或两个无单位数时, flex-basis会从auto变为0。

- `flex-basis` 表示在不伸缩的情况下子容器的原始尺寸。**主轴为横向时代表宽度，主轴为纵向时代表高度**。

##### 单独设置子容器如何沿交叉轴排列：**align-self**

![img](images/1d09fe5bb413a6dfa5dd.png)



  每个子容器也可以单独定义沿交叉轴排列的方式，此属性的可选值与父容器 `align-items` 属性完全一致，如果两者同时设置则以子容器的 `align-self` 属性为准。

> **flex-start**：起始端对齐

![img](images/93d138727b9dd780bdda.png)





> **flex-end**：末尾段对齐



![img](images/112f075777fdcb6f5d6f.png)



> **center**：居中对齐



![img](images/d7b0131447247a5228fe.png)



> **baseline**：基线对齐



![img](images/26b04323df92c4b1b023.png)



> **stretch**：拉伸对齐



![img](images/ef196e2ba84c406c9ad6.png)



### 2. 轴

如图所示，**轴** 包括 **主轴** 和 **交叉轴**，我们知道 `justify-content` 属性决定子容器沿主轴的排列方式，`align-items` 属性决定子容器沿着交叉轴的排列方式。那么轴本身又是怎样确定的呢？在 flex 布局中，`flex-direction` 属性决定主轴的方向，交叉轴的方向由主轴确定。



![img](images/5f2a17efffe8f3ab78a4.png)



##### 主轴

主轴的起始端由 `flex-start` 表示，末尾段由 `flex-end` 表示。不同的主轴方向对应的起始端、末尾段的位置也不相同。

> 向右：`flex-direction: row`



![img](images/da0c2a225cbbdba47297.png)



> 向下：`flex-direction: column`



![img](images/ab305a50ff35d7e7b6b4.png)



> 向左：`flex-direction: row-reverse`



![img](images/f3b60f80ddd45974449d.png)



> 向上：`flex-direction: column-reverse`



![img](images/c219413da157decc5b9e.png)



##### 交叉轴

主轴沿逆时针方向旋转 90° 就得到了交叉轴，交叉轴的起始端和末尾段也由 `flex-start` 和 `flex-end` 表示。

上面介绍的几项属性是 flex 布局中最常用到的部分，一般来说可以满足大多数需求，如果实现复杂的布局还需要深入了解更多的属性。

------

## flex 进阶概念

### 1. 父容器

- 设置换行方式：**flex-wrap**

  决定子容器是否换行排列，不但可以顺序换行而且支持逆序换行。



![img](images/19fb0f3a31fa497191b8.png)



> **nowrap**：不换行



![img](images/a41d1342e46cd37cd09e.png)



> **wrap**：换行



![img](images/0566bf9682ffa0890624.png)



> **wrap-reverse**：逆序换行

逆序换行是指沿着交叉轴的反方向换行。

![img](images/2f578fcc69919238bd3b.png)



- 轴向与换行组合设置：**flex-flow**

  flow 即流向，也就是子容器沿着哪个方向流动，流动到终点是否允许换行，比如 `flex-flow: row wrap`，`flex-flow` 是一个复合属性，相当于 flex-direction 与 flex-wrap 的组合，可选的取值如下：

  - `row`、`column` 等，可单独设置主轴方向
  - `wrap`、`nowrap` 等，可单独设置换行方式
  - `row nowrap`、`column wrap` 等，也可两者同时设置

- 多行沿交叉轴对齐：**align-content**

  当子容器多行排列时，设置行与行之间的对齐方式。



![img](images/ff9bd219375f048b3304.png)



> **flex-start**：起始端对齐



![img](images/0183db03d8fedadc4cf8.png)



> **flex-end**：末尾段对齐



![img](images/12e524438423ac7afc8c.png)



> **center**：居中对齐



![img](images/274a5c1282b997e423db.png)



> **space-around**：等边距均匀分布



![img](images/4a435e3fd0cab3433631.png)



> **space-between**：等间距均匀分布



![img](images/f50d931bdfeb6c24ccae.png)



> **stretch**：拉伸对齐



![img](images/878b39463db6bc499fbc.png)



### 2. 子容器

- 设置基准大小：**flex-basis**

  `flex-basis` 表示在不伸缩的情况下子容器的原始尺寸。**主轴为横向时代表宽度，主轴为纵向时代表高度**。

![img](images/af0dbf4ca6e857ff5de8.png)

![img](images/7c73d684a32fd8411db6.png)



- 设置扩展比例：**flex-grow**

  子容器弹性伸展的比例。如图，剩余空间按 1:2 的比例分配给子容器。



![img](https://lc-gold-cdn.xitu.io/bcca55b82d18e2ac2367.png?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://lc-gold-cdn.xitu.io/72e9f508dff25a474b40.png?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 设置收缩比例：**flex-shrink**

  子容器弹性收缩的比例。如图，超出的部分按 1:2 的比例从给子容器中减去。





![img](https://lc-gold-cdn.xitu.io/38596937d4f86beeac0b.png?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://lc-gold-cdn.xitu.io/d278e36c13b9643ff481.png?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





- 设置排列顺序：**order**

  改变子容器的排列顺序，覆盖 HTML 代码中的顺序，默认值为 0，可以为负值，数值越小排列越靠前。



![img](https://lc-gold-cdn.xitu.io/4eb20f9bfc611e66b069.png?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



------

  以上就是 flex 布局的全部属性，一共 12 个，父容器、子容器各 6 个，可以随时通过下图进行回顾。



![img](https://lc-gold-cdn.xitu.io/0dd26d8e99257ff36443.png?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

