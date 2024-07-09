## 背景

React 的 diff 算法（也称为协调算法）在进行比较和更新时，主要是在 `rootFiber` 和 `workInProgress` 树之间进行的。React 的 diff 算法主要遵循以下原则和步骤：

### Diff 算法的基本原则

**树分层比较** ：

* React 只会对同一层级的节点进行比较，不会跨层级比较。
* 当更新的时候节点是从fiber到rootFiber打上更新标记进行更新，所以也仅仅只对于这些fiber生成镜像节点，接着进行替换。

**类型比较**

* 如果两个节点类型相同，React 会保留节点，只更新属性。
* 如果两个节点类型不同，React 会认为它们完全不同，删除旧的fiber节点及其子树，并插入新的节点及其子树。

**列表的比较** ：

* React 使用 key 属性来优化列表节点的比较，确保节点的稳定性和性能。

#### key

让我们引入key属性

#### 例子

```jsx
<ul>
  <li key='first'>first</li>
  <li key='second'>second</li>
</ul>
<ul>
  <li key='first'>first</li>
  <li key='third'>third</li>
  <li key='second'>second</li>
</ul>
```

当没有key时：递归比较时发现第二个元素，发现 `<li>`second `</li>`变为了 `<li>`third `</li>`，于是 `react`销毁第二个元素，创建 `<li>`third `</li>`，接着 `react`发现第三个元素 `<li>`second `</li>`，于是创建 `<li>`second `</li>`并插入 `ul`列表中。这种情况下 `react`的更新存在较大性能开销

当有key值时：因为检测到元素位置的移动，所以 `react`会首先将 `<li key='second'>`second `</li>`从位置2移动到位置3，接着新增一个元素 `<li key='third'>`third `</li>`。

### 权衡

- 内容切换时，尽量保证组件类型的稳定。
- key值稳定，并且应该在兄弟元素间唯一。对于不稳定的key，比如通过render方法生成的key，会导致许多组件的实例和 `DOM`结点被不必要的重新创建，同时会导致性能的下降，并且会导致子组件元素当中的状态发生丢失 ，这是因为react在列表当中通过key唯一标识元素，如果key不稳定，每次执行render都会生成新的key，对于react来说，这是一个新的元素， 在更新的时候就会执行先销毁再创建的动作，带来不必要的性能开销。
