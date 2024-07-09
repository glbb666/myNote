### 什么是Fiber

Fiber 是 React 16 中引入的调度器算法，用于优化渲染性能。

### 什么是Fiber节点

Fiber节点是React Fiber架构中用来分解和管理渲染任务的“最小单位”。这种方式可以避免长时间占用主线程，从而提高应用程序的响应性和性能。

在 React 中，每个组件都有一个对应的 Fiber 节点。

Fiber 节点存储了组件的状态、属性、类型以及对应的 DOM 节点等信息。

Fiber 节点构成Fiber树。

### Fiber的更新

#### 更新的产生

当调用 `setState` 或其他更新方法时，会触发状态更新。

React 会创建一个更新对象，该对象包含更新的状态、过期时间和优先级等信息。

该更新对象会被添加到对应 `fiber` 节点的更新队列的队尾。

#### 更新的类型

更新可以粗略分为同步更新和异步更新。不同类型的更新有不同的调度方式。

同时，同步更新和异步更新内部还可以进行细分。

不同的更新对应着不同的处理方式，比如会不会触发批处理模式。以及不同的优先级。不同的优先级又对应着不同的更新时间。

##### **同步更新**：

**事件处理：**

通常具有最高的优先级，React会立即处理这些更新。

**动画更新：**

js动画也具有较高的优先级，使得动画可以流畅执行。

为什么说是js动画呢？

而js动画是通过js控制dom的样式实现的，所以需要通过浏览器渲染进程的主线程去执行。

##### **异步更新**

**数据获取** ：

网络请求通常是异步的，React 会等待数据返回后再更新状态。

**离屏更新** ：

对于不可见的组件（例如滚动屏幕外的内容），它们的更新优先级通常是最低的，因为用户当前看不到它们的变化。

同时，为了渲染能够更高效，React还创建了一种更新的模式，叫批处理模式。

#### 更新的优先级

React内部有一个优先级系统，可以根据不同类型的更新分配不同的优先级。

通过这个优先级我们可以获取一个该更新任务必须执行的过期时间，优先级越高那么过期时间就越近，反之亦然。这个过期时间是用来判断该任务是否已经过期，如果过期的话就会马上执行该任务。

截止时间低的一般在浏览器的主线程空闲时才执行。

一般分为同步更新和异步更新，同步更新和异步更新中的事件也有相应的优先级。

#### 更新的过期时间

过期时间基于当前时间和优先级计算。

优先级通过时间片来量化。即**过期时间为当前时间和时间片之和**

React 内部会定义不同优先级对应的时间片长度。例如：

```js
const IMMEDIATE_PRIORITY_TIMEOUT = -1;
const USER_BLOCKING_PRIORITY_TIMEOUT = 250; // 250ms
const NORMAL_PRIORITY_TIMEOUT = 5000; // 5s
const LOW_PRIORITY_TIMEOUT = 10000; // 10s
const IDLE_PRIORITY_TIMEOUT = Infinity;

```

React 会根据当前时间和对应的时间片长度来计算过期时间。例如，对于用户交互事件（假设优先级为 `USER_BLOCKING_PRIORITY`）：

```js
const expirationTime = currentTime + USER_BLOCKING_PRIORITY_TIMEOUT;
```

#### **更新队列**

存储位置：每个Fiber节点中

存储单元：Fiber的每个更新

创建：在更新被创建时，会附带一个过期时间。这个过期时间用来确定更新的优先级。过期时间的产生下面会讲。

入队：更新会被放入同步更新队列队尾。

处理更新：

- 当需要调度时，调度器会携带一个此次渲染的时间，遍历更新队列。
- 调度器基于上次Fiber的state，把符合条件的更新都在state上进行应用，并移出队列。等全部应用完成，再把Fiber的state替换成这个新的state。

### 批处理模式

批处理模式批处理的是多个Fiber节点的更新。

批处理模式执行全局同步队列，执行所有同步队列中传入的处理fiber节点的回调。让这些fiber节点的state都基于更新队列中过期时间内的更新进行变更，变为最新的state。

#### 如何进入批处理模式

* **事件处理器** ：在 React 事件处理函数中（如 `onClick`、`onChange` 等），React 会自动进入批处理模式。即使在事件处理函数中调用多个 `setState`，这些状态更新也会被批处理。
* **生命周期方法** ：在组件的生命周期方法（如 `componentDidMount`、`componentDidUpdate`）中，React 也会自动进入批处理模式。
* **调用 `batchedUpdates` 方法** ：可以显式调用 `ReactDOM.unstable_batchedUpdates` 将多个更新批处理。
* **异步任务回调** ：在异步任务（如 `Promise` 回调、`setTimeout` 回调等）中，React 也会进入批处理模式。

#### 如何退出批处理模式

* **事件处理器执行完毕** ：当 React 事件处理函数执行完毕后，批处理模式会结束，并触发状态更新。
* **生命周期方法执行完毕** ：当生命周期方法执行完毕后，批处理模式会结束，并触发状态更新。
* **显式调用 `flushSync`** ：可以显式调用 `ReactDOM.flushSync` 立即处理同步队列中的更新。
* **异步任务回调执行完毕** ：当异步任务回调执行完毕后，批处理模式会结束，并触发状态更新。

