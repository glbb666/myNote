1. 支持组件不注册也能使用

# 是什么

`setup()` 钩子是在组件中使用组合式 API 的入口。

# 怎么做

## 使用场景

`setup`通常只在以下情况下使用：

1. 需要在非单文件组件中使用组合式 API 时。
2. 需要在基于选项式 API 的组件中集成基于组合式 API 的代码时。

注意

对于结合单文件组件使用的组合式 API，推荐通过 [`<script setup>`](https://cn.vuejs.org/api/sfc-script-setup.html) 以获得更加简洁及符合人体工程学的语法。

## 数据访问

### 访问setup的数据

我们可以使用响应式 API来声明响应式的状态，在 `setup()` 函数中返回的对象会暴露给**模板和组件实例（this）**。

其他的选项也可以通过组件实例来获取 `setup()` 暴露的属性：

```html
<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)

    // 返回值会暴露给模板和其他的选项式 API 钩子
    return {
      count
    }
  },

  mounted() {
    console.log(this.count) // 0
  }
}
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

`setup()` 自身并不含对组件实例的访问权，即在 `setup()` 中访问 `this` 会是 `undefined`。你可以在选项式 API 中访问组合式 API 暴露的值，但反过来则不行。

### 访问 Props

`setup` 函数的第一个参数是组件的 `props`。和标准的组件一致，一个 `setup` 函数的 `props` 是响应式的，并且会在传入新的 props 时同步更新。

```js
export default {
  props: {
    title: String
  },
  setup(props) {
    console.log(props.title)
  }
}
```

请注意如果你解构了 `props` 对象，解构出的变量将会丢失响应性。因此我们推荐通过 `props.xxx` 的形式来使用其中的 props。

如果你确实需要解构 `props` 对象，或者需要将某个 prop 传到一个外部函数中并保持响应性，那么你可以使用 [toRefs()](https://cn.vuejs.org/api/reactivity-utilities.html#torefs) 和 [toRef()](https://cn.vuejs.org/api/reactivity-utilities.html#toref) 这两个工具函数，具体见响应式API

```js
import { toRefs, toRef } from 'vue'

export default {
  setup(props) {
    // 将 `props` 转为一个其中全是 ref 的对象，然后解构
    const { title } = toRefs(props)
    // `title` 是一个追踪着 `props.title` 的 ref
    console.log(title.value)

    // 或者，将 `props` 的单个属性转为一个 ref
    const title = toRef(props, 'title')
  }
}
```

### Setup 上下文

传入 `setup` 函数的第二个参数是一个 **Setup 上下文**对象。

它包含了组件实例的一些重要属性和方法，但没有组件实例的全部功能。

```js
export default {
  setup(props, context) {
    // 透传 Attributes（非响应式的对象，等价于 $attrs）
    console.log(context.attrs)

    // 插槽（非响应式的对象，等价于 $slots）
    console.log(context.slots)

    // 触发事件（函数，等价于 $emit）
    console.log(context.emit)

    // 暴露公共属性（函数）
    console.log(context.expose)
  }
}
```

该上下文对象是非响应式的，可以安全地解构：

```js
export default {
  setup(props, { attrs, slots, emit, expose }) {
    ...
  }
}
```

`attrs` 和 `slots` 都是有状态的对象，它们总是会随着组件自身的更新而更新。这意味着你应当避免解构它们，并始终通过 `attrs.x` 或 `slots.x` 的形式使用其中的属性。

此外还需注意，和 `props` 不同，`attrs` 和 `slots` 的属性都**不是**响应式的。如果你想要基于 `attrs` 或 `slots` 的改变来执行副作用，那么你应该在 `onBeforeUpdate` 生命周期钩子中编写相关逻辑。

```html
<template>
  <div>
    <p>Message from attrs: {{ attrs.message }}</p>
    <slot></slot>
  </div>
</template>

<script>
import { onBeforeUpdate, ref } from 'vue';

export default {
  setup(props, { attrs, slots }) {
    const previousAttrs = ref({ ...attrs });
    const previousSlots = ref({ ...slots });

    onBeforeUpdate(() => {
      if (JSON.stringify(attrs) !== JSON.stringify(previousAttrs.value)) {
        console.log('Attributes changed:', attrs);
        previousAttrs.value = { ...attrs };
        // 在这里编写基于 attrs 改变的副作用逻辑
      }

      if (JSON.stringify(slots) !== JSON.stringify(previousSlots.value)) {
        console.log('Slots changed:', slots);
        previousSlots.value = { ...slots };
        // 在这里编写基于 slots 改变的副作用逻辑
      }
    });

    return {
      attrs
    };
  }
};
</script>
```

### 访问组件实例

如果你确实需要在 `setup` 中访问组件实例，可以使用 `getCurrentInstance` 方法：

```js
import { getCurrentInstance } from 'vue';

export default {
  setup(props, context) {
    const instance = getCurrentInstance();
    console.log(instance);

    return {};
  }
};
```

### 暴露公共属性

`expose` 函数用于显式地限制该组件暴露出的属性，当父组件通过[模板引用](https://cn.vuejs.org/guide/essentials/template-refs.html#ref-on-component)访问该组件的实例时，将仅能访问 `expose` 函数暴露出的内容：

```js
export default {
  setup(props, { expose }) {
    // 让组件实例处于 “关闭状态”
    // 即不向父组件暴露任何东西
    expose()

    const publicCount = ref(0)
    const privateCount = ref(0)
    // 有选择地暴露局部状态
    expose({ count: publicCount })
  }
}
```


### 异步使用setup

`setup()` 应该*同步地*返回一个对象。唯一可以使用 `async setup()` 的情况是，该组件是 [Suspense](https://cn.vuejs.org/guide/built-ins/suspense.html) 组件的后裔。

步骤如下：

#### 1. 创建一个异步组件

首先，我们创建一个异步组件。在这个组件中，我们模拟一个异步操作，比如从服务器获取数据。

```html
// AsyncComponent.vue
<template>
  <div>
    <h1>Async Component</h1>
    <p>Data: {{ data }}</p>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  async setup() {
    const data = ref(null);

    // 模拟异步操作，例如从服务器获取数据
    await new Promise(resolve => setTimeout(resolve, 2000));
    data.value = 'Fetched data from server';

    return {
      data
    };
  }
};
</script>
```

#### 2. 使用 `Suspense` 组件

接下来，我们在父组件中使用 `Suspense` 组件，并将异步组件作为其子组件。我们还添加一个 `template #fallback` 来展示加载时的占位内容。

