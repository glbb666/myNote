# 单文件组件 CSS 功能

## 组件作用域 CSS

当 `<style>` 标签带有 `scoped` attribute 的时z n候，它的 CSS 只会影响当前组件的元素。它的实现方式是通过 PostCSS 将以下内容：

```html
<style scoped>
.example {
  color: red;
}
</style>

<template>
  <div class="example">hi</div>
</template>
```

转换为：

```html
<style>
.example[data-v-f3f3eg9] {
  color: red;
}
</style>

<template>
  <div class="example" data-v-f3f3eg9>hi</div>
</template>
```

### 子组件的根元素

使用 `scoped` 后，父组件的样式将不会渗透到子组件中。

不过，**子组件的根节点会同时被父组件的作用域样式和子组件的作用域样式影响**。

这样设计是为了让父组件可以从布局的角度出发，调整其子组件根元素的样式。

### 深度选择器

处于 `scoped` 样式中的选择器如果想要做更“深度”的选择，也即：影响到子组件，可以使用 `:deep()` 这个伪类：

```html
<style scoped>
.a :deep(.b) {
  /* ... */
}
</style>
```

上面的代码会被编译成：

```css
.a[data-v-f3f3eg9] .b {
  /* ... */
}
```

通过 `v-html` 创建的 DOM 内容虽然不会被作用域样式影响，但你shu可以使用深度选择器来设置其样式。

### 插槽选择器

默认情况下，作用域样式不会影响到 `<slot/>` 渲染出来的内容，因为它们被认为是父组件所持有并传递进来的。使用 `:slotted` 伪类以明确地将插槽内容作为选择器的目标：

```html
<style scoped>
:slotted(div) {
  color: red;
}
</style>
```

### 全局选择器

如果想让其中一个样式规则应用到全局，比起另外创建一个 `<style>`，可以使用 `:global` 伪类来实现 (看下面的代码)：

```html
<style scoped>
:global(.red) {
  color: red;
}
</style>
```

### 混合使用局部与全局样式

你也可以在同一个组件中同时包含作用域样式和非作用域样式：

```html
<style>
/* 全局样式 */
</style>

<style scoped>
/* 局部样式 */
</style>
```

### 作用域样式须知

* **作用域样式并没有消除对 class 的需求** 。由于浏览器渲染各种各样 CSS 选择器的方式，`p { color: red }` 结合作用域样式使用时 (即当与 attribute 选择器组合的时候) 会慢很多倍。如果你使用 class 或者 id 来替代，例如 `.example { color: red }`，那你几乎就可以避免性能的损失。
* **小心递归组件中的后代选择器** ！对于一个使用了 `.a .b` 选择器的样式规则来说，如果匹配到 `.a` 的元素包含了一个递归的子组件，那么所有的在那个子组件中的 `.b` 都会匹配到这条样式规则。

## CSS Modules

