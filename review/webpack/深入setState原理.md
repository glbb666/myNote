### 深入setState原理

### 是什么

setState(updater, callback)这个方法是用来告诉react组件数据有更新，有可能需要重新渲染。

我们先看看以下两个场景

场景一：点击按钮，把count的值增加2次。

```
*export* *default* class App extends React.Component {
 state = {
  count:1
 }
 add = ()=>{
   this.setState({count:this.state.count+1})
   this.setState({count:this.state.count+1})
 }
 render(){
  const {count} = this.state;
  *return* (
   {count}
  )
 }
}
```

点击前：

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=127e0d210361bd9bd66eb1cb259b2171_8f118824ce50c961_boxcneZs71ST8DKdz3ObInpBfGe_RlUYZR4UikFevX0L6M3ko8otxEYD65ON)

点击后：

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=6fdd101323289fe18c81f73f681773a0_8f118824ce50c961_boxcnduoW77zOuOqXQH0P9l9xFg_nr34ilWMaRh20cuqc5B4pqtKxlmEvrp6)

这个稍微熟悉setState的人应该都知道，因为setState是批量更新的，所以会把多个update合成为一个。

场景二：点击按钮，在setTimeout中把count的值增加2次。

```
add = ()=>{
  setTimeout(()=>{//新增行1
   this.setState({count:this.state.count+1})
   this.setState({count:this.state.count+1})
  },0)//新增行2
}
```

点击前：

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=127e0d210361bd9bd66eb1cb259b2171_8f118824ce50c961_boxcneZs71ST8DKdz3ObInpBfGe_RlUYZR4UikFevX0L6M3ko8otxEYD65ON)

点击后：

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=e687668c408bd6080c125aa9d619b5bd_8f118824ce50c961_boxcnqPeVYWan7D48oagmTjtMfh_Dw6vDNivupOIKiEVhWJVoRqXqKbvyFFn)

那这又怎么解释呢？我们可以看一看源码一探究竟。

### 背景介绍

在讲源码之前，我们首先了解一些概念。

#### Fiber

Fiber是新的调和器，每次有新的更新任务发生的时候，调度器都会按照策略给这些任务分配一个优先级。比如说动画的更新优先级会高点，离屏元素的更新优先级会低点。

通过这个优先级我们可以获取一个该更新任务必须执行的截止时间，优先级越高那么截止时间就越近，反之亦然。这个截止时间是用来判断该任务是否已经过期，如果过期的话就会马上执行该任务。

当开始执行更新任务时，如果有新的更新任务进来，那么调度器就会按照两者的优先级大小来进行决策会判断两个任务的优先级高低。假如新任务优先级高，那么打断旧的任务，重新开始，否则继续执行任务。

#### fiber&&FiberRoot&RootFiber

1. **FiberRoot**

- FiberRoot是整个React应用的起点

- FiberRoot包含应用挂载的目标节点（<div id='root'>root</div>）

- FiberRoot记录整个React应用 更新过程中的各种信息

1. **RootFiber**

RootFiber的本质是一个fiber，但是它是一个特殊的fiber，我们可以把它看成根fiber

- 它和FiberRoot的关系为

```
FiberRoot.current = RootFiber
RootFiber.stateNode = FiberRoot
```

- 它的return值为null（没有父节点）

1. **fiber**

- fiber对象与ReactElement一一对应，如下图，App组件

*|*![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=997dd37a5b5e6a12effd5ce0f1c8042e_8f118824ce50c961_boxcnaMfKpTmaQ8IJv6TOX9BH2b_YzfLAh8nX3avDFpcQ9JgRtw6Q7FAjD6r)

- fiber是一种新型的数据结构，记录类组件的各种状态（state和props）。**this**上的**state**和**props**是根据**Fiber**对象的**state**、**props**更新的

