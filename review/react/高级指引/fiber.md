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

异步任务：任务会进入队列，等到合适的时机（如当前帧有剩余时间或者浏览器空闲时间）执行

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

# 任务的调度

# 是什么

任务调度是确定何时应该执行一个任务的过程。它是由调度器负责的，其核心工作是根据任务的优先级和过期时间来决定任务何时运行。

# 怎么做

当任务入队之后，开始任务的调度。

任务的调度有两种模式，是在调度不同优先级的任务时触发的。

1. flushSyncCallbackQueue。这种模式在任务优先级高，且没有任务在执行中的时候调用，一旦调用开启，会把当前同步队列中所有回调（即任务）依次执行，当碰到高优先级任务时，会立刻执行。当碰到低优先级任务，会使用浏览器的时间分片去执行。
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

# 任务的执行（任务的Render）

## 是什么

执行是指React根据调度的结果，开始处理任务内容，如执行计算、更新DOM等操作。

## 怎么做

如果当前没有其他任务在执行，React 会立刻开始执行优先级最高的任务。

在执行任务的过程中，React 会处理对应 `Fiber` 节点上的 `updateQueue`，根据 `renderExpirationTime` 判断 `updateQueue`中的哪些update需要立刻应用。当发现节点需要更新，就会创建出一个镜像fiber节点，这个镜像fiber节点的别名为fiber工作树节点。接着把镜像fiber节点中旧的state根据这些update生成新的state。

当碰到低优先级任务时，会使用到时间分片机制，让任务延迟执行。

## 时间分片

### 是什么

时间分片是低优先级任务的调用机制，基于浏览器提供的 requestIdleCallback API。该API允许开发者在浏览器的空闲时段执行低优先级的任务。

它可以让 React 在渲染期间暂停和继续工作，以避免阻塞 UI 更新。

### 怎么做

#### 1.  **任务分割** ：

* React 会将低优先级任务分割成更小的工作单元，这些工作单元可以在多个帧（frame）内逐步执行，而不是一次性全部完成。
* 在每个时间片内，React 会执行一部分任务，当耗尽当前帧的时间配额时，会暂停任务，等到浏览器空闲时再继续执行。

#### 2.  **任务的中间状态** ：

* 任务的中间状态会被保存，React 在下一个时间片继续从中断的地方开始执行。
* 这意味着任务的执行是可中断和可恢复的。React 通过更新 Fiber 树的中间状态来实现这一点，保证在下一个时间片继续从中断点执行。

#### 3.  **时间分片的执行** ：

* 时间分片的执行是在主线程上进行的。React 通过定期检查当前帧的剩余时间来决定是否继续执行任务。
* 具体来说，React 使用了 `requestIdleCallback` 或者 `requestAnimationFrame` 来安排下一次的任务执行。这些 API 可以在浏览器空闲时调度任务，从而实现任务的分片执行。
* 如果浏览器不支持 requestIdleCallback，React 会回退到使用 setTimeout 来模拟相同的行为。

## 执行与中断

低优先级任务和高优先级任务的执行与中断机制不同。

### 1. **低优先级任务**

* **时间片机制** ：
* React 使用时间片来处理低优先级任务。在每个时间片的末尾，React 会检查当前的执行时间，如果已经超过了时间片的限制，它会中断当前的任务。
* **打断机制** ：低优先级任务在时间片结束时会被自动中断，即使没有更高优先级的任务插入，时间片结束后任务也会暂停，等到下一个时间片再继续执行。
* **恢复机制** ：在下一个时间片开始时，React 会从上次任务暂停的地方继续执行。
* **时间片内的打断** ：
* 低优先级任务在时间片内也可能会被打断。如果在时间片执行过程中，出现了一个更高优先级的任务，React 会立即中断当前的低优先级任务，转而执行新的高优先级任务。

### 2. **高优先级任务**

* **检查点机制** ：
* 高优先级任务的打断通常发生在有更高优先级的任务被插入的情况下。React 在高优先级任务执行时会设立多个检查点，这些检查点是任务执行过程中的逻辑断点。
* **打断机制** ：如果在高优先级任务的执行过程中，有更高优先级的任务被调度，React 会在下一个检查点处暂停当前任务，并保存它的状态，转而执行更高优先级的任务。
* **恢复机制** ：在更高优先级的任务执行完后，React 会恢复之前被打断的高优先级任务，从它暂停的地方继续执行。

## 并发模式

并发模式就是基于执行中断机制和时间分配机制，允许React同时处理多个任务。

低优先级任务基于时间分片切片，高优先级任务基于检查点中断。所有任务碰到更高优先级的任务时都会被中断。记录中间态，等任务执行完成之后再恢复。

使得渲染任务更加高效和丝滑。

# 任务的Commit

当任务执行完成之后，React 会进入 Commit 阶段。React 会把镜像 Fiber 节点替换到当前的 Fiber 树上，生成一棵新的 Fiber 树。

# 任务的渲染

接着，React 使用 `requestAnimationFrame` 确保在下一帧之前，将新生成的 Fiber 树所代表的变化反映到真实的 DOM 上，完成渲染。这是一个同步过程，无法被中断。

渲染最终触发的条件是任务的完成状态和浏览器的帧管理机制：

