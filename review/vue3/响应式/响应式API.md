# 是什么

响应式 API 是 Vue 3 引入的一组新的工具和方法，用于更灵活、更直观地声明和管理响应式状态。

# 为什么

1. **更灵活的状态管理** ：在 Vue 2 中，状态管理主要通过 `data` 选项来实现，而在 Vue 3 中，引入了响应式 API，使得状态管理更加灵活。
2. **逻辑复用** ：组合式 API 使得我们可以更容易地将逻辑封装成独立的函数或模块，方便复用和测试。
3. **更好的 TypeScript 支持** ：响应式 API 设计时考虑了 TypeScript 的支持，使得在 TypeScript 项目中使用 Vue 3 更加顺畅。

# 怎么做

## ref()

### 是什么

[`ref()`](https://cn.vuejs.org/api/reactivity-core.html#ref) 函数是用来声明响应式状态的。

### 为什么

你可能会好奇：为什么我们需要使用带有 `.value` 的 ref，而不是普通的变量？为了解释这一点，我们需要简单地讨论一下 Vue 的响应式系统是如何工作的。

当你在模板中使用了一个 ref，然后改变了这个 ref 的值时，Vue 会自动检测到这个变化，并且相应地更新 DOM。这是通过一个基于依赖追踪的响应式系统实现的。当一个组件首次渲染时，Vue 会**追踪**在渲染过程中使用的每一个 ref。然后，当一个 ref 被修改时，它会**触发**追踪它的组件的一次重新渲染。

在标准的 JavaScript 中，检测普通变量的访问或修改是行不通的。然而，我们可以通过 getter 和 setter 方法来拦截对象属性的 get 和 set 操作。

该 `.value` 属性给予了 Vue 一个机会来检测 ref 何时被访问或修改。在其内部，Vue 在它的 getter 中执行追踪，在它的 setter 中执行触发。从概念上讲，你可以将 ref 看作是一个像这样的对象：

```js
// 伪代码，不是真正的实现
const myRef = {
  _value: 0,
  get value() {
    track()
    return this._value
  },
  set value(newValue) {
    this._value = newValue
    trigger()
  }
}
```

另一个 ref 的好处是，与普通变量不同，你可以将 ref 传递给函数，同时保留对最新值和响应式连接的访问。当将复杂的逻辑重构为可重用的代码时，这将非常有用。

该响应性系统在深入响应式原理中有更详细的讨论。

### 怎么做

#### 基础使用

`ref()` 接收参数，返回一个响应式的、可更改的 ref 对象，此对象只有一个指向其内部值的属性 `.value`。

所有对 `.value` 的操作都将被追踪，并且写操作会触发与之相关的副作用。

```js
const count = ref(0)

console.log(count) // { value: 0 }
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

#### ts类型

ref 会根据初始化时的值推导其类型：

```ts
import { ref } from 'vue'

// 推导出的类型：Ref<number>
const year = ref(2020)

// => TS Error: Type 'string' is not assignable to type 'number'.
year.value = '2020'
```

有时我们可能想为 ref 内的值指定一个更复杂的类型，可以通过使用 `Ref` 这个类型：

```ts
import { ref } from 'vue'
import type { Ref } from 'vue'

const year: Ref<string | number> = ref('2020')

year.value = 2020 // 成功！
```

或者，在调用 `ref()` 时传入一个泛型参数，来覆盖默认的推导行为：

```ts
// 得到的类型：Ref<string | number>
const year = ref<string | number>('2020')

year.value = 2020 // 成功！
```

如果你指定了一个泛型参数但没有给出初始值，那么最后得到的就将是一个包含 `undefined` 的联合类型：

```ts
// 推导得到的类型：Ref<number | undefined>
const n = ref<number>()
```

#### 解包

##### 在 `{{ }}`中进行计算操作时，只有顶级的 ref 属性会被解包

在下面的例子中，`count` 和 `object` 是顶级属性，但 `object.id` 不是：

```js
const count = ref(0)
const object = { id: ref(1) }
```

因此，这个表达式按预期工作：

```js
{{ count + 1 }}
```

...但这个 **不会** ：

```js
{{ object.id + 1 }}
```

渲染的结果将是 `[object Object]1`，因为在计算表达式时 `object.id` 没有被解包，仍然是一个 ref 对象。为了解决这个问题，我们可以将 `id` 解构为一个顶级属性：

```js
const { id } = object
```

```js
{{ id + 1 }}
```

现在渲染的结果将是 `2`。

##### 在 `{{ }}`中进行最终展示操作时，所有的 ref 属性都会会被解包

```js
{{ object.id }} //渲染为1
```

该特性仅仅是文本插值的一个便利特性，等价于 `{{ object.id.value }}`。

#### 深层响应性

Ref 可以持有任何类型的值。

如果将一个对象赋值给 ref，那么这个对象将通过reactive()转为具有深层次响应式的对象。

Ref 会使它的值具有深层响应性。这意味着即使改变嵌套对象或数组时，变化也会被检测到：

```js
import { ref } from 'vue'

const obj = ref({
  nested: { count: 0 },
  arr: ['foo', 'bar']
})

function mutateDeeply() {
  // 以下都会按照期望工作
  obj.value.nested.count++
  obj.value.arr.push('baz')
}
```

非原始值将通过 [`reactive()`](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals#reactive) 转换为响应式代理，该函数将在后面讨论。

也可以通过 shallowRef 来放弃深层响应性。对于浅层 ref，只有 `.value` 的访问会被追踪。浅层 ref 可以用于避免对大型数据的响应性开销来优化性能、或者有外部库管理其内部状态的情况。

## computed()

### 是什么

`computed()` 函数用于创建计算属性。计算属性是基于其他响应式数据进行计算的属性，它们的值会根据依赖的数据自动更新。`computed()` 函数可以返回一个只读的计算属性，也可以返回一个可读写的计算属性。

### 为什么

模板中的表达式虽然方便，但也只能用来做简单的操作。如果在模板中写太多逻辑，会让模板变得臃肿，难以维护。

比如说，我们有这样一个包含嵌套数组的对象：

```js
const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})
```

我们想根据 `author` 是否已有一些书籍来展示不同的信息：

```template
<p>Has published books:</p>
<span>{{ author.books.length > 0 ? 'Yes' : 'No' }}</span>
```

这里的模板看起来有些复杂。我们必须认真看好一会儿才能明白它的计算依赖于 `author.books`。更重要的是，如果在模板中需要不止一次这样的计算，我们可不想将这样的代码在模板里重复好多遍。

因此我们推荐使用**计算属性**来描述依赖响应式状态的复杂逻辑。这是重构后的示例：

```html
<script setup>
import { reactive, computed } from 'vue'

