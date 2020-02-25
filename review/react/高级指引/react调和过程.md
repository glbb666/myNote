### react调和

`react`中的`render`方法，渲染的其实是`UI`的描述。`react`首先会将`UI`描述转换成虚拟`DOM`树，然后根据虚拟`DOM`树映射成真实的`DOM`节点。在更新阶段，`render`方法会重新执行，生成一颗新的虚拟`DOM`树，`react`会比较两颗虚拟`DOM`树的差别，根据差别来计算需要更新的`DOM`节点。在计算过程中，`react`引入了`diff`算法，计算生成将一棵树转换成另一棵树所需要进行的最小操作数。从而获取最高效的性能。

为了把算法的时间复杂度从`O(n3)`变为`O(n)`，提出了两个假设

- 两个不同类型的元素会产生不同的树
- 开发者可以通过key来暗示哪些元素在不同的渲染下能保持稳定

### diff算法

对比两棵树时，`react`首先比较两棵树的根节点。

- 根结点是不同类型的元素，第一个假设生效，停止继续比较， 直接拆卸原有树，建立新的树。
- 根结点是同一类型的元素，可能属性值发生了一些变化。举个🌰

```jsx
<div style={color='red',fontWeight:'bold'}>
	<Countter/>
</div>
<div style={color='green',fontWeight:'bold'}>
	<Countter/>
</div>
```

`div`元素未变，只是`style`属性值中的`color`属性值发生了变化，`react`中的`diff`算法能够精确的计算出只需要更新`color`属性值，对`fontWeight`也不会做更新的动作。

- 根结点是同一类型的组件元素，举个🌰

```jsx
<div>
	<Countter name='first'/>
</div>
<div>
	<Countter name='second'/>
</div>
```

由于只是属性值的变化，只会触发Counter更新的生命周期的执行

依次执行`getDerivedStateFromProps`=>`shouldComponentUpdate`=>`render`

`render`以后，`diff`算法会把`render`方法计算出来的新的`UI`片段和之前的进行递归的比较

- 对子节点进行递归

  在末尾增加新元素，发现新的树在根结点增加了列表节点，`react`就会在列表中插入一个新的列表元素<li>third</li>

```jsx
<ul>
	<li>first</li>
  <li>second</li>
</ul>

<ul>
	<li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

 	新增的元素在列表的第二个
```jsx
<ul>
	<li>first</li>
  <li>second</li>
</ul>
<ul>
	<li>first/</li>
  <li>third</li>
  <li>second</li>
</ul>
```
首先比较第一个元素<li>first</li>，发现没有发生变化，接着比较第二个元素，发现<li>second</li>变为了<li>third</li>，于是`react`销毁第二个元素，创建<li>third</li>，接着`react`发现第三个元素<li>second</li>，于是创建<li>second</li>并插入`ul`列表中。这种情况下`react`的更新存在较大性能开销

其实我们知道，<li>second</li>只是发生了位置的移动，元素内容并没有发生变化，新增的只有<li>third</li>元素，所以存在性能优化的空间。为了尽可能优化，引入了key机制，`react`通过检测`key`来发现列表元素是新增还是只移动位置，从而实现性能优化。

让我们引入key属性

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

因为检测到元素位置的移动，所以`react`会首先将<li key='second'>second</li>从位置2移动到位置3，接着新增一个元素<li key='third'>third</li>。

### 权衡

- 内容切换时，尽量保证组件类型的稳定。
- key值稳定，并且应该在兄弟元素间唯一。对于不稳定的key，比如通过render方法生成的key，会导致许多组件的实例和`DOM`结点被不必要的重新创建，同时会导致性能的下降，并且会导致子组件元素当中的状态发生丢失 ，这是因为react在列表当中通过key唯一标识元素，如果key不稳定，每次执行render都会生成新的key，对于react来说，这是一个新的元素， 在更新的时候就会执行先销毁再创建的动作，带来不必要的性能开销。 

 