* **高优先级任务完成后** : 如果高优先级任务（如用户输入）完成，并且时间允许，React 会立即进入 Commit 阶段进行渲染。
* **低优先级任务完成后** : 低优先级任务可能会被分成多个时间片执行。当这些任务执行完成后，如果时间允许，React 也会进入 Commit 阶段。
* **浏览器下一帧绘制之前** : React 通常会在浏览器即将进行下一帧绘制之前，通过 `requestAnimationFrame` 来触发渲染，确保 DOM 更新与绘制同步，从而避免用户界面出现闪烁或不一致。
* **强制更新** : 某些情况下，React 可能会在特定事件（如用户强制刷新）后立即触发渲染，无论是否还有未完成的低优先级任务。

# 批处理模式

## 是什么

**批处理模式** 是 React 的优化性能的机制。

在批处理模式下，React 会将多个状态更新（`setState`）或其他更新操作（如 `useReducer` 的 `dispatch`）合并在一起，延迟到一个批次中统一执行，而不是逐一同步处理每个更新。

同时任务也会进行批处理，开启批处理模式之后会把多个任务进行合并处理。

> 批处理模式下更新也会入队，但是其实这没什么必要，因为更新会被立刻创建为任务。入队是为了保持代码逻辑上的一致性。

## 为什么

* **优化性能** ：批处理模式减少了在同一帧内多次渲染的可能性，从而降低了浏览器的负载，尤其是在复杂组件或应用中，性能提升尤为明显。
* **一致性** ：在同一批次中的所有更新被视为一个事务，避免出现中间状态。

## 开启时机

批处理模式大多在React的控制范围内自动开启，如事件处理和生命周期函数中。也可以通过 `unstable_batchedUpdates` 手动开启。

当开启批处理模式时，全局的isBatchingUpdates标识被置为true。

## 关闭时机

- 原生 DOM 事件处理程序：如直接通过 `addEventListener` 绑定的事件处理器。
- 非 React 管理的环境中进行的状态更新： setTimeout 或 setInterval 回调中。
- 手动控制：你可以通过 `flushSync` 强制让 React 立即渲染，而不等待批处理。这样做会关闭当前批处理机制。

## 批处理模式&非批处理模式总流程对比（调用setState之后会发生什么）

### **1. 触发更新**

当用户触发更新（例如 `setState`）时，React 会创建一个 `update` 对象，并将其加入到触发更新的 Fiber 节点的 `updateQueue` 中。

### **2. Fiber 树的遍历和标记**

**非批处理模式** :

* 更新触发后，React 会立即从触发更新的 Fiber 节点开始，自下而上遍历 Fiber 树。
* 在遍历过程中，React 对每个沿途的 Fiber 节点进行标记，标识这些节点需要在当前任务中被处理。
* 这种标记是立即执行的，React 之后会直接进入任务调度阶段，执行这些更新。

**批处理模式** :

* 更新触发后，React 不会立即执行标记和渲染，而是将更新放入 `updateQueue` 中。
* Fiber 树不会立刻进行自下而上的遍历，标记也不会立即发生。相反，React 会等待批处理结束时，才开始标记过程。

### **3. 生成任务并放入任务队列**

**非批处理模式** :

* 在 Fiber 树标记完成后，React 会立即生成一个任务，该任务代表此次更新对应的工作，并将其放入任务队列。
* 任务生成后，React 会立即开始调度这个任务，并执行它。

**批处理模式** :

* 在批处理模式下，React 会在批处理结束时，对所有触发更新的 Fiber 节点统一处理。
* 批处理结束后，React 开始自下而上遍历 Fiber 树，对需要更新的 Fiber 节点进行标记。
* 对每个被标记的 Fiber 节点，React 会生成一个任务，这个任务会包含该节点的 `updateQueue` 中的更新。

### **4. 调度与执行任务**

任务被放入队列后，React 会根据任务的优先级来调度执行。

执行过程中，React 会处理每个任务所包含的 `updateQueue`，更新对应的 Fiber 树的镜像节点。

### 5. 渲染

当任务执行完成之后。

Fiber树需要更新的节点会被替换成相应的镜像节点，接着根据更新后的Fiber树渲染真实的DOM树

React 在此时会使用 `requestAnimationFrame` 在下一帧绘制前执行 Commit 阶段。

### **关键区别**

在 **非批处理模式** 中，更新触发后立即标记 Fiber 节点，并立即生成和执行任务。

在 **批处理模式** 中，更新触发后不会立即标记，而是等批处理结束后，再统一标记、生成任务并执行。

标记和任务生成的顺序取决于是否启用了批处理模式。如果有批处理，标记和任务生成是延迟的，等待批处理结束时才进行；如果没有批处理，标记和任务生成是同步进行的。

# 关键浏览器api

在 React 的 Fiber 架构中，`requestIdleCallback` 和 `requestAnimationFrame` 是两个关键的浏览器 API，它们在不同的情况下发挥作用，以确保 React 在执行任务时既高效又能保持 UI 的流畅性。

### 1. **`requestIdleCallback`**

* **时间分片** : 在并发模式下，React 会将低优先级任务分成小的时间片，通过 `requestIdleCallback` 来执行这些任务，从而避免阻塞 UI 渲染这种高优先级任务。

### 2. **`requestAnimationFrame`**

`requestAnimationFrame` 的主要作用是让浏览器在下一帧刷新之前执行指定的回调函数，这样可以确保所有的 DOM 操作都在浏览器的刷新周期内完成。通过在下一帧之前完成绘制，`requestAnimationFrame` 可以有效避免重绘过程中出现的闪烁或抖动问题。