const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})

// 一个计算属性 ref
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})
</script>

<template>
  <p>Has published books:</p>
  <span>{{ publishedBooksMessage }}</span>
</template>
```

我们在这里定义了一个计算属性 `publishedBooksMessage`。`computed()` 方法期望接收一个getter 函数，返回值为一个 **计算属性 ref** 。和其他一般的 ref 类似，你可以通过 `publishedBooksMessage.value` 访问计算结果。计算属性 ref 也会在模板中自动解包，因此在模板表达式中引用时无需添加 `.value`。

Vue 的计算属性会自动追踪响应式依赖。它会检测到 `publishedBooksMessage` 依赖于 `author.books`，所以当 `author.books` 改变时，任何依赖于 `publishedBooksMessage` 的绑定都会同时更新。

### 怎么做

#### 1. 只读计算属性

通过提供一个 getter 函数给 `computed()`，可以创建一个只读的计算属性。这个计算属性是一个响应式的 `ref` 对象。

```js
  const count = ref(1);
  const plusOne = computed(() => count.value + 1);

  console.log(plusOne.value); // 输出 2，因为 count 是 1，所以 plusOne 是 1 + 1 = 2

  plusOne.value++; // 错误：plusOne 是只读的，不能直接修改它的值
```

  在这个示例中，`plusOne` 是一个计算属性，它依赖于 `count` 的值。每当 `count` 的值变化时，`plusOne` 的值会自动更新。

#### 2. 可写计算属性

通过提供一个包含 `get` 和 `set` 方法的对象给 `computed()`，可以创建一个可写的计算属性。

```js
  const count = ref(1);
  const plusOne = computed({
    get: () => count.value + 1,
    set: (val) => {
      count.value = val - 1;
    }
  });

  plusOne.value = 1;
  console.log(count.value); // 输出 0，因为 plusOne 的值被设置为 1，所以 count 被设置为 1 - 1 = 0
```

  在这个示例中，`plusOne` 是一个可写的计算属性。通过 `set` 方法，可以间接地改变 `count` 的值。

#### 3. 调试计算属性

 `computed()` 还可以接收一个 `debuggerOptions` 对象，用于调试计算属性的依赖追踪和触发行为。

```js
  const plusOne = computed(() => count.value + 1, {
    onTrack(e) {
      // 当 count.value 被追踪为依赖时触发
      debugger; 
    },
    onTrigger(e) {
      // 当 count.value 被更改时触发
      debugger; 
    }
  });
```

计算属性的 `onTrack` 和 `onTrigger` 选项仅会在开发模式下工作。

#### 4. 为 `computed()` 标注类型

`computed()` 会自动从其计算函数的返回值上推导出类型：

```ts
import { ref, computed } from 'vue'

const count = ref(0)

// 推导得到的类型：ComputedRef<number>
const double = computed(() => count.value * 2)

// => TS Error: Property 'split' does not exist on type 'number'
const result = double.value.split('')
```

你还可以通过泛型参数显式指定类型：

```ts
const double = computed<number>(() => {
  // 若返回值不是 number 类型则会报错
})
```

#### 5. 计算属性稳定性

在 Vue 3.4 及更高版本中，计算属性仅在其计算值较前一个值发生更改时才会触发副作用。

例如，以下 `isEven` 计算属性仅在返回值从 `true` 更改为 `false` 时才会触发副作用，反之亦然：

```js
const count = ref(0)
const isEven = computed(() => count.value % 2 === 0)

watchEffect(() => console.log(isEven.value)) // true

// 这将不会触发新的输出，因为计算属性的值依然为 `true`
count.value = 2
count.value = 4
```

这减少了非必要副作用的触发。但不幸的是，如果计算属性在每次计算时都创建一个新对象，则不起作用：

```js
const computedObj = computed(() => {
  return {
    isEven: count.value % 2 === 0
  }
})
```

由于每次都会创建一个新对象，因此从技术上讲，新旧值始终不同。即使 `isEven` 属性保持不变，Vue 也无法知道，除非它对旧值和新值进行深度比较。这种比较可能代价高昂，并不值得。

相反，我们可以通过手动比较新旧值来优化。如果我们知道没有变化，则有条件地返回旧值：

```js
const computedObj = computed((oldValue) => {
  const newValue = {
    isEven: count.value % 2 === 0
  }
  if (oldValue && oldValue.isEven === newValue.isEven) {
    return oldValue
  }
  return newValue
})
```

值得注意的是，你应该始终在比较和返回旧值之前执行完整计算，以便在每次运行时都可以收集到相同的依赖项。

## reactive()

### 是什么

与将内部值包装在特殊对象中的 ref 不同，`reactive()` 将使对象本身具有响应性。

响应式转换是“深层”的：它会影响到所有嵌套的属性。

一个响应式对象也将深层地解包任何 [ref](https://cn.vuejs.org/api/reactivity-core#ref) 属性，同时保持响应性。

返回的对象以及其中嵌套的对象都会通过 [ES Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 包裹，因此**不等于**源对象，建议只使用响应式代理，避免使用原始对象。

### 怎么做

#### 代理

响应式对象是 [JavaScript 代理](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，其行为就和普通对象一样。不同的是，Vue 能够拦截对响应式对象所有属性的访问和修改，以便进行依赖追踪和触发更新。

```js
const raw = {}
const proxy = reactive(raw)

