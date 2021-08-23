### 什么是redux-saga

`redux-saga`是一个库，他以`redux`中间件的形式存在，目的是更好的管理`redux`应用程序中的副作用（side effect)。

`redux-saga`主要用来处理异步的`actions`。

简而言之，就是把`action->reducer`的过程变为`action->中间件->reducer`。

如果按照原始的`redux`工作流程，当组件中产生一个`action`后会直接触发`reducer`修改`state`，`reducer`又是一个纯函数，也就是不能在`reducer`中进行异步操作。

而往往实际中，组件中发生的`action`后，在进入`reducer`之前需要完成一个异步任务，比如发送`ajax`请求后拿到数据后，再进入`reducer`。这个时候急需一个中间件来处理这种业务场景，目前最优雅的处理方式自然就是`redux-saga`。

### 一些概念

在学习`redux-saga`之前，我们先了解一些概念。

#### effect

什么是副作用呢？

映射在 Javascript 程序中，Side Effects 主要指的就是：**异步网络请求**、**本地读取 localStorage/Cookie** 等外界操作。

#### saga与generator

一个`saga`文件中会有一至多个`generator`函数来处理`effect`。

#### Task

一个 task 就像是一个在后台运行的进程。在基于 redux-saga 的应用程序中，可以同时运行多个 task。通过 `fork` 函数来创建 task：

```javascript
function* saga() {
  ...
  const task = yield fork(otherSaga, ...args)
  ...
}
```

### Effect 创建器

#### `call(fn, ...args)`：调用函数（阻塞）

**call是什么**

创建一个 Effect 描述信息，用来命令 `middleware` 以参数 `args` 调用函数 `fn` 。

- `fn: Function` - 一个 `Generator `函数, 也可以是一个返回 `Promise `或任意其它值的普通函数。
- `args: Array<any>` - 传递给 `fn` 的参数数组。

**为什么使用call**

我们从 `Generator` 里` yield Effect`以表达 `Saga` 逻辑。 

其实我们也可以这样写

```javascript
const users = yield Api.fetch('/users')
```

`Api.fetch('/users')` 触发了一个 `AJAX `请求并返回一个` Promise`，`Promise` 会` resolve `请求的响应， 这个` AJAX `请求将立即执行。看起来简单又地道，但...

假设我们想测试上面的` generator`：

```javascript
const iterator = fetchProducts()
assert.deepEqual(iterator.next().value, ??) // 我们期望得到什么？我们怎么可能预先知道结果，根据结果的一致性来判断请求没有发送错误呢？
```

所以我们必须 *模拟（mock）* `Api.fetch` 函数。 也就是说，我们需要将真实的函数替换为一个假的，这个假的函数并不会真的发送 AJAX 请求而只会检查**是否用正确的参数调用了 `Api.fetch`**（在我们的情况里，正确的参数是 `'/users'`）。

相比于在 Generator 中直接调用异步函数，**我们可以仅仅 yield 一条描述函数调用的信息**。也就是说，我们将简单地 yield 一个看起来像下面这样的对象：

```javascript
// Effect -> 调用 Api.fetch 函数并传递 `./products` 作为参数
{
  CALL: {
    fn: Api.fetch,
    args: ['./users']  
  }
}
```

`Generator` 将会` yield` 包含 *指令* 的文本对象（`plain Objects`），`redux-saga` middleware 将确保执行这些指令并将指令的结果回馈给 `Generator`。 这样的话，在测试` Generator` 时，所有我们需要做的就是，将 `yield` 后的对象作一个简单的 `deepEqual` 来检查它是否 `yield `了我们期望的指令。

出于这样的原因，`redux-saga` 提供了一个不一样的方式来执行异步调用。

```javascript
import { call } from 'redux-saga/effects'

function* fetchProducts() {
  const users = yield call(Api.fetch, '/users')
  // ...
}
```


我们使用了 `call(fn, ...args)` 这个函数。**与前面的例子不同的是，现在我们不立即执行异步调用，相反，`call` 创建了一条描述结果的信息**。 就像在 Redux 里你使用 action 创建器，创建一个将被 Store 执行的、描述 action 的纯文本对象。 `call` 创建一个纯文本对象描述函数调用。`redux-saga` middleware 确保执行函数调用并在响应被` resolve` 时恢复` generator`。

