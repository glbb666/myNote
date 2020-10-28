# v-if

##### 初始渲染

初始值为 **false** 组件**不会**渲染，生命周期钩子**不会**执行，**v-if** 的渲染是**惰性**的。
初始值为 **true** 时，组件会进行渲染，并依次执行 beforeCreate,created,beforeMount,mounted 钩子。

##### 切换

false => true
依次执行 `beforeCreate,created,beforeMount,mounted `钩子。
true => false
依次执行 `beforeDestroy,destroyed `钩子。

# v-show

##### 渲染

无论初始状态，组件都会渲染，依次执行 `beforeCreate`,`created`,`beforeMount`,`mounted` 钩子，**v-show** 的渲染是**非惰性**的。

##### 切换

对生命周期钩子无影响，切换时组件始终保持在 `mounted` 钩子。

`v-if`有更高的切换开销，`v-show`有更高的初始渲染开销。因此，如果需要频繁切换，用`v-show`，如果运行时条件很少改变，用`v-if`