// 代理对象和原始对象不是全等的
console.log(proxy === raw) // false
```

只有代理对象是响应式的，更改原始对象不会触发更新。因此，使用 Vue 的响应式系统的最佳实践是 **仅使用你声明对象的代理版本** 。

为保证访问代理的一致性，对同一个原始对象调用 `reactive()` 会总是返回同样的代理对象，而对一个已存在的代理对象调用 `reactive()` 会返回其本身。

这个规则对嵌套对象也适用。依靠深层响应性，响应式对象内的嵌套对象依然是代理

```js
const proxy = reactive({})

const raw = {}
proxy.nested = raw

console.log(proxy.nested === raw) // false
```

#### 解包

##### 作为 reactive 对象的属性

一个 ref 会在作为响应式对象的属性被访问或修改时自动解包。换句话说，它的行为就像一个普通的属性：

```js
const count = ref(0)
const state = reactive({
  count
})

console.log(state.count) // 0

state.count = 1
console.log(count.value) // 1
```

如果将一个新的 ref 赋值给一个关联了已有 ref 的属性，那么它会替换掉旧的 ref：

```js
const otherCount = ref(2)

state.count = otherCount
console.log(state.count) // 2
// 原始 ref 现在已经和 state.count 失去联系
console.log(count.value) // 1
```

只有当嵌套在一个深层响应式对象内时，才会发生 ref 解包。

若要避免深层响应式转换，只想保留对这个对象顶层次访问的响应性，请使用 shallowReactive 作替代。

##### 数组和集合的注意事项

与 reactive 对象不同的是，当 ref 作为响应式数组或原生集合类型 (如 `Map`) 中的元素被访问时，它**不会**被解包：

```js
const books = reactive([ref('Vue 3 Guide')])
// 这里需要 .value
console.log(books[0].value)

const map = reactive(new Map([['count', ref(0)]]))
// 这里需要 .value
console.log(map.get('count').value)
```

### 局限性

1. **有限的值类型** ：它只能用于对象类型 (对象、数组和如 `Map`、`Set` 这样的[集合类型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects#keyed_collections))。它不能持有如 `string`、`number` 或 `boolean` 这样的[原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)。
2. **不能替换整个对象** ：由于 Vue 的响应式跟踪是通过属性访问实现的，因此我们必须始终保持对响应式对象的相同引用。这意味着我们不能轻易地“替换”响应式对象，因为这样的话与第一个引用的响应性连接将丢失：

```js
   let state = reactive({ count: 0 })

   // 上面的 ({ count: 0 }) 引用将不再被追踪
   // (响应性连接已丢失！)
   state = reactive({ count: 1 })
```

3. **对解构操作不友好** ：当我们将响应式对象的原始类型属性解构为本地变量时，或者将该属性传递给函数时，我们将丢失响应性连接：

```js
   const state = reactive({ count: 0 })

   // 当解构时，count 已经与 state.count 断开连接
   let { count } = state
   // 不会影响原始的 state
   count++

   // 该函数接收到的是一个普通的数字
   // 并且无法追踪 state.count 的变化
   // 我们必须传入整个对象以保持响应性
   callSomeFunction(state.count)