一个 `<style module>` 标签会被编译为 [CSS Modules](https://github.com/css-modules/css-modules) 并且将生成的 CSS class 作为 `$style` 对象暴露给组件。

具体见css modules。

# css-modules和scoped标识符的区别是什么

## 1. **适用的框架和生态**

**CSS Modules** ：

* 适用于 **多种框架** ，如 React、Vue、Angular 等，甚至可以在不使用框架的项目中通过 Webpack 配置使用。
* 是一个 **独立的 CSS 模块化方案** ，并不依赖具体的框架实现。

**`scoped`（Vue 中的 scoped styles）** ：

* 是**Vue 特有的**功能，主要用于 Vue 组件中，其他框架（如 React）没有直接支持 `scoped`。
* **只适用于 Vue 单文件组件（SFC）** ，并依赖 Vue 的编译器来实现局部作用域的样式。

## 2. **工作原理**

**CSS Modules** ：

* 通过 Webpack 或其他构建工具在编译阶段为每个类名动态生成一个 **唯一的哈希标识符** 。例如，`button` 类可能会被编译为 `Button_button_1a2b3`。
* 每个类名在模块内部是独立的，不会影响到其他模块。

**`scoped`** ：

* Vue 的 scoped 样式会在编译时为每个组件的 DOM 元素添加一个 **自定义属性** ，例如：`data-v-1a2b3`。对应的样式选择器也会加上这个属性选择器，使样式只作用于带有该属性的元素。
* 例如，在 HTML 中 `button` 标签可能会变成 `<button data-v-1a2b3>`，CSS 中的选择器变为 `button[data-v-1a2b3]`。
* `scoped` 是通过 **属性选择器** （而非类名）来实现样式作用域的限制。

## 3. 示例

**CSS Modules** ：

* 需要借助构建工具（如 Webpack）进行配置，并通过**模块导入的方式**来使用样式。
* 样式通过 JavaScript 模块的形式绑定到组件：

```js
import styles from './Button.module.css';

function Button() {
  return <button className={styles.btn}>Click Me</button>;
}

```

* 编译后，CSS Modules 会将 `.btn` 类名编译成一个唯一的哈希值，例如：`Button_btn_1a2b3`。
* 最终的 HTML 类名会变成 `<button class="Button_btn_1a2b3">Click Me</button>`，从而确保样式只在这个组件中生效。

编译后的 CSS 可能会看起来像这样：

```css
.Button_btn_1a2b3 {
  background-color: blue;
  color: white;
  padding: 10px;
  border-radius: 5px;
}

```

**`scoped`** ：

* 在 Vue 的单文件组件中，直接通过 `<style scoped>` 定义局部作用域的样式，无需额外的配置。
* 样式与组件中的 HTML 标签自动关联：

```js
<template>
  <button>Click Me</button>
</template>

<style scoped>
button {
  background-color: blue;
}
</style>

```

编译后的 HTML 和 CSS

* Vue 的 `scoped` 样式会为 `button` 元素添加一个自定义属性，例如 `data-v-1a2b3`，确保样式仅作用于这个组件中的 `button` 元素。
* 最终的 HTML 可能会看起来像这样：

  ```html
  <button data-v-1a2b3>Click Me</button>
  ```
* 编译后的 CSS 选择器会加上 `data-v-1a2b3` 属性选择器：

  ```css
  button[data-v-1a2b3] {
    background-color: blue;
    color: white;
    padding: 10px;
    border-radius: 5px;
  }
  ```

## 4. **灵活性和可复用性**

**CSS Modules** ：

* 更加灵活，因为它是模块化的，样式可以轻松复用。你可以通过导入不同的 CSS 模块，甚至可以动态组合样式。
* 可以在多个组件中导入同一个 CSS 模块，具有更好的**跨组件复用**能力。

**`scoped`** ：

* 样式作用域是严格局限在当前组件的，样式复用相对困难。如果需要跨组件复用样式，通常需要使用全局样式或其他方法。

## 5. **性能对比**

**CSS Modules** ：

* 由于每个类名都是独立生成的唯一标识符，样式的应用更加高效。**样式的选择器优先级**也保持常规的 CSS 规则。

**`scoped`** ：

* 通过属性选择器来限制样式的作用域，可能会稍微增加浏览器渲染样式的时间，特别是在选择器较多时，**属性选择器**的性能比类选择器略低。

# CSS 中的 `v-bind()`

单文件组件的 `<style>` 标签支持使用 `v-bind` CSS 函数将 CSS 的值链接到动态的组件状态：

```html
<template>
  <div class="text">hello</div>
</template>

<script>
export default {
  data() {
    return {
      color: 'red'
    }
  }
}
</script>

<style>
.text {
  color: v-bind(color);
}
</style>
```

编译后的 HTML 和 CSS

在编译时，Vue 会将 `v-bind(color)` 转换为一个  **CSS 自定义属性** ，并且将其绑定到组件的根元素。编译后的代码可能会像这样：

 **HTML** ：

```html
<div class="text" style="--v-bind-color: red;">hello</div>

```

 **CSS** ：

```css
.text {
  color: var(--v-bind-color);
}

```

可以看到，Vue 将 `v-bind(color)` 编译为 `--v-bind-color` 自定义属性，接着使用 `var(--v-bind-color)` 来引用该属性。

这个语法同样也适用于 [`<script setup>`](https://cn.vuejs.org/api/sfc-script-setup.html)，且支持 JavaScript 表达式 (需要用引号包裹起来)：

```html
<script setup>
import { ref } from 'vue'
const theme = ref({
    color: 'red',
})
</script>

<template>
  <p>hello</p>
</template>

<style scoped>
p {
  color: v-bind('theme.color');
}
</style>
```

编译后的 HTML 和 CSS

Vue 同样会将 JavaScript 表达式转化为一个 CSS 自定义属性，并内联到根元素上：

 **HTML** ：

```html
<p style="--v-bind-theme-color: red;" data-v-1a2b3>hello</p>

```

 **CSS** ：

```css
p[data-v-1a2b3] {
  color: var(--v-bind-theme-color);
}

```

在这个例子中，`v-bind('theme.color')` 被编译为 `--v-bind-theme-color`，然后在 CSS 中通过 `var(--v-bind-theme-color)` 来动态应用颜色。