以下是一个示例代码，展示批处理模式和非批处理模式下的行为：

```js
let syncQueue = null;
let isBatchingUpdates = false;

function batchedUpdates(callback) {
  const prevIsBatchingUpdates = isBatchingUpdates;
  isBatchingUpdates = true;
  try {
    return callback();
  } finally {
    isBatchingUpdates = prevIsBatchingUpdates;
    if (!isBatchingUpdates) {
      flushSyncCallbackQueue();
    }
  }
}

function flushSyncCallbackQueue() {
  if (syncQueue !== null) {
    const queue = syncQueue;
    syncQueue = null;
    for (let i = 0; i < queue.length; i++) {
      const callback = queue[i];
      callback();
    }
  }
}

function scheduleUpdate(fiber, update) {
  const updateQueue = fiber.updateQueue;
  if (updateQueue.lastUpdate === null) {
    updateQueue.firstUpdate = update;
  } else {
    updateQueue.lastUpdate.next = update;
  }
  updateQueue.lastUpdate = update;

  if (isBatchingUpdates) {
    if (syncQueue === null) {
      syncQueue = [() => processUpdateQueue(fiber)];
    } else {
      syncQueue.push(() => processUpdateQueue(fiber));
    }
  } else {
    processUpdateQueue(fiber);
  }
}

function processUpdateQueue(fiber) {
  const updateQueue = fiber.updateQueue;
  let firstUpdate = updateQueue.firstUpdate;
  let newState = fiber.memoizedState;

  while (firstUpdate !== null) {
    newState = applyUpdate(newState, firstUpdate);
    firstUpdate = firstUpdate.next;
  }

  fiber.memoizedState = newState;
  updateQueue.firstUpdate = null;
  updateQueue.lastUpdate = null;
}

function applyUpdate(state, update) {
  return { ...state, ...update.payload };
}

// 使用示例
batchedUpdates(() => {
  scheduleUpdate(fiber1, { payload: { foo: 'bar' } });
  scheduleUpdate(fiber2, { payload: { baz: 'qux' } });
});

```

#### 全局同步队列

批处理模式依赖全局同步队列。

存储位置：全局

存储单元：对某个Fiber节点的处理。

创建：当触发批处理模式时，对某个Fiber节点的调度不会立刻执行，而是进入同步队列。

处理更新：

当退出批处理模式时，会清空同步队列。即每个Fiber节点都会进行调度，基于自己的旧state和更新队列去更新自己的state值。

渲染：

一次批处理周期内，所有涉及的 `fiber` 节点都会完成状态更新，并在同一个渲染周期内更新实际的 DOM

### 更新处理流程（setState之后会发生什么）

见调用setState之后会发生什么

### diff流程

见react的diff算法

-------------以下施工中-----------------------------

### 执行与中断

当开始执行更新任务时，如果有新的更新任务进 来，那么调度器就会按照两者的优先级大小来进行决策会判断两个任务的优先级高低。假如新任务优先级高，那么打断旧的任务，重新开始，否则继续执行任务。

中断的任务不会被遗忘。一旦高优先级任务完成，React可以选择合适的时机回到之前被中断的任务，继续之前的工作。

Fiber 还引入了一些新的 React 特性，例如时间分片和并发模式。

#### 时间分片

##### 是什么

时间分片可以让 React 在渲染期间暂停和继续工作，以避免阻塞 UI 更新。

##### 原理

时间分片是一种调度机制，它不是依赖于传统的线程管理，而是基于浏览器提供的 requestIdleCallback API。该API允许开发者在浏览器的空闲时段执行低优先级的任务，而不影响高优先级的任务，如动画和输入响应。

React 使用这个API来实现时间分片。当React开始渲染更新时，它将工作分解成多个小任务。每个任务都有自己的截止时间（deadline），React会在浏览器空闲时处理这些小任务，如果在截止时间之前任务还没有完成，React会将控制权交还给浏览器，以便浏览器可以处理更高优先级的工作（如用户输入响应），然后在浏览器再次空闲时，React会继续之前的工作。

这种机制允许React可以在渲染组件时“暂停”工作，然后在浏览器有空闲时间时“继续”工作，这样就不会阻塞主线程，从而提高应用程序的响应性和性能。

requestIdleCallback API允许开发者指定一个回调函数，在浏览器的空闲周期调用。React 实现时间分片的方式是通过在空闲周期内，分批次执行渲染更新任务，从而实现对任务的管理和调度。如果浏览器不支持 requestIdleCallback，React 会回退到使用 setTimeout 来模拟相同的行为。

#### 并发模式

并发模式允许 React 在后台执行多个任务，以实现对用户操作的更快响应。

总之，Fiber 是 React 的一种新算法，旨在通过对任务的分解和优先级调度，提高应用程序的性能和响应性。它为 React 带来了更好的渲染策略和新的开发特性。