这让你能容易地测试` Generator`，就算它在` Redux` 环境之外。因为 `call` 只是一个返回纯文本对象的函数而已。

```javascript
import { call } from 'redux-saga/effects'
import Api from '...'

const iterator = fetchUsers()

// expects a call instruction
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, '/users'),
  "fetchProducts should yield an Effect call(Api.fetch, './users')"
)
```

`yield`后如果是一个`promise`，因为`promise`不能进行比较，为了**方便测试**，在调用函数的时候都写成`call`的形式。`yield` 后的表达式 `call(fetch, '/users')` 被传递给 `next` 的调用者。

现在我们不需要模拟任何东西了，一个简单的相等测试就足够了。

这些 *声明式调用（declarative calls）* 的优势是，我们可以通过简单地遍历 Generator 并在 yield 后的成功的值上面做一个 `deepEqual` 测试， 就能测试 Saga 中所有的逻辑。这是一个真正的好处，因为复杂的异步操作都不再是黑盒，你可以详细地测试操作逻辑，不管它有多么复杂。

`call` 同样支持调用对象方法，你可以使用以下形式，为调用的函数提供一个 `this` 上下文：

```javascript
yield call([obj, obj.method], arg1, arg2, ...) // 如同 obj.method(arg1, arg2 ...)
```

**怎么使用call**

- 先后执行多个任务

```javascript
import { call } from 'redux-saga/effects'

const users = yield call(fetch, '/users'),
      repos = yield call(fetch, '/repos')
```

- 同步执行多个任务

当我们需要 `yield` 一个包含 effects 的数组， generator 会被阻塞**直到所有的 effects 都执行完毕**，或者当一个 effect 被拒绝。一旦其中任何一个任务被拒绝，并行的 Effect 将会被拒绝。在这种情况中，所有其他的 Effect 将被自动取消。

```javascript
import { call } from 'redux-saga/effects'

const [users, repos] = yield [
  call(fetch, '/users'),
  call(fetch, '/repos')
]
```

- 获取执行最快的任务 

在 `race` Effect 中。所有参与 race 的任务，除了优胜者（译注：最先完成的任务），其他任务都会被取消。

1. 可用作超时处理

```javascript
import { race, call, put } from 'redux-saga/effects'
import { delay } from 'redux-saga'

function* fetchPostsWithTimeout() {
  //这里的 posts和timeout 只有执行快的那个能取得值
  const {posts, timeout} = yield race({
    posts: call(fetchApi, '/posts'),
    timeout: call(delay, 1000)
  })

  if (posts)
    put({type: 'POSTS_RECEIVED', posts})
  else
    put({type: 'TIMEOUT_ERROR'})
}	
```

2. 自动取消失败的 `Effects`。

```javascript
import { race, take, call } from 'redux-saga/effects'

function* backgroundTask() {
  while (true) { ... }
}

function* watchStartBackgroundTask() {
  while (true) {
    yield take('START_BACKGROUND_TASK')
    //当 CANCEL_TASK 被发起，race 将自动取消 backgroundTask，并在 backgroundTask 中抛出取消错误。
    yield race({
      task: call(backgroundTask),
      cancel: take('CANCEL_TASK')
    })
  }
}
```

#### put`(action)`：发起action（非阻塞）

**put是什么**

创建一个 Effect 描述信息，用来命令 middleware 向 Store 发起一个 action。 这个 effect 是非阻塞型的，并且所有向下游抛出的错误（例如在 reducer 中），都不会冒泡回到 saga 当中。

**为什么使用put**

在前面的例子上更进一步，假设每次保存之后，我们想发起一些 action 通知 Store 数据获取成功了（目前我们先忽略失败的情况）。

我们可以找出一些方法来传递 Store 的 `dispatch` 函数到 Generator。然后 Generator 可以在接收到获取的响应之后调用它。

```javascript
//...

function* fetchUsers(dispatch)
  const products = yield call(Api.fetch, '/users')
  dispatch({ type: 'USERS_RECEIVED', users })
}
```

然而，该解决方案与我们在上一节中看到的从` Generator` 内部直接调用函数，有着相同的缺点。如果我们想要测试 `fetchUsers` 接收到` AJAX `响应之后执行` dispatch`， 我们还需要模拟 `dispatch` 函数。

