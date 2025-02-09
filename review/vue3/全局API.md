# 全局 API：常规

## version

暴露当前所使用的 Vue 版本。

```js
import { version } from 'vue'

console.log(version)
```

## nextTick()

### 是什么

等待下一次 DOM 更新刷新的工具方法。

### 为什么

当你在 Vue 中更改响应式状态时，最终的 DOM 更新并不是同步生效的，而是由 Vue 将它们缓存在一个队列中，直到下一个“tick”才一起执行。这样是为了确保每个组件无论发生多少状态改变，都仅执行一次更新。

### 怎么做

`nextTick()` 可以在状态改变后立即使用，以等待 DOM 更新完成。你可以传递一个回调函数作为参数，或者 await 返回的 Promise。

```ts
function nextTick(callback?: () => void): Promise<void>
```

**示例**

```html
<script setup>
import { ref, nextTick } from 'vue'

const count = ref(0)

async function increment() {
  count.value++

  // DOM 还未更新
  console.log(document.getElementById('counter').textContent) // 0

  await nextTick()
  // DOM 此时已经更新
  console.log(document.getElementById('counter').textContent) // 1
}
</script>

<template>
  <button id="counter" @click="increment">{{ count }}</button>
</template>
```

## defineComponent()

### 是什么

在定义 Vue 组件时提供类型推导的辅助函数。

#### 1. 选项式 API

这是 Vue 传统的组件定义方式，使用一个组件选项对象：

##### 语法

```ts
function defineComponent(
  component: ComponentOptions
): ComponentConstructor
```

> 为了便于阅读，对类型进行了简化。

##### 例子

```js
import { defineComponent } from 'vue';

const MyComponent = defineComponent({
  data() {
    return {
      message: 'Hello World'
    };
  },
  methods: {
    greet() {
      console.log(this.message);
    }
  }
});
```

#### 2. 组合式 API (需要 Vue 3.3+)

这种方式更适合与组合式 API 和渲染函数或 JSX 一起使用：

##### 语法

```ts
// 函数语法 (需要 3.3+)
function defineComponent(
  setup: ComponentOptions['setup'],
  extraOptions?: ComponentOptions
): () => any
```

> 为了便于阅读，对类型进行了简化。

##### 例子

```ts
import { defineComponent, ref, h } from 'vue';

const MyComponent = defineComponent(
  (props) => {
    const count = ref(0);

    return () => h('div', count.value);
  },
  {
    props: {
      // 声明 props
    }
  }
);
```

在将来，我们计划提供一个 Babel 插件，自动推断并注入运行时 props (就像在单文件组件中的 `defineProps` 一样)，以便省略运行时 props 的声明。

### 为什么

defineComponent返回值的类型是一个构造函数类型，它的实例类型是根据选项推断出的组件实例类型。

这是为了能让该返回值在使用 TypeScript 时提供类型推导支持。

### 怎么做

#### 泛型支持

在组合式 API 的用法中，`defineComponent()` 可以支持泛型：

```ts
const Comp = defineComponent(
  <T extends string | number>(props: { msg: T; list: T[] }) => {
    const count = ref(0);

    return () => <div>{count.value}</div>;
  },
  {
    props: ['msg', 'list']
  }
);
```

* **功能** ：通过泛型，可以在组件中使用更灵活的类型定义，确保类型安全。
* **未来展望** ：Vue 计划提供一个 Babel 插件，自动推断并注入运行时 props，从而在未来可以省略手动声明运行时 props。

#### webpack Treeshaking 的注意事项

因为 `defineComponent()` 是一个函数调用，所以它可能被某些构建工具认为会产生副作用，如 webpack。即使一个组件从未被使用，也有可能不被 tree-shake。
为了告诉 webpack 这个函数调用可以被安全地 tree-shake，我们可以在函数调用之前添加一个 `/*#__PURE__*/` 形式的注释：
**js**

```
export default /*#__PURE__*/ defineComponent(/* ... */)
```

请注意，如果你的项目中使用的是 Vite，就不需要这么做，因为 Rollup (Vite 底层使用的生产环境打包工具) 可以智能地确定 `defineComponent()` 实际上并没有副作用，所以无需手动注释。

## defineAsyncComponent()

定义一个异步组件，它在运行时是懒加载的。参数可以是一个异步加载函数，或是对加载行为进行更具体定制的一个选项对象。

* **类型**
  **ts**
  ```
  function defineAsyncComponent(
    source: AsyncComponentLoader | AsyncComponentOptions
  ): Component

  type AsyncComponentLoader = () => Promise<Component>

  interface AsyncComponentOptions {
    loader: AsyncComponentLoader
    loadingComponent?: Component
    errorComponent?: Component
    delay?: number
    timeout?: number
    suspensible?: boolean
    onError?: (
      error: Error,
      retry: () => void,
      fail: () => void,
      attempts: number
    ) => any
  }
  ```
* **参考**[指南 - 异步组件](https://cn.vuejs.org/guide/components/async.html)
