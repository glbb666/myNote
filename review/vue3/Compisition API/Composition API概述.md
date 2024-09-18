# **是什么？**

Composition API是 Vue 3 中引入的一种新的编写组件逻辑的方式。

与选项式 API 中将逻辑按类别（如 `data`、`methods`、`computed`）分散定义不同，组合式 API通过导入和调用函数来定义组件逻辑，它允许你将相关的逻辑聚合在一起。

# **为什么它重要？**

* **逻辑聚合** ：在选项式 API 中，组件的逻辑往往分散在不同的选项中，这使得复杂的组件难以维护。而组合式 API 允许将相关的逻辑集中在一起，代码更加清晰。
* **逻辑复用** ：通过组合式 API，开发者可以封装逻辑为可复用的函数或模块，然后在多个组件中使用，提高了代码的复用性。
* **与 TypeScript 兼容更好** ：组合式 API 提供的函数式写法使得在 TypeScript 中使用 Vue 变得更直观和高效。
* **更灵活的状态管理** ：你可以在更灵活的地方定义响应式数据、方法和生命周期钩子。

# **如何使用？**

#### 基本用法

在组合式 API 中，最常用的两个函数是 `setup` 和 Vue 的响应式 API（如 `ref`、`reactive`）。`setup` 是组件的入口函数，所有逻辑都在这个函数内处理。

```html
<script setup>
import { ref } from 'vue'

// 创建一个响应式数据
const count = ref(0)

// 定义方法来修改响应式数据
const increment = () => {
  count.value++
}
</script>
```

#### 逻辑复用

你可以将组件的逻辑封装成可复用的组合函数（composables）。例如，将计数逻辑抽取出来：

```js
// useCounter.js
import { ref } from 'vue'

export function useCounter() {
  const count = ref(0)
  const increment = () => {
    count.value++
  }
  return { count, increment }
}

// 组件中使用
<script setup>
import { useCounter } from './useCounter'

const { count, increment } = useCounter()
</script>

```

#### 生命周期钩子

在组合式 API 中，生命周期钩子同样可以在 `setup` 中使用，并通过 Vue 提供的对应函数来定义，例如 `onMounted`、`onUnmounted` 等。

```js
<script setup>
import { onMounted } from 'vue'

onMounted(() => {
  console.log('组件挂载完成')
})
</script>

```