相反，我们需要同样的声明式的解决方案。只需创建一个对象来指示` middleware `我们需要发起一些` action`，然后让 `middleware `执行真实的` dispatch`。 这种方式我们就可以同样的方式测试` Generator `的 `dispatch`：只需检查` yield `后的` Effect`，并确保它包含正确的指令。

**怎么使用put**

redux-saga 为此提供了另外一个函数 `put`，这个函数用于创建 dispatch Effect。

```javascript
import { call, put } from 'redux-saga/effects'
//...

function* fetchProducts() {
  const products = yield call(Api.fetch, '/users')
  // 创建并 yield 一个 dispatch Effect
  yield put({ type: 'USERS_RECEIVED', users })
}
```

#### `take(pattern)`（阻塞）

**take是什么**

创建一个 Effect 描述信息，用来命令 middleware 在 Store 上等待指定的 action。 在发起与 `pattern` 匹配的 action 之前，Generator 将暂停。

**为什么要使用take**

`take` Effect 让我们可以在一个集中的地方更好地去描述一个非常规的流程。

**怎么使用take**

- 如果它是一个函数，那么将匹配 `pattern(action)` 为 true 的 action。（例如，`take(action => action.entities)` 将匹配哪些 `entities` 字段为真的 action）

  > 注意: 如果 pattern 函数上定义了 `toString`，`action.type` 将改用 `pattern.toString` 来测试。这个设定在你使用 action 创建函数库（如 redux-act 或 redux-actions）时非常有用。

- 如果它是一个字符串，那么将匹配 `action.type === pattern` 的 action。

  ```
  take('LOGOUT') 
  ```

- 如果它是一个数组，那么数组中的每一项都适用于上述规则 —— 因此它是支持字符串与函数混用的。不过，最常见的用例还属纯字符串数组，其结果是用 `action.type` 与数组中的每一项相对比。（例如，`take([INCREMENT, DECREMENT])` 将匹配 `INCREMENT` 或 `DECREMENT` 类型的 action）

  ```
  take(['LOGOUT', 'LOGIN_ERROR'])
  ```

- 监听所有的`action`：空参数或 `'*'` 

  ```
  take('*')，take()
  ```
  
  middleware 提供了一个特殊的 action —— `END`。如果你发起 END action，则无论哪种 pattern，只要是被 take Effect 阻塞的 Sage 都会被终止。假如被终止的 Saga 下仍有分叉（forked）任务还在运行，那么它在终止任务前，会先等待其所有子任务均被终止。

```javascript
import { select, take } from 'redux-saga/effects'

function* watchAndLog() {
  while (true) {
    const action = yield take('*')
    const state = yield select()

    console.log('action', action)
    console.log('state after', state)
  }
}
```

我们先把上面的代码`copy`下来

```javascript
import { takeEvery } from 'redux-saga/effects'

// FETCH_USERS
function* fetchUsers(action) { ... }

// CREATE_USER
function* createUser(action) { ... }

// 同时使用它们
export default function* rootSaga() {
  yield takeEvery('FETCH_USERS', fetchUsers)
  yield takeEvery('CREATE_USER', createUser)
}
```

当两到多个`action`互相之间没有逻辑关系时，我们可以使用`takeEvery`。

但是，当`action`之间存在逻辑关系后，使用`takeEvery`就会出现问题。比如登陆和登出。

由于`takeEvery`对`action`的分开监管降低了可读性，程序员必须阅读多个处理函数的`takeEvery`源代码并建立起它们之间的逻辑关系。

我们可以用`take`把它改成这样，这样两个代码

```javascript
export default function* loginFlow() {
  while(true) {
    yield take('LOGIN')
    // ... perform the login logic
    yield take('LOGOUT')
    // ... perform the logout logic
  }
}

export default function* rootSaga() {
  yield* userSaga()
}
```

优点

1. `saga`主动拉取`action`，可以控制监听的开始和结束
2. 用同步的风格描述控制流，提高了可读性。

#### `fork(fn, ...args)`：调用函数（无阻塞）

**fork是什么**

创建一个 Effect 描述信息，用来命令 `middleware` 以 **非阻塞调用** 的形式执行 `fn`。

- `fn: Function` - 一个` Generator` 函数，或返回` Promise` 的普通函数
- `args: Array<any>` - 传递给 `fn` 的参数数组。

