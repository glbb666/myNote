# 是什么

`transform` 允许你对元素进行 **2D 或 3D 空间变换** ，例如：平移、旋转、缩放、倾斜。

它不会影响周围元素的布局（元素在文档流中仍占据原始位置），只是视觉上被“变形”了。

# 基本语法

```css
selector {
  transform: 变换函数1 变换函数2 ...;
}
```

多个变换按**从右向左**的顺序应用（即先写后执行，但直观上按书写顺序作用于元素）。

# 常用 2D 变换函数

| 函数                            | 描述                                         | 示例                                   |
| ------------------------------- | -------------------------------------------- | -------------------------------------- |
| `translate(x, y)`             | 沿 X/Y 轴移动（px, %，百分比相对于自身宽高） | `transform: translate(50px, 100px);` |
| `translateX(n)`               | 仅沿 X 轴移动                                | `transform: translateX(20%);`        |
| `translateY(n)`               | 仅沿 Y 轴移动                                |                                        |
| `rotate(deg)`                 | 顺时针旋转（deg 或 rad）                     | `transform: rotate(45deg);`          |
| `scale(x, y)`                 | 缩放（宽度和高度倍数）                       | `transform: scale(2, 1.5);`          |
| `scaleX(n)` / `scaleY(n)`   | 单向缩放                                     |                                        |
| `skew(xdeg, ydeg)`            | 倾斜（沿 X 和 Y 轴的角度）                   | `transform: skew(10deg, 5deg);`      |
| `skewX(deg)` / `skewY(deg)` | 单向倾斜                                     |                                        |
| `matrix(a,b,c,d,e,f)`         | 6 值矩阵，组合所有 2D 变换                   | 高级用法                               |

# 重要关联属性

1. **`transform-origin`**
   改变变换的原点，默认为 `center`（`50% 50%`）。
   可设 `left` / `top` / `bottom` / `right` 或具体长度/百分比。

   ```css
   transform-origin: top left;     /* 左上角为原点 */
   transform-origin: 20px 30px;
   ```

# 实际示例与效果

## 示例 1：悬浮放大图标

```css
.icon {
  transition: transform 0.3s ease;
}
.icon:hover {
  transform: scale(1.2);
}
```

## 示例 2：旋转加平移（制作加载动画）

```css
.loader {
  animation: spin 1s linear infinite;
}
@keyframes spin {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}
```

## 注意事项

1. **不影响文档流** ：`transform` 只改变视觉表现，元素占位的原始位置不变，周围的元素不会重新排列。
2. **层级覆盖** ：变换后的元素可能覆盖其他元素（类似相对定位），可通过 `z-index` 配合 `position` 调整。
3. **性能** ：`transform` 和 `opacity` 会触发 **合成器（Compositor）** ，实现硬件加速，比改变 `top`/`left` 等几何属性性能更好（避免重排）。
4. **百分比单位** ：`translate` 中的百分比是相对于**自身元素**的宽高，而非父元素。
5. **行内元素** ：`transform` 对 `display: inline` 的元素无效，需设为 `display: inline-block `或 `block`。
6. **矩阵执行顺序** ：`transform: translate(100px) rotate(30deg);` 先旋转再平移？错！书写顺序与视觉应用顺序相反：先 rotate 再 translate。实际上矩阵从右向左乘。简单记忆： **最后写的变换先发生** 。
