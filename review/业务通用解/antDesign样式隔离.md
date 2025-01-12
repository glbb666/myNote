# 背景

开发了一个sdk，被业务层项目引用。

sdk的antd版本是3，业务层项目的antd版本是4。

Ant Design 4 的一些组件样式可能与 Ant Design 3 的样式存在冲突，导致样式覆盖或者错乱。

业务层的antd版本和sdk层的不一致，出现了作用域污染的问题。

# 解法

## 初级解法

一开始的解法是在sdk项目中，给打包好的全局css样式外层加一层scope。

但是问题在于，antd有的文件是直接挂载在body下的。比如Modal、Popover、Tooltip、Dropdown等。

添加scope只能影响scope内的组件，但是无法影响挂载在body下的。

## 第一次升级

对于body下的样式。通过高阶组件的方式，在app.component挂载全局组件的时候，修改它外层overlayClassName。

```js
const withOverlayScope = (WrappedComponent, scope) => {
  return (props) => {
    const { overlayClassName, ...restProps } = props;
    return (
      <WrappedComponent
        {...restProps}
        overlayClassName={`${scope} ${overlayClassName || ''}`}
      />
    );
  };
};

... 

const init = (app)=>{
	app.component('Modal', withOverlayScope(Modal));
}
```

接着在postcss中，通过extracy-css-rules这个plugin拦截antd在body下的组件的全局样式并克隆保存在单独的文件中。

当body下组件的样式都扫描过之后，所有的组件样式就都被加了一层前缀，即被处理为作用域样式。

### 缺点：

但是这样做的缺点很明显

- postcss对css树的遍历拦截属于运行时的操作，在开发时进行修改，需要重新计算css样式，会造成卡顿，并带来一定的性能损失。
- 插件会克隆匹配的规则并添加前缀，这会增加输出的 CSS 体积，并引入额外的解析和处理时间。
- 将修改后的样式保存为单独的文件带来的I/O 操作的时间成本

## 第二次升级

所以决定选用ConfigProvider的方案。

即在组件最外层加一个ConfigProvider的前缀，属于编译时的方案。不仅改善了性能，而且只需要全局匹配，修改关于antd的前缀，改为带有自定义前缀的类名即可。

1. **统一前缀管理：**
   * 通过 `ConfigProvider` 设置 `prefixCls`，使得所有组件的类名都有统一的作用域前缀，避免了类名冲突。
2. **编译时生效：**
   * 不需要在运行时修改 CSS 样式，避免了不必要的样式重计算，提升了性能。
3. **覆盖 body 下的组件：**
   * 对于挂载在 `body` 下的组件（如 `Modal`、`Dropdown` 等），`ConfigProvider` 的 `prefixCls` 也会生效，因为这些组件的类名继承了 `prefixCls`。
4. **无额外样式文件生成：**
   * 不需要额外保存和管理拦截后的 CSS 文件，减少了构建复杂度和 I/O 成本。