```html
// App.vue
<template>
  <div id="app">
    <h1>Suspense Example</h1>
    <Suspense>
      <template #default>
        <AsyncComponent />
      </template>
      <template #fallback>
        <div>Loading...</div>
      </template>
    </Suspense>
  </div>
</template>

<script>
import AsyncComponent from './AsyncComponent.vue';

export default {
  components: {
    AsyncComponent
  }
};
</script>
```

## 与渲染函数一起使用

`setup` 也可以返回一个[渲染函数](https://cn.vuejs.org/guide/extras/render-function.html)，此时在渲染函数中可以直接使用在同一作用域下声明的响应式状态：

**js**

```
import { h, ref } from 'vue'

export default {
  setup() {
    const count = ref(0)
    return () => h('div', count.value)
  }
}
```

返回一个渲染函数将会阻止我们返回其他东西。对于组件内部来说，这样没有问题，但如果我们想通过模板引用将这个组件的方法暴露给父组件，那就有问题了。

我们可以通过调用 [`expose()`](https://cn.vuejs.org/api/composition-api-setup.html#exposing-public-properties) 解决这个问题：

**js**

```
import { h, ref } from 'vue'

export default {
  setup(props, { expose }) {
    const count = ref(0)
    const increment = () => ++count.value

    expose({
      increment
    })

    return () => h('div', count.value)
  }
}
```

此时父组件可以通过模板引用来访问这个 `increment` 方法。