- fiber是改进版本的虚拟DOM(树形->链表）。每个ReactElement通过**props.children**与其他ReactElement连结起来

下图阐述了它们之间的关系

*|*![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=be7be4bdb9a609913da49798f33ec600_8f118824ce50c961_boxcnzdtFM9kPyjvn9Yi25xpaac_JlXYYHPhIgJCAXT7K4XyqfEUGOF14mLE)

- ReactElement只会把子节点（props.children）的第一个子节点当做child节点，其余的子节点（也就是第一个子节点的兄弟节点）都是从第一个子节点开始，依次**单向连接**至后一个兄弟节点

- 每个子节点都会指向父节点（红箭头），也就是fiber对象的return属性

**遍历过程**

- 找子节点（非上一次的）

- 找兄弟节点

- 返回父节点

#### 为什么要使用fiber呢？

在 React 15 版本的时候，组件更新需要递归遍历整个虚拟 DOM 树来判断需要更新的地方。这种递归无法中断，必须更新完所有组件才会停止。如果我们需要更新一些庞大的组件，在更新的过程中可能就会长时间阻塞主线程，从而造成用户的交互、动画的更新等等都不能及时响应。

React 的组件更新过程简而言之就是在持续调用函数的一个过程，这样的一个过程会形成一个虚拟的调用栈。假如我们控制这个调用栈的执行，把整个更新任务进行**拆解**，尽可能地将更新任务放到浏览器空闲的时候去执行，那么就能解决以上的问题。

Fiber 重新实现了 React 的核心算法，带来了杀手锏增量更新功能。它将整个更新任务拆**分为一个个小的任务**，并且能控制这些任务的执行。

#### FiberRoot的结构

```
function FiberRootNode(containerInfo, tag, hydrate) {
 this.tag = tag;
 this.current = null;
 this.containerInfo = containerInfo;
 this.pendingChildren = null;
 this.pingCache = null;
 this.finishedExpirationTime = NoWork;
 this.finishedWork = null;
 this.timeoutHandle = noTimeout;
 this.context = null;
 this.pendingContext = null;
 this.hydrate = hydrate;
 this.firstBatch = null;
 this.callbackNode = null;
 //跟root有关联的回调函数的时间
 this.callbackExpirationTime = NoWork;
 //存在root中，最旧的挂起时间
 this.firstPendingTime = NoWork;
 //存在root中，最新的挂起时间
 this.lastPendingTime = NoWork;
 //挂起的组件通知root再次渲染的时间
 this.pingTime = NoWork;
 if (enableSchedulerTracing) {
  this.interactionThreadID = unstable_getThreadID();
  this.memoizedInteractions = new Set();
  this.pendingInteractionMap = new Map();
 }
}
```

#### fiber对象的结构

以下是fiber对象中一些我认为比较重要的属性

```
export type Fiber = {
 //当前Fiber的状态（比如浏览器环境就是DOM节点）
 //不同类型的实例都会记录在stateNode上
 //比如DOM组件对应DOM节点实例
 //ClassComponent对应Class实例
 //FunctionComponent没有实例，所以stateNode值为null
 //state更新了或props更新了均会更新到stateNode上
 stateNode: any,
 // 形成链表结构
 return: Fiber | null,
 child: Fiber | null,
 sibling: Fiber | null,
 // 更新相关
 pendingProps: any,  // 新的 props
 memoizedProps: any,  // 旧的 props
 // 该fiber对应的组件，所产生的update，都会放在该队列中
 // 如果不存在，更新队列的默认值就为null
 updateQueue: UpdateQueue | null,
 memoizedState: any, // 旧的 state
 //新的变动带来的新的props，即nextProps
 
 // 调度相关
 expirationTime: ExpirationTime,//任务过期时间
 childExpirationTime: ExpirationTime,//子节点的任务过期时间
 // 在fiber树更新的过程中，每个fiber都有与其对应的alternate
 //我们称之为 current <==> workInProgress
 //在渲染完成后，会交换位置
 //doubleBuffer Fiber在更新后，不用再重新创建对象，
 // 而是复制自身，并且两者相互复用，用来提高性能
 alternate: Fiber | null,
|};
```

#### expirationTime

作用：当时间到了ExpirationTime的时候，如果某个update还未执行的话，React将会强制执行该update

- expirationTime越大优先级越高

- 高优先级update的expirationTime**间隔**是10ms，低优先级update的expirationTime间隔是25ms

- React让两个相近的update得到相同的expirationTime，目的就是让这两个update自动合并成一个Update，从而达到批量更新的目的

### 源码解析

介绍完上述概念，我们正式开始源码的解析，这是入口。

```
Component.prototype.setState = function(partialState, callback) {
 ...
 //*将setState事务放入队列中*
 this.updater.enqueueSetState(this, partialState, callback, 'setState');
};
```

我们先看看enqueueSetState，它的主要作用是将update入队，这个队列会在fiber内存储，如果fiber的镜像存在，镜像中也会存储一份队列。

它接受了三个参数

- inst 是this.setState的调用者，即类组件的实例

- payload是需要合并的state

- callback是合并state后的回调

```
enqueueSetState(inst, payload, callback) {
  //此处的this指向类组件
  //this本身有存储 fiber对象 的属性，叫 _reactInternalFiber
  const fiber = getInstance(inst);
  //计算当前时间
  const currentTime = requestCurrentTime();
  //根据当前计算fiber对象的过期时间
  const expirationTime = computeExpirationForFiber(currentTime, fiber);
  //创建update对象
  const update = createUpdate(expirationTime);
  //setState传进来的要更新的对象
  update.payload = payload;
  //callback就是setState({},()=>{})的回调函数
  if (callback !== undefined && callback !== null) {
   if (__DEV__) {
    warnOnInvalidCallback(callback, 'setState');
   }
   update.callback = callback;
  }
  //将被动影响的属性刷新一遍，这里不讲。
  flushPassiveEffects();
  //将需要更新的update入队
  enqueueUpdate(fiber, update);
  //根据过期时间进行任务调度
  scheduleWork(fiber, expirationTime);
 }
```

上面代码中的重点为enqueueUpdate和scheduleWork

我们先说enqueueUpdate，它的主要作用是获取队列并进行具体的入队操作。

在下面我会把它拆解成两部分进行讲解。

#### update入队：enqueueUpdate

##### 1. 获取队列

作用：获取一个存放update的单向链表，用next来串联update。

- queue1将来要保存的是fiber的update队列

- queue2将来要保存的是fiber的镜像的update队列

```
*export* function enqueueUpdate(fiber: Fiber, update: Update) {
 *// 获取 fiber 的镜像*
 const alternate = fiber.alternate;
 let queue1;
 let queue2;
 // updateQueue是一个单向链表，用next来串联update
 *// 第一次 render 的时候肯定是没有这个镜像的，所以进第一个条件*
 *if* (alternate === null) {
  *// 一开始也没这个 queue，所以需要创建一次*
  queue1 = fiber.updateQueue;
  queue2 = null;
  *if* (queue1 === null) {
   *// UpdateQueue 是一个链表组成的队列*
   queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState);
  }
 } *else* {
  *// There are two owners.*
  queue1 = fiber.updateQueue;
  queue2 = alternate.updateQueue;
  *// 以下就是在判断 q1、q2 存不存在了，不存在的话就赋值一遍*
  *// clone 的意义也是为了节省开销*
  *if* (queue1 === null) {
   *if* (queue2 === null) {
    queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState);
    queue2 = alternate.updateQueue = createUpdateQueue(
     alternate.memoizedState,
    );
   } *else* {
    queue1 = fiber.updateQueue = cloneUpdateQueue(queue2);
   }
  } *else* {
   *if* (queue2 === null) {
    *// Only one fiber has an update queue. Clone to create a new one.*
    queue2 = alternate.updateQueue = cloneUpdateQueue(queue1);
   } *else* {
    // 都不为空，不操作
   }
  }
 }
```

获取队列总结：

- queue1取的是fiber.updateQueue，queue2取的是alternate.updateQueue

1. 如果两者均为null，则调用createUpdateQueue()获取初始队列

1. 如果两者之一为null，则调用cloneUpdateQueue()从对方中获取队列

1. 如果两者都存在则不作处理

根据createUpdateQueue的代码，可以看出，用createUpdateQueue创建的是空队列

```
*export* function createUpdateQueue(baseState: State): UpdateQueue {
 const queue: UpdateQueue = {
  *//应用更新后的state*
  baseState,
  *// 链表头*
  firstUpdate: null,
  *// 链表尾*
  lastUpdate: null,
  firstCapturedUpdate: null,
  lastCapturedUpdate: null,
  firstEffect: null,
  lastEffect: null,
  firstCapturedEffect: null,
  lastCapturedEffect: null,
 };
 *return* queue;
}
```

而cloneUpdateQueue则是把新队列的队头和队尾指向老队列的队头队尾，两个队列是共享结构。

```
function cloneUpdateQueue(
 currentQueue: UpdateQueue,
): UpdateQueue {
 const queue: UpdateQueue = {
  baseState: currentQueue.baseState,
  firstUpdate: currentQueue.firstUpdate,
  lastUpdate: currentQueue.lastUpdate,
  *//* *TODO:* *With resuming, if we bail out and resuse the child tree, we should*
  *// keep these effects.*
  firstCapturedUpdate: null,
  lastCapturedUpdate: null,
  firstEffect: null,
  lastEffect: null,
  firstCapturedEffect: null,
  lastCapturedEffect: null,
 };
 *return* queue;
}
```

*|*![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=3c2301fd4b15f54d73515bce02e56342_8f118824ce50c961_boxcn86a2AxHh09UM5J9hXZinIc_h0HTQnvJvedgyEC3VgpafaDJ2Er4Kmbx)

经过上面的获取队列操作后，现在只有两种情况

- alternate不存在，一个（空）队列。

- alternate存在，两个（空）队列，共享结构。

##### 2. 入队操作

```
 *// 入队操作*
 *// 以下的代码很简单，熟悉链表的应该清楚链表添加一个节点的逻辑*
 *if* (queue2 === null || queue1 === queue2) {
  // 只有一个队列，alternate为null的情况。
  appendUpdateToQueue(queue1, update);
 } *else* {
  // 存在两个队列
  *if* (queue1.lastUpdate === null || queue2.lastUpdate === null) {
   *// 两个队列中存在空队列。*
   appendUpdateToQueue(queue1, update);
   appendUpdateToQueue(queue2, update);
  } *else* {
   // 不存在空队列
   appendUpdateToQueue(queue1, update);
   queue2.lastUpdate = update;
  }
 }
}
```

大家肯定存在疑惑，为什么存在空队列和不存在空队列的更新方式不一样。

这其实是因为共享结构，让我们看看appendUpdateToQueue的代码

```
function appendUpdateToQueue(
 queue: UpdateQueue,
 update: Update,
) {
 *// Append the update to the end of the list.*
 *if* (queue.lastUpdate === null) {
  *// Queue is empty*
  queue.firstUpdate = queue.lastUpdate = update;
 } *else* {
  queue.lastUpdate.next = update;
  queue.lastUpdate = update;
 }
}
```

*|*![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=0c9cec194271f05fdb49867eae582be6_8f118824ce50c961_boxcniEBXrkyBMGSZRvON2suOcb_IJyg18atZnOb7UiSIBLtZgXzS87P8Zm9)

- 大家可以看到，当两个队列不为空时，queue1和queue2是共享结构。

- 当执行appendUpdateToQueue时，queue1的尾节点发生了改变。

- 而queue2的lastUpdate指向的是queue1之前的lastUpdate，所以只需要将queue2的尾节点和queue1同步即可。

#### 任务调度：scheduleWork

scheduleWork的作用是任务调度

```
*export* const scheduleWork = scheduleUpdateOnFiber;
//scheduleWork
export function scheduleUpdateOnFiber(
 fiber: Fiber,
 expirationTime: ExpirationTime,
) {
 //判断是否是无限循环update
 checkForNestedUpdates();
 *// 获取FiberRoot对象*  
 const root = markUpdateTimeFromFiberToRoot(fiber, expirationTime);
 *// 如果root为空就说明没找到FiberRoot直接中断任务*  
 if (root === null) {
  warnAboutUpdateOnUnmountedFiberInDEV(fiber);
  return;
 }
 //找到rootFiber并遍历更新子节点的expirationTime
 const root = markUpdateTimeFromFiberToRoot(fiber, expirationTime);
 ...
 const priorityLevel = getCurrentPriorityLevel();
 //如果是同步任务的过期时间的话
 if (expirationTime === Sync) {
  //也就是第一次渲染前
  if (
   //如果还未渲染，update是未分批次的：
   // 执行环境是LegacyUnbatchedContext
   (executionContext & LegacyUnbatchedContext) !== NoContext &&
   //检测是不是未渲染过
   // 执行环境不是RenderContext或者CommitContext
   (executionContext & (RenderContext | CommitContext)) === NoContext
  ) {
   //跟踪这些update，并计数、检测它们是否会报错
   schedulePendingInteractions(root, expirationTime);
   //批量更新时，render是要保持同步的，但布局的更新要延迟到批量更新的末尾才执行
   //初始化root，调用workLoop进行循环单元更新，这部分我们不讲
   let callback = renderRoot(root, Sync, true);
   while (callback !== null) {
    callback = callback(true);
   }
  }
  //render后
  else {
   //立即执行调度任务
   scheduleCallbackForRoot(root, ImmediatePriority, Sync);
   //当前没有update时
   if (executionContext === NoContext) {
    //刷新同步任务队列
    flushSyncCallbackQueue();
   }
  }
 }
 //如果是异步任务的话
 else {
  scheduleCallbackForRoot(root, priorityLevel, expirationTime);
 }
}
```

总结，上述条件其实是根据exprirationTime对react进行判断

- 同步

- - 初次挂载，调用renderRoot进行循环渲染更新。

- - 非初次挂载，调用scheduleCallbackForRoot进行调度

- 异步

- - 获取优先级，调用scheduleCallbackForRoot进行调度

##### 1. 遍历更新过期时间

我们先来看看markUpdateTimeFromFiberToRoot，它的作用是找到rootFiber并遍历更新子节点的expirationTime。

```
//目标fiber会向上寻找rootFiber对象，在寻找的过程中会进行一些操作
function markUpdateTimeFromFiberToRoot(fiber, expirationTime) {
 //如果fiber对象的过期时间小于 expirationTime，则更新fiber对象的过期时间
 //也就是说，当前fiber的优先级是小于expirationTime的优先级的，现在要调高fiber的优先级
 if (fiber.expirationTime < expirationTime) {
  fiber.expirationTime = expirationTime;
 }
 //在enqueueUpdate()中有讲到，与fiber.current是映射关系
 let alternate = fiber.alternate;
 //同上
 if (alternate !== null && alternate.expirationTime < expirationTime) {
  alternate.expirationTime = expirationTime;
 }
 // Walk the parent path to the root and update the child expiration time.
 //向上遍历父节点，直到root节点，在遍历的过程中更新子节点的expirationTime
 //fiber的父节点
 let node = fiber.return;
 let root = null;
 //node=null,表示是没有父节点了，也就是到达了RootFiber，即最大父节点
 //HostRoot即树的顶端节点root
 if (node === null && fiber.tag === HostRoot) {
  //RootFiber的stateNode就是FiberRoot
  root = fiber.stateNode;
 }
 //没有到达FiberRoot的话，则进行循环
 else {
  while (node !== null) {
   alternate = node.alternate;
   //如果父节点的所有子节点中优先级最高的更新时间仍小于expirationTime的话
   //则提高优先级
   if (node.childExpirationTime < expirationTime) {
    //重新赋值
    node.childExpirationTime = expirationTime;
    //alternate是相对于fiber的另一个对象，也要进行更新
    if (
     alternate !== null &&
     alternate.childExpirationTime < expirationTime
    ) {
     alternate.childExpirationTime = expirationTime;
    }
   }
   //别看差了是对应(node.childExpirationTime < expirationTime)的if
   else if (
    alternate !== null &&
    alternate.childExpirationTime < expirationTime
   ) {
    alternate.childExpirationTime = expirationTime;
   }
   //如果找到顶端rootFiber，结束循环
   if (node.return === null && node.tag === HostRoot) {
    root = node.stateNode;
    break;
   }
   node = node.return;
  }
 }
 //更新该rootFiber的最旧、最新的挂起时间
 if (root !== null) {
  // Update the first and last pending expiration times in this root
  const firstPendingTime = root.firstPendingTime;
  if (expirationTime > firstPendingTime) {
   root.firstPendingTime = expirationTime;
  }
  const lastPendingTime = root.lastPendingTime;
  if (lastPendingTime === NoWork || expirationTime < lastPendingTime) {
   root.lastPendingTime = expirationTime;
  }
 }
 return root;
}
```

##### 2. 进行调度

scheduleCallbackForRoot

```
function scheduleCallbackForRoot(
 root: FiberRoot,
 priorityLevel: ReactPriorityLevel,
 expirationTime: ExpirationTime,
) {
 //获取root的回调过期时间
 const existingCallbackExpirationTime = root.callbackExpirationTime;
 //更新root的回调过期时间
 if (existingCallbackExpirationTime < expirationTime) {
  //当新的expirationTime比已存在的callback的expirationTime优先级更高的时候
  const existingCallbackNode = root.callbackNode;
  if (existingCallbackNode !== null) {
   //取消已存在的callback（打断）
   //将已存在的callback节点从链表中移除
   cancelCallback(existingCallbackNode);
  }
  //更新callbackExpirationTime
  root.callbackExpirationTime = expirationTime;
  //如果是同步任务
  if (expirationTime === Sync) {
   //在临时队列中同步被调度的callback
   root.callbackNode = scheduleSyncCallback(
    runRootCallback.bind(
     null,
     root,
     renderRoot.bind(null, root, expirationTime),
    ),
   );
  } else {
   let options = null;
   if (expirationTime !== Never) {
    //(Sync-2 - expirationTime) * 10-now()
    let timeout = expirationTimeToMs(expirationTime) - now();
    options = {timeout};
   }
   //callbackNode即经过处理包装的新task
   root.callbackNode = scheduleCallback(
    priorityLevel,
    //bind()的意思是绑定this，xx.bind(y)()这样才算执行
    runRootCallback.bind(
     null,
     root,
     renderRoot.bind(null, root, expirationTime),
    ),
    options,
   );
   ...
  }
 }
 ...
}
```

总结：

作用：比较新的回调的过期时间，和root的callback的过期时间。如果新的回调的过期时间更大，说明优先级更高，需要把root的callback取消掉。接着进行任务的调度

- 同步任务：调用scheduleSyncCallback，在临时队列中进行调度

- 异步任务：调用scheduleCallback，更新调度队列的状态。

###### 2.1 中断正在执行的任务

cancelCallback的作用为中断正在执行的任务。

```
*export* function cancelCallback(callbackNode: mixed) {
 *if* (callbackNode !== fakeCallbackNode) {
  Scheduler_cancelCallback(callbackNode);
 }
}
const {
 unstable_cancelCallback: Scheduler_cancelCallback,
} = Scheduler;
//从链表中移除task节点
function unstable_cancelCallback(task) {
 //获取callbackNode的next节点
 var next = task.next;
 //由于链表是双向循环链表，一旦next是null则证明该节点已不存在于链表中
 if (next === null) {
  // Already cancelled.
  return;
 }
 //自己等于自己，说明链表中就这一个callback节点
 //firstTask/firstDelayedTask应该是类似游标的概念，即正要执行的节点
 if (task === next) {
  //置为null，即删除callback节点
  //重置firstTask/firstDelayedTask
  if (task === firstTask) {
   firstTask = null;
  } else if (task === firstDelayedTask) {
   firstDelayedTask = null;
  }
 } else {
  //将firstTask/firstDelayedTask指向下一节点
  if (task === firstTask) {
   firstTask = next;
  } else if (task === firstDelayedTask) {
   firstDelayedTask = next;
  }
  var previous = task.previous;
  //熟悉的链表操作，删除已存在的callbackNode
  previous.next = next;
  next.previous = previous;
 }
 task.next = task.previous = null;
}
```

###### 2.2 同步调度

scheduleSyncCallback是进行同步调度的函数。

它的作用是：将调度任务入队，并返回入队后的临时队列。

```
//入队callback，并返回临时的队列
export function scheduleSyncCallback(callback: SchedulerCallback) {
 //在下次调度或调用 刷新同步回调队列 的时候刷新callback队列
 //如果同步队列为空的话
 if (syncQueue === null) {
  //初始化同步队列
  syncQueue = [callback];
  //并在下次调度的一开始就刷新队列
  immediateQueueCallbackNode = Scheduler_scheduleCallback(
   //赋予调度立即执行的高权限
   Scheduler_ImmediatePriority,
   flushSyncCallbackQueueImpl,
  );
 }
 //如果同步队列不为空的话，则将callback入队
 else {
  //在入队的时候，不必去调度callback，因为在创建队列的时候就已经调度了
  syncQueue.push(callback);
 }
 //返回临时队列
 return fakeCallbackNode;
}
```

- 当同步队列为空时，调用Scheduler_scheduleCallback()，将该callback入队并包装成newTask

- 当同步队列不为空，将该callback入队，在入队的时候不需要执行callback，因为在创造队列的时候已经调度了，该callback将会在之后的调度中被函数自动执行。

###### 2.3 异步调度

scheduleCallback是进行异步调度的函数

```
export function scheduleCallback(
 reactPriorityLevel: ReactPriorityLevel,
 callback: SchedulerCallback,
 options: SchedulerCallbackOptions | void | null,
) {
 //获取调度的优先级
 const priorityLevel = reactPriorityToSchedulerPriority(reactPriorityLevel);
 return Scheduler_scheduleCallback(priorityLevel, callback, options);
}
```

总结：

- 同步的时候，root.callbackNode被赋值fakeCallbackNode。并用Scheduler_scheduleCallback调度同步队列。

- 异步的时候，则直接执行Scheduler_scheduleCallback()，对callback进行包装处理并赋值给root.callbackNode

##### 3. 调度过程中用到的一些函数

###### 3.1 返回task

Scheduler_scheduleCallback的作用是，返回包装处理后的task

它的callback入参，有两种情况

- 同步：flushSyncCallbackQueueImpl

- 异步：runRootCallback

我们先不区分，把它们一律视作callback来进行看待

```
const {
 unstable_scheduleCallback: Scheduler_scheduleCallback,
} = Scheduler;
//返回经过包装处理的task
function unstable_scheduleCallback(priorityLevel, callback, options) {
 var currentTime = getCurrentTime();
 var startTime;
 var timeout;
 //更新 开始调度的时间startTime（默认是现在）和延迟更新时间 timeout（默认5s）
 //timeout是用来基于开始调度时间startTime计算过期时间expirationTime的
 if (typeof options === 'object' && options !== null) {
  var delay = options.delay;
  if (typeof delay === 'number' && delay > 0) {
   startTime = currentTime + delay;
  } else {
   startTime = currentTime;
  }
  timeout =
   typeof options.timeout === 'number'
    ? options.timeout
    : timeoutForPriorityLevel(priorityLevel);
 } else {
  timeout = timeoutForPriorityLevel(priorityLevel);
  startTime = currentTime;
 }
 //过期时间是当前时间+5s，也就是默认是5s后，react进行更新
 var expirationTime = startTime + timeout;
 //封装成新的任务
 var newTask = {
  callback,
  priorityLevel,
  startTime,
  expirationTime,
  next: null,
  previous: null,
 };
 //如果开始调度的时间已经错过了
 if (startTime > currentTime) {
  // This is a delayed task.
  //将延期的callback插入到延期队列中
  insertDelayedTask(newTask, startTime);
  ...  
 }
 //没有延期的话，则插入正常调度队列
 else {
  insertScheduledTask(newTask, expirationTime);
  ...
 }
 //返回经过包装处理的task
 return newTask;
}
```

###### 3.2 循环更新

runRootCallback是这样进行使用的：

```
runRootCallback.bind(
   null,
   root,
   renderRoot.bind(null, root, expirationTime),
),
```

那我们可以看出，callback其实就是renderRoot方法。renderRoot的主要作用是进行循环更新

可以看出，runRootCallback的作用就是调用renderRoot进行循环更新。并且在root调度结束后将root的callbackNode和callbackExpirationTime进行初始化。

```
function runRootCallback(root, callback, isSync) {
 const prevCallbackNode = root.callbackNode;
 let continuation = null;
 try {
  *// 其实执行的就是 renderRoot 方法*
   continuation = callback(isSync);
  *// 如果 renderRoot 有返回值，就把这个函数当作节点的回调组成一个新的节点任务，然后插入链表* 
   if (continuation !== null) {
    return runRootCallback.bind(null, root, continuation);
   } else {
     return null;
   }
  } finally {
  *// 这里其实是判断方法执行的过程中，root.callbackNode 是否发生了变化*
  *// 如果没有发生变化，说明没有更高优先级的任务进来*
  *// 任务执行完毕后，root 的调度任务也就结束了，将 callbackNode 和 callbackExpirationTime 重置。*   
  if (continuation === null && prevCallbackNode === root.callbackNode) {
   root.callbackNode = null;
   root.callbackExpirationTime = NoWork;
  }
 }}
```

###### 3.3 更新同步队列

flushSyncCallbackQueue的作用是更新同步队列的状态

```
//刷新同步任务队列
export function flushSyncCallbackQueue() {
 //如果即时节点存在则中断当前节点任务，从链表中移除task节点
 //这个即时节点就是上一次的同步更新队列
 if (immediateQueueCallbackNode !== null) {
  Scheduler_cancelCallback(immediateQueueCallbackNode);
 }
 //更新同步队列
 flushSyncCallbackQueueImpl();
}
```

接着看flushSyncCallbackQueueImpl

```
//更新同步队列
function flushSyncCallbackQueueImpl() {
 //如果同步队列未更新过并且同步队列不为空
 if (!isFlushingSyncQueue && syncQueue !== null) {
  //防止重复执行，相当于一把锁
  isFlushingSyncQueue = true;
  let i = 0;
  try {
   const isSync = true;
   const queue = syncQueue;
   //遍历同步队列，并更新刷新的状态isSync=true
   runWithPriority(ImmediatePriority, () => {
    for (; i < queue.length; i++) {
     let callback = queue[i];
     do {
      //之前的callback是用bind绑定的，且只传了两个参数，此处是传入第三个参数并执行
      //第三个参数是exprirationTime
      callback = callback(isSync);
     } while (callback !== null);
    }
   });
   //遍历结束后置为null
   syncQueue = null;
  } catch (error) {
   // If something throws, leave the remaining callbacks on the queue.
   if (syncQueue !== null) {
    syncQueue = syncQueue.slice(i + 1);
   }
   // Resume flushing in the next tick
   Scheduler_scheduleCallback(
    Scheduler_ImmediatePriority,
    flushSyncCallbackQueue,
   );
   throw error;
  } finally {
   isFlushingSyncQueue = false;
  }
 }
}
```

![img](https://bytedance.feishu.cn/space/api/box/stream/download/asynccode/?code=4554c5f1d1aa90b8f137964c2b418b8f_8f118824ce50c961_boxcnjWnwVVJEwW06tMVkkebHHh_SkjjActcLIKYWte7H1dUOKO2EqRapnas)

看到这里，我们在看看[场景一和场景二](https://bytedance.feishu.cn/docs/doccna6AmUR2bUCTxQGMH3AOhQd#Fgkyz4)，应该可以看出进行add时有无setTimeout包裹的区别了。

在场景一中，多个setState都被当作一个task，推入同步队列依次调度，而在react当中，相近时间，假如是低优先级，那就是25ms的update，计算出来的exprirationtime是相同的，他们就会被合成一个update，由同一个runRootCallback进行渲染。

而在场景二中，每个setState被当成一个task。每一次task都会执行runRootCallback进行渲染。

参考文章

[组件更新流程一（调度任务）](https://yuchengkai.cn/react/2019-07-29.html#文章相关资料)

[你真的理解setState吗？](https://juejin.im/post/6844903636749778958#heading-4)

[React源码解析之RootFiber](https://juejin.im/post/6844903919588474888)

[React源码解析之FiberRoot](https://juejin.im/post/6844903938068611079)

[React源码解析之ExpirationTime](https://juejin.im/post/6844903929004687368)

[React源码解析之Update和UpdateQueue](https://juejin.im/post/6844903924076396557)

[React源码 | 7. 任务调度：scheduleWork（上）](https://zhuanlan.zhihu.com/p/108849110)

[React源码 | 8. 任务调度：scheduleWork（下）](https://zhuanlan.zhihu.com/p/109063290)