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

通过这个优先级我们可以获取一个该更新必须执行的过期时间，优先级越高那么过期时间就越近，反之亦然。这个过期时间是用来判断该更新是否已经过期，如果过期的话就会马上执行该更新。

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

#### **更新队列**

存储位置：每个Fiber节点中

存储单元：Fiber的每个更新

创建：在更新被创建时，会附带一个过期时间。这个过期时间用来确定更新的优先级。过期时间的产生下面会讲。

入队：更新会被放入同步更新队列队尾。

处理更新：

- 当需要调度时，调度器会携带一个此次渲染的时间，遍历更新队列。
- 调度器基于上次Fiber的state，把符合条件的更新都在state上进行应用，并移出队列。等全部应用完成，再把Fiber的state替换成这个新的state。

### Fiber的任务

#### 任务的产生

在调度阶段，会遍历Fiber树。（以什么形式遍历，是一整棵树还是一部分树我们之后再讨论）

当发现Fiber的更新队列存在时，就会**根据更新队列和最早的更新时间创建一个任务**。

```js
// 合成任务的函数
function createTask(expirationTime) {
  return {
    expirationTime,
    updates: [],
  };
}

// 在调度阶段合成任务
function createTaskFromUpdates(updates) {
  //这里的updates是单个fiber节点的更新队列
  let earliestExpiration = NoWork;
  for (const update of updates) {
    if (update.expirationTime < earliestExpiration) {
      earliestExpiration = update.expirationTime;
    }
  }
  const task = createTask(earliestExpiration);
  task.updates = updates;
  return task;
}
```

#### 任务的类型

同步任务：如果是高优先级的任务且当前没有任务在执行，会直接执行

异步任务：任务会进入队列，等到合适的时机（如当前帧有空闲时间）执行

#### 任务的优先级 & 过期时间

这一块比较复杂。放在一起讲。

1. 任务生成时，从更新队列中获取最早的 `expirationTime`。
2. 任务的优先级基于这个 `expirationTime`计算。`expirationTime`与**当前时间**的差异越小，任务优先级越高，越早执行。
3. 在调度任务时，根据任务的优先级进一步调整任务的 `expirationTime`，计算出最终的 `renderExpirationTime`，这确保了高优先级任务能按时执行。

#### 任务队列

存储位置：全局

存储单元：任务

任务队列是一个全局唯一的同步队列。

入队：所有的任务都会被放入同步任务队列中，然后按任务过期时间进行升序排序，这样每次都可以取出最早的任务。

#### 任务的调度

当任务入队之后，开始任务的调度。

任务调度有两种模式，是在调度不同优先级的任务时触发的。

1. flushSyncCallbackQueue。这种模式在任务优先级高，且没有任务在执行中的时候调用，一旦调用开启，会把当前同步队列中所有回调（即任务）依次执行，这里的任务优先级较高，是不会被打断的。
2. flushWork。这种模式在当前任务优先级低的时候使用，使用的是 `requestIdleCallback` 在空闲时间处理这些任务。一旦开启，只会执行过期任务，或者当前帧有剩余时间，会继续执行任务。这里的任务优先级较低，当碰到更高优先级的任务的时候会被打断。

```js
// scheduleCallback 是任务调度的入口
function scheduleCallback(priorityLevel, callback, options) {
  // 获取当前时间
  const currentTime = getCurrentTime();
  const startTime = currentTime;
  // 根据任务优先级获取时间片
  const timeout = expirationTimeForPriorityLevel(priorityLevel);
  // 计算出任务的过期时间
  const expirationTime = startTime + timeout;

  const newTask = {
    id: taskIdCounter++,
    callback,
    priorityLevel,
    startTime,
    expirationTime,
  };

  // 将任务放入队列
  pushTask(newTask);
  // 按过期时间升序排序任务队列
  sortTaskQueue(taskQueue);


  // 如果任务优先级高，立即调度执行
  if (newTask.priorityLevel === ImmediatePriority) {
    // 如果当前没有正在执行的任务
    if (!isPerformingWork) {
      // 执行任务
      flushSyncCallbackQueue();
    }
    // 如果当前有在执行的任务，因为任务已经入队，且flushSyncCallbackQueue会清空同步队列，所以等待执行就好了。
  } else {
    // 注册下一个空闲时间执行任务
    requestHostCallback(flushWork);
  }
}

function flushSyncCallbackQueue() {
  while (syncQueue.length > 0) {
    const callback = syncQueue.shift();
    try {
      callback();
    } catch (error) {
      console.error(error);
    }
  }
}


// flushWork 执行任务队列
function flushWork(hasTimeRemaining, initialTime) {
  let currentTask = peekTaskQueue();

  while (currentTask !== null) {
    //当任务已经过期 或者 当前帧还有剩余的时间去执行任务
    if (currentTask.expirationTime <= initialTime || hasTimeRemaining) {
      const callback = currentTask.callback;
      if (callback !== null) {
        currentTask.callback = null;
        const didTimeout = currentTask.expirationTime <= initialTime;
        callback(didTimeout);
      }
    } else {
      break;
    }
    //从任务队列头部取出一个任务
    currentTask = peekTaskQueue();
  }

  return null;
}

```