```

由于这些限制，我们建议使用 `ref()` 作为声明响应式状态的主要 API。

## readonly()

接受一个对象 (不论是响应式还是普通的) 或是一个 [ref](https://cn.vuejs.org/api/reactivity-core#ref)，返回一个原值的只读代理。

* **类型**
  **ts**
  ```
  function readonly<T extends object>(
    target: T
  ): DeepReadonly<UnwrapNestedRefs<T>>
  ```
* **详细信息**
  只读代理是深层的：对任何嵌套属性的访问都将是只读的。它的 ref 解包行为与 `reactive()` 相同，但解包得到的值是只读的。
  要避免深层级的转换行为，请使用 [shallowReadonly()](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreadonly) 作替代。
* **示例**
  **js**
  ```
  const original = reactive({ count: 0 })

  const copy = readonly(original)

  watchEffect(() => {
    // 用来做响应性追踪
    console.log(copy.count)
  })

  // 更改源属性会触发其依赖的侦听器
  original.count++

  // 更改该只读副本将会失败，并会得到一个警告
  copy.count++ // warning!
  ```

## watchEffect()

立即运行一个函数，同时响应式地追踪其依赖，并在依赖更改时重新执行。

* **类型**
  **ts**

  ```
  function watchEffect(
    effect: (onCleanup: OnCleanup) => void,
    options?: WatchEffectOptions
  ): WatchHandle

  type OnCleanup = (cleanupFn: () => void) => void

  interface WatchEffectOptions {
    flush?: 'pre' | 'post' | 'sync' // 默认：'pre'
    onTrack?: (event: DebuggerEvent) => void
    onTrigger?: (event: DebuggerEvent) => void
  }

  interface WatchHandle {
    (): void // 可调用，与 `stop` 相同
    pause: () => void
    resume: () => void
    stop: () => void
  }
  ```
* **详细信息**
  第一个参数就是要运行的副作用函数。这个副作用函数的参数也是一个函数，用来注册清理回调。清理回调会在该副作用下一次执行前被调用，可以用来清理无效的副作用，例如等待中的异步请求 (参见下面的示例)。
  第二个参数是一个可选的选项，可以用来调整副作用的刷新时机或调试副作用的依赖。
  默认情况下，侦听器将在组件渲染之前执行。设置 `flush: 'post'` 将会使侦听器延迟到组件渲染之后再执行。详见[回调的触发时机](https://cn.vuejs.org/guide/essentials/watchers.html#callback-flush-timing)。在某些特殊情况下 (例如要使缓存失效)，可能有必要在响应式依赖发生改变时立即触发侦听器。这可以通过设置 `flush: 'sync'` 来实现。然而，该设置应谨慎使用，因为如果有多个属性同时更新，这将导致一些性能和数据一致性的问题。
  返回值是一个用来停止该副作用的函数。
* **示例**
  **js**

  ```
  const count = ref(0)

  watchEffect(() => console.log(count.value))
  // -> 输出 0

  count.value++
  // -> 输出 1
  ```

  副作用清除：
  **js**

  ```
  watchEffect(async (onCleanup) => {
    const { response, cancel } = doAsyncWork(id.value)
    // `cancel` 会在 `id` 更改时调用
    // 以便取消之前
    // 未完成的请求
    onCleanup(cancel)
    data.value = await response
  })
  ```

  停止侦听器：
  **js**

  ```
  const stop = watchEffect(() => {})

  // 当不再需要此侦听器时:
  stop()
  ```

  暂停/恢复侦听器：^^
  **js**

  ```
  const { stop, pause, resume } = watchEffect(() => {})

  // 暂停侦听器
  pause()

  // 稍后恢复
  resume()

  // 停止
  stop()
  ```

  副作用清理：
  **js**

  ```
  watchEffect(async (onCleanup) => {
    const { response, cancel } = doAsyncWork(newId)
    // 如果 `id` 变化，则调用 `cancel`，
    // 如果之前的请求未完成，则取消该请求
    onCleanup(cancel)
    data.value = await response
  })
  ```

  3.5+ 中的副作用清理：
  **js**

  ```
  import { onWatcherCleanup } from 'vue'

  watchEffect(async () => {
    const { response, cancel } = doAsyncWork(newId)
    // 如果 `id` 变化，则调用 `cancel`，
    // 如果之前的请求未完成，则取消该请求
    onWatcherCleanup(cancel)
    data.value = await response
  })
  ```

  选项：
  **js**

  ```
  watchEffect(() => {}, {
    flush: 'post',
    onTrack(e) {
      debugger
    },
    onTrigger(e) {
      debugger
    }
  })
  ```
* **参考**

  * [指南 - 侦听器](https://cn.vuejs.org/guide/essentials/watchers.html#watcheffect)
  * [指南 - 侦听器调试](https://cn.vuejs.org/guide/extras/reactivity-in-depth.html#watcher-debugging)

## watchPostEffect()

[`watchEffect()`](https://cn.vuejs.org/api/reactivity-core#watcheffect) 使用 `flush: 'post'` 选项时的别名。

## watchSyncEffect()

[`watchEffect()`](https://cn.vuejs.org/api/reactivity-core#watcheffect) 使用 `flush: 'sync'` 选项时的别名。

## watch()

侦听一个或多个响应式数据源，并在数据源变化时调用所给的回调函数。

* **类型**
  **ts**

  ```
  // 侦听单个来源
  function watch<T>(
    source: WatchSource<T>,
    callback: WatchCallback<T>,
    options?: WatchOptions
  ): WatchHandle

  // 侦听多个来源
  function watch<T>(
    sources: WatchSource<T>[],
    callback: WatchCallback<T[]>,
    options?: WatchOptions
  ): WatchHandle

  type WatchCallback<T> = (
    value: T,
    oldValue: T,
    onCleanup: (cleanupFn: () => void) => void
  ) => void

  type WatchSource<T> =
    | Ref<T> // ref
    | (() => T) // getter
    | T extends object
    ? T
    : never // 响应式对象

  interface WatchOptions extends WatchEffectOptions {
    immediate?: boolean // 默认：false
    deep?: boolean | number // 默认：false
    flush?: 'pre' | 'post' | 'sync' // 默认：'pre'
    onTrack?: (event: DebuggerEvent) => void
    onTrigger?: (event: DebuggerEvent) => void
    once?: boolean // 默认：false (3.4+)
  }

  interface WatchHandle {
    (): void // 可调用，与 `stop` 相同
    pause: () => void
    resume: () => void
    stop: () => void
  }
  ```

  > 为了便于阅读，对类型进行了简化。
  >
* **详细信息**
  `watch()` 默认是懒侦听的，即仅在侦听源发生变化时才执行回调函数。
  第一个参数是侦听器的 **源** 。这个来源可以是以下几种：

  * 一个函数，返回一个值
  * 一个 ref
  * 一个响应式对象
  * ...或是由以上类型的值组成的数组

  第二个参数是在发生变化时要调用的回调函数。这个回调函数接受三个参数：新值、旧值，以及一个用于注册副作用清理的回调函数。该回调函数会在副作用下一次重新执行前调用，可以用来清除无效的副作用，例如等待中的异步请求。
  当侦听多个来源时，回调函数接受两个数组，分别对应来源数组中的新值和旧值。
  第三个可选的参数是一个对象，支持以下这些选项：

  * **`immediate`** ：在侦听器创建时立即触发回调。第一次调用时旧值是 `undefined`。
  * **`deep`** ：如果源是对象，强制深度遍历，以便在深层级变更时触发回调。在 3.5+ 中，此参数还可以是指示最大遍历深度的数字。参考[深层侦听器](https://cn.vuejs.org/guide/essentials/watchers.html#deep-watchers)。
  * **`flush`** ：调整回调函数的刷新时机。参考[回调的刷新时机](https://cn.vuejs.org/guide/essentials/watchers.html#callback-flush-timing)及 [`watchEffect()`](https://cn.vuejs.org/api/reactivity-core.html#watcheffect)。
  * **`onTrack / onTrigger`** ：调试侦听器的依赖。参考[调试侦听器](https://cn.vuejs.org/guide/extras/reactivity-in-depth.html#watcher-debugging)。
  * **`once`** ：(3.4+) 回调函数只会运行一次。侦听器将在回调函数首次运行后自动停止。

  与 [`watchEffect()`](https://cn.vuejs.org/api/reactivity-core#watcheffect) 相比，`watch()` 使我们可以：

  * 懒执行副作用；
  * 更加明确是应该由哪个状态触发侦听器重新执行；
  * 可以访问所侦听状态的前一个值和当前值。
* **示例**
  侦听一个 getter 函数：
  **js**

  ```
  const state = reactive({ count: 0 })
  watch(
    () => state.count,
    (count, prevCount) => {
      /* ... */
    }
  )
  ```

  侦听一个 ref：
  **js**

  ```
  const count = ref(0)
  watch(count, (count, prevCount) => {
    /* ... */
  })
  ```

  当侦听多个来源时，回调函数接受两个数组，分别对应来源数组中的新值和旧值：
  **js**

  ```
  watch([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
    /* ... */
  })
  ```

  当使用 getter 函数作为源时，回调只在此函数的返回值变化时才会触发。如果你想让回调在深层级变更时也能触发，你需要使用 `{ deep: true }` 强制侦听器进入深层级模式。在深层级模式时，如果回调函数由于深层级的变更而被触发，那么新值和旧值将是同一个对象。
  **js**

  ```
  const state = reactive({ count: 0 })
  watch(
    () => state,
    (newValue, oldValue) => {
      // newValue === oldValue
    },
    { deep: true }
  )
  ```

  当直接侦听一个响应式对象时，侦听器会自动启用深层模式：
  **js**

  ```
  const state = reactive({ count: 0 })
  watch(state, () => {
    /* 深层级变更状态所触发的回调 */
  })
  ```

  `watch()` 和 [`watchEffect()`](https://cn.vuejs.org/api/reactivity-core#watcheffect) 享有相同的刷新时机和调试选项：
  **js**

  ```
  watch(source, callback, {
    flush: 'post',
    onTrack(e) {
      debugger
    },
    onTrigger(e) {
      debugger
    }
  })
  ```

  停止侦听器：
  **js**

  ```
  const stop = watch(source, callback)

  // 当已不再需要该侦听器时：
  stop()
  ```

  暂停/恢复侦听器：^^
  **js**

  ```
  const { stop, pause, resume } = watchEffect(() => {})

  // 暂停侦听器
  pause()

  // 稍后恢复
  resume()

  // 停止
  stop()
  ```

  副作用清理：
  **js**

  ```
  watch(id, async (newId, oldId, onCleanup) => {
    const { response, cancel } = doAsyncWork(newId)
    // 当 `id` 变化时，`cancel` 将被调用，
    // 取消之前的未完成的请求
    onCleanup(cancel)
    data.value = await response
  })
  ```

  3.5+ 中的副作用清理：
  **js**

  ```
  import { onWatcherCleanup } from 'vue'

  watch(id, async (newId) => {
    const { response, cancel } = doAsyncWork(newId)
    onWatcherCleanup(cancel)
    data.value = await response
  })
  ```
* **参考**

  * [指南 - 侦听器](https://cn.vuejs.org/guide/essentials/watchers.html)
  * [指南 - 侦听器调试](https://cn.vuejs.org/guide/extras/reactivity-in-depth.html#watcher-debugging)

## onWatcherCleanup()

注册一个清理函数，在当前侦听器即将重新运行时执行。只能在 `watchEffect` 作用函数或 `watch` 回调函数的同步执行期间调用 (即不能在异步函数的 `await` 语句之后调用)。

* **类型**
  **ts**

  ```
  function onWatcherCleanup(
    cleanupFn: () => void,
    failSilently?: boolean
  ): void
  ```
* **示例**
  **ts**

  ```
  import { watch, onWatcherCleanup } from 'vue'

  watch(id, (newId) => {
    const { response, cancel } = doAsyncWork(newId)
    // 如果 `id` 变化，则调用 `cancel`，
    // 如果之前的请求未完成，则取消该请求
    onWatcherCleanup(cancel)
  })
  ```

# 工具函数

## isRef()

检查某个值是否为 ref。

* **类型**
  **ts**

  ```
  function isRef<T>(r: Ref<T> | unknown): r is Ref<T>
  ```

  请注意，返回值是一个[类型判定](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates) (type predicate)，这意味着 `isRef` 可以被用作类型守卫：
  **ts**

  ```
  let foo: unknown
  if (isRef(foo)) {
    // foo 的类型被收窄为了 Ref<unknown>
    foo.value
  }
  ```

## unref()

如果参数是 ref，则返回内部值，否则返回参数本身。这是 `val = isRef(val) ? val.value : val` 计算的一个语法糖。

* **类型**
  **ts**
  ```
  function unref<T>(ref: T | Ref<T>): T
  ```
* **示例**
  **ts**
  ```
  function useFoo(x: number | Ref<number>) {
    const unwrapped = unref(x)
    // unwrapped 现在保证为 number 类型
  }
  ```

## toRef()

可以将值、refs 或 getters 规范化为 refs (3.3+)。

也可以基于响应式对象上的一个属性，创建一个对应的 ref。这样创建的 ref 与其源属性保持同步：改变源属性的值将更新 ref 的值，反之亦然。

* **类型**
  **ts**

  ```
  // 规范化签名 (3.3+)
  function toRef<T>(
    value: T
  ): T extends () => infer R
    ? Readonly<Ref<R>>
    : T extends Ref
    ? T
    : Ref<UnwrapRef<T>>

  // 对象属性签名
  function toRef<T extends object, K extends keyof T>(
    object: T,
    key: K,
    defaultValue?: T[K]
  ): ToRef<T[K]>

  type ToRef<T> = T extends Ref ? T : Ref<T>
  ```
* **示例**
  规范化签名 (3.3+)：
  **js**

  ```
  // 按原样返回现有的 ref
  toRef(existingRef)

  // 创建一个只读的 ref，当访问 .value 时会调用此 getter 函数
  toRef(() => props.foo)

  // 从非函数的值中创建普通的 ref
  // 等同于 ref(1)
  toRef(1)
  ```

  对象属性签名：
  **js**

  ```
  const state = reactive({
    foo: 1,
    bar: 2
  })

  // 双向 ref，会与源属性同步
  const fooRef = toRef(state, 'foo')

  // 更改该 ref 会更新源属性
  fooRef.value++
  console.log(state.foo) // 2

  // 更改源属性也会更新该 ref
  state.foo++
  console.log(fooRef.value) // 3
  ```

  请注意，这不同于：
  **js**

  ```
  const fooRef = ref(state.foo)
  ```

  上面这个 ref **不会**和 `state.foo` 保持同步，因为这个 `ref()` 接收到的是一个纯数值。
  `toRef()` 这个函数在你想把一个 prop 的 ref 传递给一个组合式函数时会很有用：
  **vue**

  ```
  <script setup>
  import { toRef } from 'vue'

  const props = defineProps(/* ... */)

  // 将 `props.foo` 转换为 ref，然后传入
  // 一个组合式函数
  useSomeFeature(toRef(props, 'foo'))

  // getter 语法——推荐在 3.3+ 版本使用
  useSomeFeature(toRef(() => props.foo))
  </script>
  ```

  当 `toRef` 与组件 props 结合使用时，关于禁止对 props 做出更改的限制依然有效。尝试将新的值传递给 ref 等效于尝试直接更改 props，这是不允许的。在这种场景下，你可能可以考虑使用带有 `get` 和 `set` 的 [`computed`](https://cn.vuejs.org/api/reactivity-core.html#computed) 替代。详情请见[在组件上使用 `v-model`](https://cn.vuejs.org/guide/components/v-model.html) 指南。
  当使用对象属性签名时，即使源属性当前不存在，`toRef()` 也会返回一个可用的 ref。这让它在处理可选 props 的时候格外实用，相比之下 [`toRefs`](https://cn.vuejs.org/api/reactivity-utilities.html#torefs) 就不会为可选 props 创建对应的 refs。

## toValue()

* 仅在 3.3+ 中支持

将值、refs 或 getters 规范化为值。这与 [unref()](https://cn.vuejs.org/api/reactivity-utilities.html#unref) 类似，不同的是此函数也会规范化 getter 函数。如果参数是一个 getter，它将会被调用并且返回它的返回值。

这可以在[组合式函数](https://cn.vuejs.org/guide/reusability/composables.html)中使用，用来规范化一个可以是值、ref 或 getter 的参数。

* **类型**
  **ts**

  ```
  function toValue<T>(source: T | Ref<T> | (() => T)): T
  ```
* **示例**
  **js**

  ```
  toValue(1) //       --> 1
  toValue(ref(1)) //  --> 1
  toValue(() => 1) // --> 1
  ```

  在组合式函数中规范化参数：
  **ts**

  ```
  import type { MaybeRefOrGetter } from 'vue'

  function useFeature(id: MaybeRefOrGetter<number>) {
    watch(() => toValue(id), id => {
      // 处理 id 变更
    })
  }

  // 这个组合式函数支持以下的任意形式：
  useFeature(1)
  useFeature(ref(1))
  useFeature(() => 1)
  ```

## toRefs()

将一个响应式对象转换为一个普通对象，这个普通对象的每个属性都是指向源对象相应属性的 ref。每个单独的 ref 都是使用 [`toRef()`](https://cn.vuejs.org/api/reactivity-utilities.html#toref) 创建的。

* **类型**
  **ts**

  ```
  function toRefs<T extends object>(
    object: T
  ): {
    [K in keyof T]: ToRef<T[K]>
  }

  type ToRef = T extends Ref ? T : Ref<T>
  ```
* **示例**
  **js**

  ```
  const state = reactive({
    foo: 1,
    bar: 2
  })

  const stateAsRefs = toRefs(state)
  /*
  stateAsRefs 的类型：{
    foo: Ref<number>,
    bar: Ref<number>
  }
  */

  // 这个 ref 和源属性已经“链接上了”
  state.foo++
  console.log(stateAsRefs.foo.value) // 2

  stateAsRefs.foo.value++
  console.log(state.foo) // 3
  ```

  当从组合式函数中返回响应式对象时，`toRefs` 相当有用。使用它，消费者组件可以解构/展开返回的对象而不会失去响应性：
  **js**

  ```
  function useFeatureX() {
    const state = reactive({
      foo: 1,
      bar: 2
    })

    // ...基于状态的操作逻辑

    // 在返回时都转为 ref
    return toRefs(state)
  }

  // 可以解构而不会失去响应性
  const { foo, bar } = useFeatureX()
  ```

  `toRefs` 在调用时只会为源对象上可以枚举的属性创建 ref。如果要为可能还不存在的属性创建 ref，请改用 [`toRef`](https://cn.vuejs.org/api/reactivity-utilities.html#toref)。

## isProxy()

检查一个对象是否是由 [`reactive()`](https://cn.vuejs.org/api/reactivity-core.html#reactive)、[`readonly()`](https://cn.vuejs.org/api/reactivity-core.html#readonly)、[`shallowReactive()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreactive) 或 [`shallowReadonly()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreadonly) 创建的代理。

* **类型**
  **ts**
  ```
  function isProxy(value: any): boolean
  ```

## isReactive()

检查一个对象是否是由 [`reactive()`](https://cn.vuejs.org/api/reactivity-core.html#reactive) 或 [`shallowReactive()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreactive) 创建的代理。

