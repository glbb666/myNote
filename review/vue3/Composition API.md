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

<pre class="!overflow-visible"><div class="dark bg-gray-950 contain-inline-size rounded-md border-[0.5px] border-token-border-medium relative"><div class="flex items-center text-token-text-secondary bg-token-main-surface-secondary px-4 py-2 text-xs font-sans justify-between rounded-t-md h-9">js</div><div class="sticky top-9 md:top-[5.75rem]"><div class="absolute bottom-0 right-2 flex h-9 items-center"><div class="flex items-center rounded bg-token-main-surface-secondary px-2 font-sans text-xs text-token-text-secondary"><span class="" data-state="closed"><button class="flex gap-1 items-center py-1"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" class="icon-sm"><path fill-rule="evenodd" clip-rule="evenodd" d="M7 5C7 3.34315 8.34315 2 10 2H19C20.6569 2 22 3.34315 22 5V14C22 15.6569 20.6569 17 19 17H17V19C17 20.6569 15.6569 22 14 22H5C3.34315 22 2 20.6569 2 19V10C2 8.34315 3.34315 7 5 7H7V5ZM9 7H14C15.6569 7 17 8.34315 17 10V15H19C19.5523 15 20 14.5523 20 14V5C20 4.44772 19.5523 4 19 4H10C9.44772 4 9 4.44772 9 5V7ZM5 9C4.44772 9 4 9.44772 4 10V19C4 19.5523 4.44772 20 5 20H14C14.5523 20 15 19.5523 15 19V10C15 9.44772 14.5523 9 14 9H5Z" fill="currentColor"></path></svg>复制代码</button></span></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="!whitespace-pre hljs language-js"><script setup>
import { ref } from 'vue'

// 创建一个响应式数据
const count = ref(0)

// 定义方法来修改响应式数据
const increment = () => {
  count.value++
}
</script>

`code`

code

#### 逻辑复用

你可以将组件的逻辑封装成可复用的组合函数（composables）。例如，将计数逻辑抽取出来：

<pre class="!overflow-visible"><div class="dark bg-gray-950 contain-inline-size rounded-md border-[0.5px] border-token-border-medium relative"><div class="flex items-center text-token-text-secondary bg-token-main-surface-secondary px-4 py-2 text-xs font-sans justify-between rounded-t-md h-9">js</div><div class="sticky top-9 md:top-[5.75rem]"><div class="absolute bottom-0 right-2 flex h-9 items-center"><div class="flex items-center rounded bg-token-main-surface-secondary px-2 font-sans text-xs text-token-text-secondary"><span class="" data-state="closed"><button class="flex gap-1 items-center py-1"><svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" class="icon-sm"><path fill-rule="evenodd" clip-rule="evenodd" d="M7 5C7 3.34315 8.34315 2 10 2H19C20.6569 2 22 3.34315 22 5V14C22 15.6569 20.6569 17 19 17H17V19C17 20.6569 15.6569 22 14 22H5C3.34315 22 2 20.6569 2 19V10C2 8.34315 3.34315 7 5 7H7V5ZM9 7H14C15.6569 7 17 8.34315 17 10V15H19C19.5523 15 20 14.5523 20 14V5C20 4.44772 19.5523 4 19 4H10C9.44772 4 9 4.44772 9 5V7ZM5 9C4.44772 9 4 9.44772 4 10V19C4 19.5523 4.44772 20 5 20H14C14.5523 20 15 19.5523 15 19V10C15 9.44772 14.5523 9 14 9H5Z" fill="currentColor"></path></svg>复制代码</button></span></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="!whitespace-pre hljs language-js">// useCounter.js
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

`code`

#### `生命周期钩子`

`在组合式 API 中，生命周期钩子同样可以在 `

code

`code`

### `code`

`code`

* `code`
* `code`
* `code`

`code`
