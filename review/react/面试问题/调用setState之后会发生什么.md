
1. **创建更新，入队流程** ：

* 当 `setState` 或其他更新操作被调用时，会创建一个更新对象。
* 这个更新对象会被添加到对应组件的 `fiber` 节点的 `updateQueue` 中。

2. **从 fiber 节点的父节点开始遍历，到 rootFiber，给这些节点全部打上更新标记** ：

* React 会从当前 `fiber` 节点开始向上遍历，直到找到 `rootFiber`，并将这些节点标记为需要更新。

3. **异步和同步更新的处理** ：

* **异步更新** ：会给 `rootFiber` 的 `callbackNode` 绑定一个回调函数来触发后续的更新内容。
* **同步更新** ：直接触发后续的更新内容。

4. **创建 rootFiber 的镜像，即 `workInProgress` 树** ：

* React 会创建 `rootFiber` 的镜像，这个镜像被称为 `workInProgress` 树，用来表示当前更新操作的工作节点树。

5. **在 `workInProgress` 树执行所有的更新操作** ：

* **更新流程** ：
  * React 会遍历每个 `fiber` 的更新队列，根据过期时间来决定是否执行该更新。
  * 使用在渲染过期时间之前的 `update` 对旧的 `state` 进行更新，并将该 `update` 从链表中移除。
* **批处理模式** ：
  * 在批处理模式下，React 会使用全局同步队列一次性处理多个 `fiber` 的状态更新。

6. **React 会将 `workInProgress` 树与当前的 `rootFiber` 进行 diff** ：

* 在 `workInProgress` 树上执行所有更新操作后，React 会将 `workInProgress` 树与当前的 `rootFiber` 进行对比（diff），找出需要更新、插入或删除的节点。

7. **提交阶段** ：

* 在 diff 过程结束后，React 会进入提交阶段，将 `workInProgress` 树中的变化应用到真实的 DOM 中。
* 提交阶段的操作包括：
  * **插入新节点**
  * **更新已有节点**
  * **删除旧节点**

8. **替换 `rootFiber`** ：

* 最后，`workInProgress` 树会取代当前的 `rootFiber`，成为新的 `rootFiber`，表示最新的状态和结构。
