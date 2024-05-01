### 什么是 `useLayoutEffect`

类似于 `componentWillMount`（在 React 17 及以前）和 `componentWillUpdate`，以及 `getSnapshotBeforeUpdate`。
它在所有 DOM 变更之后立即**同步执行**，会阻塞浏览器的绘制进程。这意味着你可以在渲染发生前同步修改 DOM。

### 为什么用 `useLayoutEffect`

为了让开发者可以在浏览器绘制之前执行紧急的、必须同步完成的操作。

useLayoutEffect 一般在需要同步执行DOM操作或者计算布局的时候使用，可以避免浏览器绘制出现闪烁的问题。
它在渲染阶段之后，浏览器执行绘制之前执行，因为一些元素在渲染之后才能确定位置，如果在useEffect之后再修改，其实会引起重绘和重排，在useLayoutEffect中进行可以避免这类的问题。

只有在对布局和视觉一致性有同步需求的场景下，才使用 `useLayoutEffect`。

常见的场景包括：

* 测量DOM元素的大小或者位置（刚刚渲染完成，还没绘制，能够获取到最新，最准确的布局信息）
* 同步更新DOM元素的样式
* 在DOM更新后执行一些需要立即反馈的操作，例如focus一个输入框
* 需要在组件首次渲染后立即执行DOM操作的情况

总的来说，如果你的操作需要在浏览器绘制之前完成，那么就应该使用useLayoutEffect。