返回一个 [Task](https://redux-saga-in-chinese.js.org/docs/api/#task) 对象。

**注意事项**

`fork` 类似于 `call`，可以用来调用普通函数和 `Generator` 函数。不过，`fork` 的调用是非阻塞的，`Generator` 不会在等待 `fn` 返回结果的时候被 `middleware `暂停；恰恰相反地，它在 `fn` 被调用时便会立即恢复执行。

`fork`，以及 `race`，都是用于管理` Saga` 间并发的中心化` Effect`。

`yield fork(fn ...args)` 的结果是一个 [Task](https://redux-saga-in-chinese.js.org/docs/api/#task) 对象 —— 一个具备着某些实用方法及属性的对象。

所有分叉任务（forked tasks）都会被附加到它们的父级任务身上。当父级任务终止其自身命令的执行，它会在返回之前等待所有分叉任务终止。

来自于子级任务的错误会自动地冒泡到它们的父级任务。如果有任何分叉任务引发了一个未被捕获的错误，那么父级任务会因该子级错误而中断，并且父级的整个执行树（即分叉任务 + 如果还在运行的由父体扮演的 **主任务**）也会被取消。

一个分叉任务的取消，将自动地取消所有还在执行的分叉任务。另外，这个被取消的任务在被阻塞时所处的当前 Effect（若有的话）也会被取消。

如果一个分叉任务以同步的形式失败（即在它执行完成后且未开始做异步操作前立即失败），则不会返回 Task，并且父级任务将被尽快地中断（由于父子任务均并行地运行，父级任务将在收到子级任务失败的通知时中断）。

若要创建 **被分离的（detached）** 分叉，请使用 `spawn` 代替。

**为什么要使用fork**

`call`调用时会发生阻塞。 当我们不想错过`call`下面的`take`等待的`action`，想让异步调用和等待并行发生时，我们可以用`fork`取代`call`。

当 `fork` 被调用时，它会在后台启动 task 并返回 task 对象。

```javascript
import { take, put, call, fork, cancel } from 'redux-saga/effects'

// ...

function* loginFlow() {
  while(true) {
    const {user, password} = yield take('LOGIN_REQUEST')
    // fork return a Task object
    const task = yield fork(authorize, user, password)
    const action = yield take(['LOGOUT', 'LOGIN_ERROR'])
    if(action.type === 'LOGOUT')
      yield cancel(task)
    yield call(Api.clearItem('token'))
  }
}
```

##### fork Model

在 `redux-saga` 的世界里，你可以使用 2 个 Effects 在后台动态地 fork task

- `fork` 用来创建 *attached forks*
- `spawn` 用来创建 *detached forks*

**Attached forks (using `fork`)**

Attached forks 通过以下规则继续附加在它们的 parent

**结论**

- Saga 只会在这之后终止：
  - 它终止了自己的指令
  - 所有附加的 forks 本身被终止

假如我们有以下情形：

```js
import { delay } from 'redux-saga'
import { fork, call, put } from 'redux-saga/effects'
import api from './somewhere/api' // app specific
import { receiveData } from './somewhere/actions' // app specific

function* fetchAll() {
  const task1 = yield fork(fetchResource, 'users')
  const task2 = yield fork(fetchResource, 'comments')
  yield call(delay, 1000)
}

function* fetchResource(resource) {
  const {data} = yield call(api.fetch, resource)
  yield put(receiveData(data))
}

function* main() {
  yield call(fetchAll)
}
```

`call(fetchAll)` 将在这之后终止:

- `fetchAll` 本身终止了, 意味着这 3 个 effect 都会被执行. 由于 `fork` effects 是非阻塞的, task 将被阻塞在 `call(delay, 1000)`
- 这 2 个被 fork 的 task 终止, 意思是在 fetch 所需的资源之后放入对应的 `receiveData` action

所以整个 task 将阻塞直到一个 1000 毫秒的 delay 被传送，并且 task1 和 task2 完成了他们的任务。

比方说，1000 毫秒的 delay 和这两个 task 还沒有完成，然后 fetchAll 在终止整个 task 之前，将一直等待直到所有被 fork 的 task 完成。

细心的读者可能会注意到 `fetchAll` saga 可以使用平行 Effect 来重写。

```js
function* fetchAll() {
  yield all([
    call(fetchResource, 'users'),     // task1
    call(fetchResource, 'comments'),  // task2,
    call(delay, 1000)
  ])
}
```

事实上，被附加的 fork 与平行 Effect 共享相同的语意：

- 在平行情況下我们执行 task
- 在所有被 launch 的 task 终止后，parent 将会终止

这也适用于其他语意（错误和取消传播）。你可以简单地把它考虑作为一个*动态平行* Effect，来理解附加 fork 的行为。

**Error 传播**

按照同样的比喻，让我们来详细的检查在平行的 Effect 中是如何处理错误的

例如，假设我们有这么一个 Effect：

```js
yield all([
  call(fetchResource, 'users'),
  call(fetchResource, 'comments'),
  call(delay, 1000)
])
```

一旦其中一个子 Effect 失敗，上方的 Effect 就会失敗。 此外，未捕获的错误将会造成平行 Effect 取消所有其他 pending 中的 Effect。例如，如果 `call(fetchResource, 'users')` 发出了一个未捕获的错误，平行 Effect 将会取消其他两个 task（如果它们依然在 pending），然后从失败的调用中，以错误终止它本身。

类似于被 attach 的 forks，Saga 会在以下情况马上终止：

- 指令的 main body 拋出了一个错误
- 一个未捕获的错误通过其中一个被 attach 的 forks 抛出

所以在先前的例子中：

```js
//... imports

function* fetchAll() {
  const task1 = yield fork(fetchResource, 'users')
  const task2 = yield fork(fetchResource, 'comments')
  yield call(delay, 1000)
}

function* fetchResource(resource) {
  const {data} = yield call(api.fetch, resource)
  yield put(receiveData(data))
}

function* main() {
  try {
    yield call(fetchAll)
  } catch (e) {
    // handle fetchAll errors
  }
}
```

如果在这样的情況下，例如 `fetchAll` 被阻塞在 `call(delay, 1000)` Effect，假如 `task1` 失败了，然后整个 `fetchAll` task 将会因此失敗

- 取消所有其他在 pending 的 task。包含：
  - *main task*（`fetchAll` 的本身）：取消的意思是，取消目前的 `call(delay, 1000)` 的 Effect
  - 其他被 fork 的 task 仍然在 pending。例如我们例子中的 `task2`。
- 在 `main` 的 `catch` 内将会捕获由 `call(fetchAll)` 抛出的错误

注意，因为我们使用了一个阻塞的 call，所以我们只能从 `main` 內部 catch `call(fetchAll)` 的错误，而且我们不能直接从 `fetchAll` 捕获错误。这是首要的原则，**你不能从被 fork 的 task 捕获错误**。在一个被附加的 fork 中的错误将会导致被 fork 的 parent 被终止（就像没有方法在一个平行 Effect *内*捕捉错误一样，只能从外部通过被阻塞的平行 Effect）。

#### `spawn(fn, ...args)`

与 `fork(fn, ...args)` 相同，但创建的是 **被分离的** 任务。被分离的任务与其父级任务保持独立，并像顶级任务般工作。父级任务不会在返回之前等待被分离的任务终止，并且所有可能影响父级或被分离的任务的事件都是完全独立的（错误、取消）。

#### `cancel(task)`：取消函数调用

**cancel是什么**

创建一个 Effect 描述信息，用来命令 middleware 取消之前的一个`fork`任务。

- `task: Task` - 由之前 `fork` 指令返回的 [Task](https://redux-saga-in-chinese.js.org/docs/api/#task) 对象

**注意事项**

若要取消正在运行的任务，middleware 将调用底层 Generator 对象上的 `return`，这将取消任务中的当前 Effect，并跳转至 finally 区块（若有定义的话）。

在 finally 区块中。

- 你可以执行任何清理逻辑，或发起某些 action 来保持 store 处于一致状态(例如，当 ajax 请求被取消时，将 spinner 的状态重置为 false)。
- 你可以在yield cancelled()条件下编写只有在取消时执行的逻辑。

`cancel` 是一个非阻塞的` Effect`。也就是说，执行 `cancel` 的 Saga 会在发起取消动作后立即恢复执行。

对于返回 `Promise` 结果的函数，你可以通过给 `promise` 附加一个 `[CANCEL]` 来插入自己的取消逻辑。

下述例子演示了如何将取消逻辑附加到` Promise` 结果上:



```javascript
import { take, call, put, cancelled } from 'redux-saga/effects'
import Api from '...'

function* authorize(user, password) {
  try {
    const token = yield call(Api.authorize, user, password)
    yield put({type: 'LOGIN_SUCCESS', token})
    yield call(Api.storeItem, {token})
    return token
  } catch(error) {
    yield put({type: 'LOGIN_ERROR', error})
  } finally {
    if (yield cancelled()) {
      // ... put special cancellation handling code here
    }
  }
}
```

取消正在执行的任务，也将同时取消被阻塞在当前 Effect 中的任务。

举个例子，假设在应用程序生命周期的某个时刻，还有挂起的（未完成的）调用链：

```javascript
function* main() {
  const task = yield fork(subtask)
  ...
  // later
  yield cancel(task)
}

function* subtask() {
  ...
  yield call(subtask2) // currently blocked on this call
  ...
}

function* subtask2() {
  ...
  yield call(someApi) // currently blocked on this call
  ...
}
```

**取消信息会向下传播到子 saga**。当取消任务时，`middleware` 还会取消当前 `Effect`（当前阻塞` task `的 `Effect`，即一些`ajax`请求等）。如果当前` Effect `调用了另一个` saga`，那么该` saga` 也会被取消。当取消` saga` 时，所有 **附加分叉（attached forks）**（用 `yield fork()` 分叉出的 saga）将被取消。这意味着取消动作会有效地影响属于取消任务的整个执行树。

⚠️注意：`yield cancel(task)` **不会等待被取消的任务完成（即执行其 catch 区块）**。一旦取消，任务通常应尽快完成它的清理逻辑然后返回。

#### `cancelled()`

创建一个 Effect，用来命令 middleware 返回该 generator 是否已经被取消。通常你会在 finally 区块中使用这个 Effect 来运行取消时专用的代码。

```javascript
function* saga() {
  try {
    // ...
  } finally {
    if (yield cancelled()) {
      // 只应在取消时执行的逻辑
    }
    // 应在所有情况下都执行的逻辑（例如关闭一个 channel）
  }
}
```

##### 自动取消

除了手动取消任务，还有一些情况的取消是自动触发的。

1. 在 `race` Effect 中。所有参与 race 的任务，除了优胜者（译注：最先完成的任务），其他任务都会被取消。
2. 并行的 Effect (`yield [...]`)。一旦其中任何一个任务被拒绝，并行的 Effect 将会被拒绝（受 `Promise.all` 启发）。在这种情况中，所有其他的 Effect 将被自动取消。

### 辅助函数

#### 什么是辅助函数

> 辅助函数是构建在Effect构建器以上的高级API。

#### `takeEvery(pattern, saga, ...args)`（阻塞）

**takeEvery是什么**

在发起（`dispatch`）到` Store` 并且匹配 `pattern` 的每一个` action` 上派生一个 `saga`。

- `pattern: String | Array | Function` - 有关更多信息，请参见 [`take(pattern)`](https://redux-saga-in-chinese.js.org/docs/api/#takepattern) 的文档
- `saga: Function` - 一个 `Generator `函数
- `args: Array<any>` - 传递给启动任务的参数。`takeEvery` 会把当前的 action 追加到参数列表中。（即 action 将是 `saga` 的最后一个参数）

**takeEvery如何使用**

把监听的动作换为通配符`*`。在每次`action `被匹配时一遍又一遍地被调用。**无法控制何时被调用**，**无法控制何时停止监听**。

```javascript
import { select, takeEvery } from 'redux-saga/effects'

function* watchAndLog() {
  yield takeEvery('*', function* logger(action) {
    const state = yield select()

    console.log('action', action)
    console.log('state after', state)
  })
}
```

**注意事项**

`takeEvery` 是一个使用 `take` 和 `fork` 构建的高级 API。下面演示了这个辅助函数是如何由低级 Effect 实现的：

```javascript
const takeEvery = (patternOrChannel, saga, ...args) => fork(function*() {
  while (true) {
    const action = yield take(patternOrChannel)
    yield fork(saga, ...args.concat(action))
  }
})
```

`takeEvery` 允许处理并发的 action（译注：即同时触发相同的 action）。在上面的例子里，当发起一个 `USER_REQUESTED` action 时，即使前一个 `fetchUser` 任务还未处理结束，也将会启动一个新的 `fetchUser` 任务。 （举个例子，用户以极快的速度连续点击一个 `Load User` 按钮 2 次，即使第一个触发的 `fetchUser` 任务还未结束，第二次点击依然会发起一个 `USER_REQUESTED` action。）

`takeEvery` 不会对多个任务的响应进行排序，并且不保证任务将会以它们启动的顺序结束。如果要对响应进行排序，可以关注以下的 `takeLatest`。

#### `takeLatest(pattern, saga, ...args)`（阻塞）

**takeLatest是什么**

每当一个 `action` 被发起到 `Store`，并且匹配 `pattern` 时，则 `takeLatest` 将会在后台启动一个新的 `saga` 任务。 如果此前已经有一个 `saga` 任务启动了（在当前 `action` 之前发起的最后一个` action`），并且仍在执行中，那么这个任务将被取消。

- `pattern: String | Array | Function` - 有关更多信息，请参见 [`take(pattern)`](https://redux-saga-in-chinese.js.org/docs/api/#takepattern) 的文档
- `saga: Function` - 一个 Generator 函数
- `args: Array<any>` - 传递给启动任务的参数。`takeLatest` 会把当前的 action 追加到参数列表中。（即 action 将是 `saga` 的最后一个参数）

**takeLatest如何使用**

在下面的示例中，我们创建了一个简单的任务 `fetchUser`。我们在每次 `USER_REQUESTED` action 被发起时，使用 `takeLatest` 来启动一个新的 `fetchUser` 任务。 由于 `takeLatest` 取消了所有之前启动且未完成的任务，这样便可以保证：即使用户以极快的速度连续多次触发 `USER_REQUESTED` action，我们都只会以最后的一个结束。

```javascript
import { takeLatest } from `redux-saga/effects`

function* fetchUser(action) {
  ...
}

function* watchLastFetchUser() {
  yield takeLatest('USER_REQUESTED', fetchUser)
}
```

**注意事项**

`takeLatest` 是一个使用 `take` 和 `fork` 构建的高级 API。下面演示了这个辅助函数是如何由低级 Effect 实现的：

```javascript
const takeLatest = (patternOrChannel, saga, ...args) => fork(function*() {
  let lastTask
  while (true) {
    const action = yield take(patternOrChannel)
    if (lastTask) {
      yield cancel(lastTask) // 如果任务已经结束，cancel 则是空操作
    }
    lastTask = yield fork(saga, ...args.concat(action))
  }
})
```

### Effect组合器

#### `race(effects)`

**race是什么**

创建一个 Effect 描述信息，用来命令 middleware 在多个 Effect 间运行 *竞赛（Race）*（与 [`Promise.race([...\])`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise/race) 的行为类似）。

`effects: Object` - 一个 {label: effect, ...} 形式的字典对象

**race如何使用**

1. 可以用`race`进行超时处理

```javascript
import { race, call, put } from 'redux-saga/effects'
import { delay } from 'redux-saga'

function* fetchPostsWithTimeout() {
  const {posts, timeout} = yield race({
    posts: call(fetchApi, '/posts'),
    timeout: call(delay, 1000)
  })

  if (posts)
    put({type: 'POSTS_RECEIVED', posts})
  else
    put({type: 'TIMEOUT_ERROR'})
}
```

2. 自动取消那些失败的 Effects。

假设我们有 2 个 UI 按钮：

- 第一个用于在后台启动一个任务，这个任务运行在一个无限循环的 `while(true)` 中（例如：每 x 秒钟从服务器上同步一些数据）
- 一旦该后台任务启动了，我们启用第二个按钮，这个按钮用于取消该任务。

```javascript
import { race, take, call } from 'redux-saga/effects'

function* backgroundTask() {
  while (true) { ... }
}

function* watchStartBackgroundTask() {
  while (true) {
    yield take('START_BACKGROUND_TASK')
    yield race({
      task: call(backgroundTask),
      cancel: take('CANCEL_TASK')
    })
  }
}
```

在 `CANCEL_TASK` action 被发起的情况下，`race` Effect 将自动取消 `backgroundTask`，并在 `backgroundTask` 中抛出一个取消错误。

### 错误处理

1. 使用 `try/catch` 语法在 Saga 中捕获错误

```javascript
import Api from './path/to/api'
import { call, put } from 'redux-saga/effects'

// ...

function* fetchProducts() {
  try {
    const products = yield call(Api.fetch, '/products')
    yield put({ type: 'PRODUCTS_RECEIVED', products })
  }
  catch(error) {
    yield put({ type: 'PRODUCTS_REQUEST_FAILED', error })
  }
}
```

2. 捕捉 `Promise `的拒绝操作，并将它们映射到一个错误字段对象。

```javascript
import Api from './path/to/api'
import { call, put } from 'redux-saga/effects'

function fetchProductsApi() {
  return Api.fetch('/products')
    .then(response => ({ response }))
    .catch(error => ({ error }))
}

function* fetchProducts() {
  const { response, error } = yield call(fetchProductsApi)
  if (response)
    yield put({ type: 'PRODUCTS_RECEIVED', products: response })
  else
    yield put({ type: 'PRODUCTS_REQUEST_FAILED', error })
}
```

### 测试

```javascript
//saga.spec.js
import { call, put } from 'redux-saga/effects'
import Api from '...'

const iterator = fetchProducts()

// 期望一个 call 指令
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, '/products'),
  "fetchProducts should yield an Effect call(Api.fetch, './products')"
)

// 创建一个假的响应对象
const products = {}

// 期望一个 dispatch 指令
assert.deepEqual(
  iterator.next(products).value,
  put({ type: 'PRODUCTS_RECEIVED', products }),
  "fetchProducts should yield an Effect put({ type: 'PRODUCTS_RECEIVED', products })"
)

// 创建一个模拟的 error 对象
const error = {}

// 期望一个 dispatch 指令
assert.deepEqual(
  iterator.throw(error).value,
  put({ type: 'PRODUCTS_REQUEST_FAILED', error }),
  "fetchProducts should yield an Effect put({ type: 'PRODUCTS_REQUEST_FAILED', error })"
)
```

当测试出现分叉的时候，可以调用`clone`方法

```javascript
import { put, take } from 'redux-saga/effects';
import { cloneableGenerator } from 'redux-saga/utils';

test('doStuffThenChangeColor', assert => {
  const gen = cloneableGenerator(doStuffThenChangeColor)();
  //前面都是一样的
  gen.next(); // DO_STUFF
  gen.next(); // CHOOSE_NUMBER
	
  
  //判断奇偶的时候出现了分叉
  assert.test('user choose an even number', a => {
    const clone = gen.clone();
    a.deepEqual(
      clone.next(chooseNumber(2)).value,
      put(changeUI('red')),
      'should change the color to red'
    );

    a.equal(
      clone.next().done,
      true,
      'it should be done'
    );

    a.end();
  });

  assert.test('user choose an odd number', a => {
    const clone = gen.clone();
    a.deepEqual(
      clone.next(chooseNumber(3)).value,
      put(changeUI('blue')),
      'should change the color to blue'
    );

    a.equal(
      clone.next().done,
      true,
      'it should be done'
    );

    a.end();
  });
});

```

为了运行上面的测试代码，我们需要运行：

```sh
$ yarn test
```

### 在项目中使用

安装

```sh
$ yarn add react-saga
```

引入

```javascript
//main.js
import { createStore,applyMiddleware } from 'redux'//applyMiddleware把中间件应用起来，可以放多个
import createSagaMiddleware from 'redux-saga'
import rootSaga from './sagas'
//使用 `redux-saga` 中间件将 Saga 与 Redux Store 建立连接。
const sagaMiddleware = createSagaMiddleware();
const store = createStore(
  rootReducer,
  applyMiddleware(sagaMiddleware)
)
sagaMiddleware.run(rootSaga)
```

有多个`saga`同时启动，需要进行整合，在`saga.js`模块中引入`rootSaga`

```javascript
//sagas.js
import { delay } from 'redux-saga'
import { put, takeEvery,all } from 'redux-saga/effects'
export function* helloSaga() {
  console.log('Hello Sagas!');
}
export function* incrementAsync() {
  ...
}

function* watchIncrementAsync() {
	...
}
export default function* rootSaga() {
  yield all([
    helloSaga(),
    watchIncrementAsync()
  ])
}
```

### 
