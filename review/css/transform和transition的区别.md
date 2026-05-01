`transform` 和 `transition` 是 CSS 中两个不同但经常一起使用的属性：

* **`transform`** ：用于 **改变元素的外观** （形状、位置、大小、角度等），例如旋转、缩放、平移、倾斜。它通常是**瞬间**完成的（除非配合 `transition` 或动画）。
* **`transition`** ：用于 **控制属性变化时的过渡效果** ，让变化过程平滑、有动画感。它可以作用于任何能变化的 CSS 属性（如 `width`、`opacity`、`color`，也包括 `transform`）。

# 核心区别

| 特性                   | `transform`             | `transition`                                  |
| ---------------------- | ------------------------- | ----------------------------------------------- |
| **作用**         | 定义元素“变成什么样”    | 定义变化“如何发生”（时间、速度曲线等）        |
| **是否产生动画** | 不产生动画，直接变化      | 产生动画，让变化平滑过渡                        |
| **依赖条件**     | 可直接应用，可独立存在    | 需要触发（如 `:hover`、类名变化）或属性值改变 |
| **可变化的属性** | 只改变自身的几何/空间属性 | 可对任何支持动画的 CSS 属性生效                 |

# 举例

```css
/* 单独使用 transform：瞬间旋转 45 度 */
.box:hover {
  transform: rotate(45deg);
}

/* 单独使用 transition：宽度的变化会有过渡 */
.box {
  transition: width 0.3s ease;
}
.box:hover {
  width: 200px;
}

/* 两者结合：旋转时产生平滑动画 */
.box {
  transition: transform 0.3s ease-in-out;
}
.box:hover {
  transform: rotate(45deg);
}
```

简单总结： **`transform` 改变终点状态，`transition` 控制到达终点的过程** 。

深度思考

智能搜索

内容由 AI 生成，请仔细甄别
