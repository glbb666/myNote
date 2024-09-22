参考 [CSS Modules spec](https://github.com/css-modules/css-modules) 以查看更多详情，例如 [global exceptions](https://github.com/css-modules/css-modules/blob/master/docs/composition.md#exceptions) 和 [composition](https://github.com/css-modules/css-modules/blob/master/docs/composition.md#composition)。

# 什么是 CSS Modules？

CSS Modules 是一种 CSS 文件的模块化方案，允许你在项目中定义局部作用域的 CSS 类。它通过为每个类名生成独一无二的标识符，确保样式不会意外地在全局范围内被覆盖或影响其他组件。例如，原来的 `.btn` 类在打包后可能会变成 `Button_btn_1a2b3`，这就保证了类名的唯一性。

# 为什么 CSS Modules 很重要？

1. **避免样式冲突** ：在大型应用中，全局 CSS 容易发生样式覆盖问题。CSS Modules 通过作用域隔离，确保每个组件的样式都是独立的，避免了这种冲突。
2. **可维护性** ：由于样式是局部的，维护和更新样式变得更加简单，尤其是在多人协作和复杂项目中。
3. **复用性** ：CSS Modules 可以轻松地将样式与组件绑定，方便组件复用时不受外部影响。

# 如何使用 CSS Modules？

在 Webpack 项目中，你可以通过配置 `css-loader` 的 `modules` 选项来启用 CSS Modules：

## 配置步骤：

**Webpack 配置** ：
在 Webpack 配置文件中添加对 CSS Modules 的支持：

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          }
        ]
      }
    ]
  }
};

```

## **在文件中使用 CSS Modules** ：

在你的 React 或 Vue 组件中，可以通过 `import` 引入 CSS 文件，并将类名作为对象的属性来使用：

* **React** ：
  ```jsx
  import styles from './Button.module.css';

  function Button() {
    return <button className={styles.btn}>Click Me</button>;
  }

  ```
* **Vue** ：
  ```jsx
  <template>
    <button :class="$style.btn">Click Me</button>
  </template>

  <style module>
  .btn {
    background-color: blue;
  }
  </style>

  ```

## 具体功能

### **`composes` 的基础用法**

#### 是什么

`composes` 允许你将一个类的样式组合到另一个类中。

#### 为什么

方便复用样式。

#### 怎么做

##### 基本使用

`composes`必须放在其他样式规则之前。

```css
.className {
  color: green;
  background: red;
}

.otherClassName {
  composes: className; /* 引入 className 的样式 */
  color: yellow;
}

```

在上面的例子中，`.otherClassName` 通过 `composes` 引用了 `.className` 的样式，所以编译后的 `.otherClassName` 实际上会拥有 `.className` 的样式（`color: green; background: red;`），并且会叠加它自身的样式（`color: yellow`），最终效果如下：

```less
.otherClassName {
  background: red;
  color: yellow;
}

```

##### **复用伪类**

如果 `composes` 的类中有伪类（例如 `:hover`），这些伪类也会一并被复用。

```less
.className {
  color: green;
}

.className:hover {
  color: red;
}

.otherClassName {
  composes: className;
  background: black;
}

```

在这个例子中，`.otherClassName` 不仅复用了 `.className` 的基础样式（`color: green`），也复用了 `.className:hover` 的伪类样式（`color: red`）。因此，`.otherClassName` 最终的效果会是：

```less
.otherClassName {
  color: green;
  background: black;
}

.otherClassName:hover {
  color: red;
}

```

##### **从其他文件复用样式**

你可以通过 `composes` 从另一个 CSS 文件中复用样式：

 `style.css` 文件

```css
.className {
  color: red;
  font-size: 16px;
  padding: 10px;
}

```

在这个 `style.css` 文件中，我们定义了一个类 `.className`，它有一些基本的样式，比如 `color: red;` 和 `font-size: 16px;`。

在 `component.module.css` 文件中，我们通过 `composes` 从 `style.css` 文件中引入了 `.className`，并给 `.otherClassName` 增加了新的样式 `background-color: blue;`。

```less
.otherClassName {
  composes: className from './style.css'; /* 引入其他文件中的 className */
}

```

**注意：** 如果你从不同文件复用了多个类， **不要在多个类中定义相同的属性值** ，因为在这种情况下，样式应用的顺序是未定义的（无法保证哪个类的样式会生效）。

##### **避免循环依赖**

`composes` 不应该形成循环依赖。例如，`classA` 依赖 `classB`，而 `classB` 又依赖 `classA`。这样的情况可能导致样式覆盖的结果是未定义的，甚至会抛出错误。

##### **从全局类中复用样式**

有时你可能需要复用全局样式（即不是 CSS Modules 内部的局部样式），可以通过 `composes` 和 `global` 关键字来实现：

```css
.otherClassName {
  composes: globalClassName from global; /* 引入全局的 globalClassName */
}

```

##### **`global` 和 `local` 关键字**

在 CSS Modules 中，所有类默认都是局部的，但你可以使用 `:global` 来声明某些选择器为全局作用域。例如：

```css
:global(.some-selector) {
  color: blue;
}
```

上面的代码声明 `.some-selector` 为全局类，这意味着这个样式在整个项目的任何地方都能被应用到，而不是仅限于当前模块。

你也可以同时使用局部和全局的选择器：

假设我们有两个样式文件，一个是局部的 CSS Modules 文件，另一个是全局的普通 CSS 文件。

全局 CSS 文件 (`global.css`)

```less
/* global.css */
.global-header {
  background-color: blue;
  color: white;
  padding: 10px;
}

```

局部 CSS 文件 (`styles.module.css`)

`styles.module.css` 因为带有 `module` 后缀，会被构建工具（如 Webpack）自动按照 **CSS Modules** 的规则来解析，确保样式仅在当前模块内生效。

```css
/* styles.module.css */
.local-title {
  font-size: 20px;
  color: red;
}

//这里使用 `:global` 修改全局类，会影响项目中所有引用该全局类的文件。
.local-container :global(.global-header) {
  border: 2px solid green;
}

```

HTML 结构

```html
<div class="local-container">
  <h1 class="local-title">This is a local title</h1>
  <div class="global-header">
    This is a global header with some custom styles.
  </div>
</div>

```

##### 自定义注入名称

你可以通过给 `module` attribute 一个值来自定义注入 class 对象的属性名：

```html
<template>
  <p :class="classes.red">red</p>
</template>

<style module="classes">
.red {
  color: red;
}
</style>
```

##### 与组合式 API 一同使用

可以通过 `useCssModule` API 在 `setup()` 和 `<script setup>` 中访问注入的 class。

对于使用了自定义注入名称的 `<style module>` 块，`useCssModule` 接收一个匹配的 `module` attribute 值作为第一个参数：

```js
import { useCssModule } from 'vue'

// 在 setup() 作用域中...
// 默认情况下，返回 <style module> 的 class
useCssModule()

// 具名情况下，返回 <style module="classes"> 的 class
useCssModule('classes')
```
