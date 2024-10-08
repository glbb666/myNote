### 框架整体方面

#### F框架和React框架的区别

React 本身**只关注视图层**，不涉及应用的其它部分，比如状态管理和路由处理。一般需要手动引入React-router、Redux、redux-saga等流行库进行管理。

F框架基于Dva 框架，**同时集成了 Redux、React-redux和 redux-saga** 等流行的库，是一个更为完善的前端应用解决方案。它不仅包含了 React 的视图层功能，还对应用中的数据流管理（通过 Redux）和异步操作处理（通过 redux-saga）提供了一套解决方案。

下面从路由和数据管理论述F框架和传统React框架的区别

### 路由方面

#### F框架和React框架的区别

React一般使用React-router做路由管理，适用于SPA单页面应用，通过切换组件的方式进行路由。

F框架适用于MPA多页面应用，每个页面都有自己的URL，并且在用户访问不同页面时会重新加载整个页面。

### 数据管理方面

#### F框架和React框架的区别

F框架推荐使用 model 和 service 的结构来组织代码，它并没有直接抽离出 Services 层，而是提供了一种机制（通过 effects 和 models）来组织和处理与服务层交互的逻辑。

```js
// models/todos.js
import * as todoServices from '../services/todos';

export default {
  namespace: 'todos',
  state: [],
  effects: {
    *fetchTodos(action, { call, put }) {
      const response = yield call(todoServices.fetchTodos);
      yield put({ type: 'saveTodos', payload: response.data });
    },
  },
  reducers: {
    saveTodos(state, { payload: todos }) {
      return [...todos];
    },
  },
};

// services/todos.js
export function fetchTodos() {
  return fetch('/api/todos').then(response => response.json());
}

```

对于React来说，services部分的代码只能在组件中调用。

```js
// services/todos.js
export function fetchTodos() {
  return fetch('/api/todos').then(response => response.json());
}

// components/TodoList.js
import React, { useState, useEffect } from 'react';
import { fetchTodos } from '../services/todos';

function TodoList() {
  const [todos, setTodos] = useState([]);

  useEffect(() => {
    fetchTodos().then(data => setTodos(data));
  }, []);

  return (
    <div>
      {todos.map(todo => (
        <div key={todo.id}>{todo.text}</div>
      ))}
    </div>
  );
}

export default TodoList;
```

引入redux可以解决视图层和数据层耦合的问题

### F框架和redux的区别

- 可处理的数据上：redux只能处理同步数据，F框架能处理同步数据和异步数据
- 和action的关系上：
  - redux不会监听action的更新。redux提供了一个store，store内部维护了一系列reducer和当前的应用状态state。当action被dispatch到store之后，store会自动调用所有的reducer，并传入两个参数，当前的state和被呈递的action。当多个reducer需要响应同一个action时，它们各自独立的处理自己的状态部分，并返回新的状态。redux会把这些返回的状态合并成一个新的状态。

#### 和redux-saga的区别

引入react-saga其实可以解决视图逻辑和副作用耦合的问题。

> 这里讨论的是F框架中集成的redux-saga和React中引用的原生redux-saga的区别

`redux-saga` 是一个 Redux 中间件，它通过使用 ES6 的 `Generator` 函数来让你用同步的方式编写异步的代码。

- **快速开发和减少配置**：在React中如果需要使用该中间件，需要手动配置，手动引入。F框架继承了redux-saga，做到了开箱即用。
- **易用性**：使用saga有一定的学习成本。Sagas 通过 `takeLatest`, `takeEvery`, `all`, `call`, `put` 等 effects 功能构建复杂的数据流和副作用管理。F框架封装了redux-saga，在model中调用reducers和effects，effects中处理副作用，简化了调用流程

```js
// 使用 F框架
export default {
  namespace: 'user',
  state: {},
  reducers: {
    save(state, { payload: { user } }) {
      return { ...state, user };
    },
  },
  effects: {
    *fetchUser({ payload: { id } }, { call, put }) {
      const user = yield call(ApiService.fetchUser, id);
      yield put({ type: 'save', payload: { user } });
    },
  },
};
```

总结：F框架 和 redux-saga 都可以实现副作用的管理。不过，redux-saga 更倾向于提供一系列底层 API，允许你有更多的控制权和灵活性，但相应地也带来了更复杂的代码和配置。而 F框架 提供了一种更简洁、更高级的抽象方式，有助于减少样板代码和提高开发效率，特别适合不太熟悉 Redux 体系和需要快速开发的场景。然而，F框架隐藏了很多细节和内部工作原理，对于需要更细致控制其 Redux 应用的开发者来说，这可能会是一个限制。

### 为什么使用F框架（思考）

**使用React框架的情况：**

1. **小型项目或简单的应用：** 如果你的项目比较小，不需要复杂的状态管理和路由，或者你只需要构建一个简单的用户界面，使用React可能就足够了。
2. **自定义需求：** 如果你想要**更细粒度的控制你的应用结构或选择自己的库来处理路由和状态管理**，那么直接使用React会比较合适。

**使用F框架的情况：**

因为F框架用来搭建MPA多页面应用，适用于中到大型项目，业务结构复杂，数据量比较庞大的情况。