### 批处理模式

#### 是什么

**批处理模式** 是 React 的优化性能的机制。

在批处理模式下，React 会将多个状态更新（`setState`）或其他更新操作（如 `useReducer` 的 `dispatch`）合并在一起，延迟到一个批次中统一执行，而不是逐一同步处理每个更新。

#### 为什么

* **优化性能** ：批处理模式减少了在同一帧内多次渲染的可能性，从而降低了浏览器的负载，尤其是在复杂组件或应用中，性能提升尤为明显。
* **一致性** ：在同一批次中的所有更新被视为一个事务，避免出现中间状态。

#### 开启时机

批处理模式大多在React的控制范围内自动开启，如事件处理和生命周期函数中。也可以通过 `unstable_batchedUpdates` 手动开启。

当开启批处理模式时，全局的isBatchingUpdates标识被置为true。

#### 关闭时机

- 原生 DOM 事件处理程序：如直接通过 `addEventListener` 绑定的事件处理器。
- 非 React 管理的环境中进行的状态更新： setTimeout 或 setInterval 回调中。
- 手动控制：你可以通过 `flushSync` 强制让 React 立即渲染，而不等待批处理。这样做会关闭当前批处理机制。

#### 批处理模式和非批处理模式下的表现

我们先说非批处理模式，因为它更简单。

以下是非批处理模式下的更新处理流程的源码片段示例：

1. **立即更新触发** ：非批处理模式下，更新触发后会立即调用调度函数。

```js
function setState(newState) {
  // 调用更新状态的函数，不使用批处理模式
  const update = createUpdate(newState);

  // 立即调度
  scheduleUpdate(update);
}
```

2. **任务生成和调度** ：在 `scheduleUpdate` 中，React 会根据更新生成一个任务，并立即开始调度这个任务。

```js
function scheduleUpdate(update) {
  const currentTime = getCurrentTime();
  const expirationTime = computeExpirationTime(currentTime, update);

  // 创建任务并立即调度
  const task = createTask(expirationTime);
  scheduleCallback(task);
}
```

3. **立即执行更新** ：调度函数 `scheduleCallback` 会立即尝试执行任务，不会等待其他更新的完成。

```js
function scheduleCallback(task) {
  // 如果当前没有执行中的任务，直接执行
  if (!isPerformingWork) {
    performWork(task);
  } else {
    // 否则任务进入队列等待执行
    pushTaskToQueue(task);
  }
}

```

4. **立即渲染** ：如果 `isPerformingWork` 为 `false`，React 会立即调用 `performWork` 函数来处理任务。任务会包含更新队列中的更新并立即进行渲染。

```js
function performWork(task) {
  isPerformingWork = true;

  // 执行任务，渲染组件
  renderComponent(task);

  isPerformingWork = false;
}
```

总结：在非批处理模式下，每一个更新会立刻触发调度，并且在没有其他任务执行的情况下，会立刻渲染。这个过程是同步的，不会累积多次更新来进行批量处理。

# flushSyncCallbackQueue和批处理模式的区别

flushSyncCallbackQueue是在高优先级任务调度，需要立刻执行的时候触发。批处理模式是React根据剩余帧，任务调度等情况，判断出一些更新可以合并在一次渲染内执行。

# 接着是调度（todo:什么时候会发生调度——

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