* **类型**
  **ts**
  ```
  function isReactive(value: unknown): boolean
  ```

## isReadonly()

检查传入的值是否为只读对象。只读对象的属性可以更改，但他们不能通过传入的对象直接赋值。

通过 [`readonly()`](https://cn.vuejs.org/api/reactivity-core.html#readonly) 和 [`shallowReadonly()`](https://cn.vuejs.org/api/reactivity-advanced.html#shallowreadonly) 创建的代理都是只读的，因为他们是没有 `set` 函数的 [`computed()`](https://cn.vuejs.org/api/reactivity-core.html#computed) ref。

* **类型**
  **ts**
  ```
  function isReadonly(value: unknown): boolean
  ```

# 进阶API

## shallowRef()

### 是什么

`shallowRef` 只会对其持有的值，即 `.value` 进行浅层的响应式处理。如果这个值本身是一个对象，改变这个对象的属性不会触发响应式更新。

```ts
function shallowRef<T>(value: T): ShallowRef<T>

interface ShallowRef<T> {
  value: T
}
```

### 为什么

`shallowRef()` 常常用于对大型数据结构的性能优化或是与外部的状态管理系统集成。

### 怎么做

```js
const state = shallowRef({ count: 1 })

// 不会触发更改
state.value.count = 2

// 会触发更改
state.value = { count: 2 }
```

## triggerRef()

强制触发依赖于一个[浅层 ref](https://cn.vuejs.org/api/reactivity-advanced#shallowref) 的副作用，这通常在对浅引用的内部值进行深度变更后使用。

* **类型**
  **ts**
  ```
  function triggerRef(ref: ShallowRef): void
  ```
* **示例**
  **js**
  ```
  const shallow = shallowRef({
    greet: 'Hello, world'
  })

  // 触发该副作用第一次应该会打印 "Hello, world"
  watchEffect(() => {
    console.log(shallow.value.greet)
  })

  // 这次变更不应触发副作用，因为这个 ref 是浅层的
  shallow.value.greet = 'Hello, universe'

  // 打印 "Hello, universe"
  triggerRef(shallow)
  ```

## customRef()

创建一个自定义的 ref，显式声明对其依赖追踪和更新触发的控制方式。

* **类型**
  **ts**

  ```
  function customRef<T>(factory: CustomRefFactory<T>): Ref<T>

  type CustomRefFactory<T> = (
    track: () => void,
    trigger: () => void
  ) => {
    get: () => T
    set: (value: T) => void
  }
  ```
* **详细信息**
  `customRef()` 预期接收一个工厂函数作为参数，这个工厂函数接受 `track` 和 `trigger` 两个函数作为参数，并返回一个带有 `get` 和 `set` 方法的对象。
  一般来说，`track()` 应该在 `get()` 方法中调用，而 `trigger()` 应该在 `set()` 中调用。然而事实上，你对何时调用、是否应该调用他们有完全的控制权。
* **示例**
  创建一个防抖 ref，即只在最近一次 set 调用后的一段固定间隔后再调用：
  **js**

  ```
  import { customRef } from 'vue'

  export function useDebouncedRef(value, delay = 200) {
    let timeout
    return customRef((track, trigger) => {
      return {
        get() {
          track()
          return value
        },
        set(newValue) {
          clearTimeout(timeout)
          timeout = setTimeout(() => {
            value = newValue
            trigger()
          }, delay)
        }
      }
    })
  }
  ```

  在组件中使用：
  **vue**

  ```
  <script setup>
  import { useDebouncedRef } from './debouncedRef'
  const text = useDebouncedRef('hello')
  </script>

  <template>
    <input v-model="text" />
  </template>
  ```

  [在演练场中尝试一下](https://play.vuejs.org/#eNplUkFugzAQ/MqKC1SiIekxIpEq9QVV1BMXCguhBdsyaxqE/PcuGAhNfYGd3Z0ZDwzeq1K7zqB39OI205UiaJGMOieiapTUBAOYFt/wUxqRYf6OBVgotGzA30X5Bt59tX4iMilaAsIbwelxMfCvWNfSD+Gw3++fEhFHTpLFuCBsVJ0ScgUQjw6Az+VatY5PiroHo3IeaeHANlkrh7Qg1NBL43cILUmlMAfqVSXK40QUOSYmHAZHZO0KVkIZgu65kTnWp8Qb+4kHEXfjaDXkhd7DTTmuNZ7MsGyzDYbz5CgSgbdppOBFqqT4l0eX1gZDYOm057heOBQYRl81coZVg9LQWGr+IlrchYKAdJp9h0C6KkvUT3A6u8V1dq4ASqRgZnVnWg04/QWYNyYzC2rD5Y3/hkDgz8fY/cOT1ZjqizMZzGY3rDPC12KGZYyd3J26M8ny1KKx7c3X25q1c1wrZN3L9LCMWs/+AmeG6xI=)
  谨慎使用

  当使用 customRef 时，我们应该谨慎对待其 getter 的返回值，尤其是在每次运行 getter 时都生成新对象数据类型的情况下。当这样的 customRef 作为 prop 传递时，将影响父组件和子组件之间的关系。

  父组件的渲染函数可能会被其他的响应式状态变化触发。在重新渲染过程中，我们会重新评估 customRef 的值，并返回一个新的对象数据类型作为子组件的 prop。这个 prop 会与其上一个值进行比较，由于两者不同，子组件中 customRef 的响应式依赖将被触发。与此同时，因为没有调用 customRef 的 setter，父组件中的响应式依赖不会运行。

  [在演练场中尝试一下](https://play.vuejs.org/#eNqFVEtP3DAQ/itTS9Vm1ZCt1J6WBZUiDvTQIsoNcwiOkzU4tmU7+9Aq/71jO1mCWuhlN/PyfPP45kAujCk2HSdLsnLMCuPBcd+Zc6pEa7T1cADWOa/bW17nYMPPtvRsDT3UVrcww+DZ0flStybpKSkWQQqPU0IVVUwr58FYvdvDWXgpu6ek1pqSHL0fS0vJw/z0xbN1jUPHY/Ys87Zkzzl4K5qG2zmcnUN2oAqg4T6bQ/wENKNXNk+CxWKsSlmLTSk7XlhedYxnWclYDiK+MkQCoK4wnVtnIiBJuuEJNA2qPof7hzkEoc8DXgg9yzYTBBFgNr4xyY4FbaK2p6qfI0iqFgtgulOe27HyQRy69Dk1JXY9C03JIeQ6wg4xWvJCqFpnlNytOcyC2wzYulQNr0Ao+Mhw0KnTTEttl/CIaIJiMz8NGBHFtYetVrPwa58/IL48Zag4N0ssquNYLYBoW16J0vOkC3VQtVqk7cG9QcHz1kj0QAlgVYkNMFk6d0bJ1pbGYKUkmtD42HmvFfi94WhOEiXwjUnBnlEz9OLTJwy5qCo44D4O7en71SIFjI/F9VuG4jEy/GHQKq5hQrJAKOc4uNVighBF5/cygS0GgOMoK+HQb7+EWvLdMM7weVIJy5kXWi0Rj+xaNRhLKRp1IvB9hxYegA6WJ1xkUe9PcF4e9a+suA3YwYiC5MQ79KlFUzw5rZCZEUtoRWuE5PaXCXmxtuWIkpJSSr39EXXHQcWYNWfP/9A/uV3QUXJjueN2E1ZhtPnSIqGS+er3T77D76Ox1VUn0fsd4y3HfewCxuT2vVMVwp74RbTX8WQI1dy5qx12xI1Fpa1K5AreeEHCCN8q/QXul+LrSC3s4nh93jltkVPDIYt5KJkcIKStCReo4rVQ/CZI6dyEzToCCJu7hAtry/1QH/qXncQB400KJwqPxZHxEyona0xS/E3rt1m9Ld1rZl+uhaxecRtP3EjtgddCyimtXyj9H/Ii3eId7uOGTkyk/wOEbQ9h)

## shallowReactive()

### 是什么

`shallowReactive` 只会对对象的顶层属性进行响应式处理，不会递归处理嵌套的对象属性。

这也意味着值为 ref 的属性**不会**被自动解包了。

```ts
function shallowReactive<T extends object>(target: T): T
```

### 怎么做

```js
const state = shallowReactive({
  foo: 1,
  nested: {
    bar: 2
  }
})

// 更改状态自身的属性是响应式的
state.foo++

// ...但下层嵌套对象不会被转为响应式
isReactive(state.nested) // false

// 不是响应式的
state.nested.bar++
```

## shallowReadonly()

[`readonly()`](https://cn.vuejs.org/api/reactivity-core.html#readonly) 的浅层作用形式

* **类型**
  **ts**

  ```
  function shallowReadonly<T extends object>(target: T): Readonly<T>
  ```
* **详细信息**
  和 `readonly()` 不同，这里没有深层级的转换：只有根层级的属性变为了只读。属性的值都会被原样存储和暴露，这也意味着值为 ref 的属性**不会**被自动解包了。
  谨慎使用

  浅层数据结构应该只用于组件中的根级状态。请避免将其嵌套在深层次的响应式对象中，因为它创建的树具有不一致的响应行为，这可能很难理解和调试。
* **示例**
  **js**

  ```
  const state = shallowReadonly({
    foo: 1,
    nested: {
      bar: 2
    }
  })

  // 更改状态自身的属性会失败
  state.foo++

  // ...但可以更改下层嵌套对象
  isReadonly(state.nested) // false

  // 这是可以通过的
  state.nested.bar++
  ```

## toRaw()

根据一个 Vue 创建的代理返回其原始对象。

* **类型**
  **ts**
  ```
  function toRaw<T>(proxy: T): T
  ```
* **详细信息**
  `toRaw()` 可以返回由 [`reactive()`](https://cn.vuejs.org/api/reactivity-core.html#reactive)、[`readonly()`](https://cn.vuejs.org/api/reactivity-core.html#readonly)、[`shallowReactive()`](https://cn.vuejs.org/api/reactivity-advanced#shallowreactive) 或者 [`shallowReadonly()`](https://cn.vuejs.org/api/reactivity-advanced#shallowreadonly) 创建的代理对应的原始对象。
  这是一个可以用于临时读取而不引起代理访问/跟踪开销，或是写入而不触发更改的特殊方法。不建议保存对原始对象的持久引用，请谨慎使用。
* **示例**
  **js**
  ```
  const foo = {}
  const reactiveFoo = reactive(foo)

  console.log(toRaw(reactiveFoo) === foo) // true
  ```

## markRaw()

将一个对象标记为不可被转为代理。返回该对象本身。

* **类型**
  **ts**

  ```
  function markRaw<T extends object>(value: T): T
  ```
* **示例**
  **js**

  ```
  const foo = markRaw({})
  console.log(isReactive(reactive(foo))) // false

  // 也适用于嵌套在其他响应性对象
  const bar = reactive({ foo })
  console.log(isReactive(bar.foo)) // false
  ```

  谨慎使用

  `markRaw()` 和类似 `shallowReactive()` 这样的浅层式 API 使你可以有选择地避开默认的深度响应/只读转换，并在状态关系谱中嵌入原始的、非代理的对象。它们可能出于各种各样的原因被使用：

  * 有些值不应该是响应式的，例如复杂的第三方类实例或 Vue 组件对象。
  * 当呈现带有不可变数据源的大型列表时，跳过代理转换可以提高性能。

  这应该是一种进阶需求，因为只在根层能访问到原始值，所以如果把一个嵌套的、没有标记的原始对象设置成一个响应式对象，然后再次访问它，你获取到的是代理的版本。这可能会导致 **对象身份风险** ，即执行一个依赖于对象身份的操作，但却同时使用了同一对象的原始版本和代理版本：

  **js**

  ```
  const foo = markRaw({
    nested: {}
  })

  const bar = reactive({
    // 尽管 `foo` 被标记为了原始对象，但 foo.nested 却没有
    nested: foo.nested
  })

  console.log(foo.nested === bar.nested) // false
  ```

  识别风险一般是很罕见的。然而，要正确使用这些 API，同时安全地避免这样的风险，需要你对响应性系统的工作方式有充分的了解。

## effectScope()

创建一个 effect 作用域，可以捕获其中所创建的响应式副作用 (即计算属性和侦听器)，这样捕获到的副作用可以一起处理。对于该 API 的使用细节，请查阅对应的 [RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0041-reactivity-effect-scope.md)。

* **类型**
  **ts**
  ```
  function effectScope(detached?: boolean): EffectScope

  interface EffectScope {
    run<T>(fn: () => T): T | undefined // 如果作用域不活跃就为 undefined
    stop(): void
  }
  ```
* **示例**
  **js**
  ```
  const scope = effectScope()

  scope.run(() => {
    const doubled = computed(() => counter.value * 2)

    watch(doubled, () => console.log(doubled.value))

    watchEffect(() => console.log('Count: ', doubled.value))
  })

  // 处理掉当前作用域内的所有 effect
  scope.stop()
  ```

## getCurrentScope()

如果有的话，返回当前活跃的 [effect 作用域](https://cn.vuejs.org/api/reactivity-advanced#effectscope)。

* **类型**
  **ts**
  ```
  function getCurrentScope(): EffectScope | undefined
  ```

## onScopeDispose()

在当前活跃的 [effect 作用域](https://cn.vuejs.org/api/reactivity-advanced#effectscope)上注册一个处理回调函数。当相关的 effect 作用域停止时会调用这个回调函数。

这个方法可以作为可复用的组合式函数中 `onUnmounted` 的替代品，它并不与组件耦合，因为每一个 Vue 组件的 `setup()` 函数也是在一个 effect 作用域中调用的。

如果在没有活跃的 effect 作用域的情况下调用此函数，将会抛出警告。在 3.5+ 版本中，可以通过将第二个参数设为 `true` 来消除此警告。

* **类型**
  **ts**
  ```
  function onScopeDispose(fn: () => void, failSilently?: boolean): void
  ```