- 性能和加载时间：SPA项目载入时会加载整个页面的资源，如果页面的资源过于庞大，可能会导致首屏时间过长，MPA项目只会加载需要使用的页面的资源。

  - 比如一个大型组织中，可能有多个开发小组，每个小组手里都有n个项目，这些项目都比较复杂。由于引流、或者需求设计链路的原因，需要短暂的互相跳转，MPA数据记载时间短的特性就比较合适
- 可拓展性：由于每个页面都是独立的，有利于拓展和增加新的页面。
- 项目的独立性与维护：每个项目都能独立存在，易于维护，一旦其中一个页面需要更新或维护，不会影响到其他页面，降低了代码冲突的风险，有利于团队分工。
- **快速开发和减少配置：** F封装了多个技术栈，并带有一套初始化的配置，这让开发者可以快速开始编写业务代码而不需要花太多时间在环境搭建上。

### F框架与Create React App脚手架的异同

#### 相同点

- 简化了开发流程，提供了一系列工具和配置来加速开发

#### 差异点

- **主要功能**

  - F框架是基于 Redux 和 Redux-saga 实现的数据管理框架，提供**数据管理和副作用管理的标准化**解决方案
  - CRA 是一个开箱即用的脚手架，可以加快搭建项目基础结构，并且提供开发、构建、测试环境
- **初始化项目**

  * F框架不提供创建新项目结构的工具；它是集成到现有项目中的框架。
  * CRA 提供了一条命令（`npx create-react-app my-app`），可以快速生成新的 React 项目结构，包括配置、脚本和一些初始文件。
- **配置和可定制性**

  - F框架是不可配置的，它已经把应用的状态管理和路由都配置好了
  - CRA 提供了预定义的Webpack配置，可以进行修改
- **项目结构和编码规范**

  - F框架推荐使用 model 和 service 的结构来组织代码，以及具体的编写模式。
  - CRA 不强制任何项目结构或编码规范，它只提供了一个非常基本的项目模板。
- **工具链**

  * F框架提供了一整套工具来管理数据（model、effects、subscriptinos）和处理异步操作。
  * CRA 提供了一整套工具来处理应用开发环境，比如静态服务器、热模块替换等。

总结：F框架 是一个**集成的数据流解决方案**，而 CRA 是一个帮助你**初始化一个新 React 项目的工具**。虽然它们都是 React 生态的一部分，但分别关注了应用开发过程中的不同方面：F框架 **关注于状态管理和数据流**，而 CRA 关注于**提供项目的起始模板和开发环境**。在实践中，可以使用 CRA 初始化项目，然后在项目中集成 F框架 来管理数据流。

### F框架的弊端以及解决方案

#### 问题：MPA导致的每个页面都要刷新的问题

背景：对于MPA（多页面应用）而言，它的首页需要呈现多个模块，每个模块都是单独的功能，并且都有一定的重要性，用户需要频繁的切换。这时MPA导致的页面刷新问题会导致组件不流畅。

##### 解决方案1：可配置底部BottomTab。（可行）

原因：使用底部的 tab 导航来在不同组件之间切换，可以实现一个类似单页面应用 (SPA) 的用户体验，在视觉上提供“切换页面”的感觉，而实际上只是在同一页面内交替显示或隐藏不同的内容。

好处：

- **用户体验** ：底部的 tab 导航提供了一种直观的方式让用户在不同的功能模块间快速切换，用户无需等待新页面加载，从而得到更加流畅的体验。
- **数据共享**：由于实际页面没有发生变化，你可以很容易地在组件间共享状态。用户的交互状态也可以得到保留（提取到外层不被销毁）。

实践：在业务通用解-多选底部栏目中。

缺点：

- **首屏时间**：因为所有的组件都在首页加载，所以初始页面的加载时间可能会更长。

> 使用类似方案的app：闲鱼

##### 解决方案2：model的懒加载。（不可行）

原因：虽然dva框架支持，但是F框架不支持。

思路：model是按照不同的数据类型划分的，而不同的tab的组件会选用这些数据类型的其中几个。比如tab1使用的是数据类型a和数据类型b，那么就先懒加载这两个model，然后之后tab2使用数据类型a的时候就不需要加载了。

##### 解决方案3：请求维度优化（可行）

- 请求按需发起：在用户切换到对应的 tab 时，通过调用 model 中的 `effects` 来发起数据请求。
- 请求缓存：如果数据在一段时间内不会变化，或者发生变化其实没有那么重要。可以将响应数据缓存起来，当组件需要相同的数据时，先检查是否存在缓存的数据，如果有则使用缓存的数据，不再发起新的请求。（如果一段时间需要刷新一次可以加一个时间戳，判断数据是什么时候拉取的，有没有必要再拉）
- 智能加载和查找：应用有海量列表数据，分页加载只加载用户即将查看的内容。
- 避免不必要的 re-render：使用 `React.memo`、正确使用 React Hooks 的依赖数组等进行优化。
- 懒加载组件：在组件切换时加载新的组件代码，减少 JavaScript bundle 的大小，减少初次页面加载的时间。
