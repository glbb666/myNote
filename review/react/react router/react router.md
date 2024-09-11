# 是什么

`react-router` 是一个用于在 React 中实现客户端路由的库。它允许你根据 URL 的变化来渲染不同的组件，从而创建单页应用程序（SPA）。

# 为什么

它可以实现类似于多页面应用（MPA）的导航效果，但在实际操作中，页面是单页的，用户体验更流畅。

# 核心概念

**`Router` 组件** ：这是路由系统的顶层组件。常见的 `Router` 实现包括 `BrowserRouter` 和 `HashRouter`。`BrowserRouter` 使用 HTML5 的历史 API 来管理路由，而 `HashRouter` 使用 URL 的哈希部分来进行路由。

**`Route` 组件** ：定义了 URL 路径与组件之间的映射关系。当 URL 匹配到指定的路径时，`Route` 组件会渲染相应的组件。

```jsx
<Route path="/about" component={AboutPage} />

```

**`Switch` 组件** ：用于包裹多个 `Route` 组件，确保在多个路径匹配时只渲染第一个匹配的路由。它在路由列表中进行“开关”匹配。

```jsx
<Switch>
  <Route exact path="/" component={HomePage} />
  <Route path="/about" component={AboutPage} />
  <Route component={NotFoundPage} />
</Switch>

```

**`Link` 组件** ：用于创建导航链接。与 HTML 的 `<a>` 标签类似，但它不会导致页面重新加载，而是通过 `react-router` 管理路由。

```js
<Link to="/about">About Us</Link>

```

**`useHistory`、`useLocation` 和 `useParams` 钩子** ：这些钩子允许你在函数组件中访问路由信息和执行路由操作。

* `useHistory`：访问历史对象，用于程序化导航。
* `useLocation`：获取当前的 URL 信息。
* `useParams`：获取路由参数。

```js
import { useHistory, useLocation, useParams } from 'react-router-dom';

function MyComponent() {
  const history = useHistory();
  const location = useLocation();
  const params = useParams();

  return (
    <div>
      <button onClick={() => history.push('/home')}>Go Home</button>
      <p>Current location: {location.pathname}</p>
      <p>Route parameter: {params.id}</p>
    </div>
  );
}

```

# 怎么做

一个简单的 `react-router` 使用示例可能如下：

```js
import React from 'react';
import { BrowserRouter as Router, Route, Switch, Link } from 'react-router-dom';

function HomePage() {
  return <h2>Home Page</h2>;
}

function AboutPage() {
  return <h2>About Page</h2>;
}

function NotFoundPage() {
  return <h2>404 Page Not Found</h2>;
}

function App() {
  return (
    <Router>
      <nav>
        <ul>
          <li><Link to="/">Home</Link></li>
          <li><Link to="/about">About</Link></li>
        </ul>
      </nav>
      <Switch>
        <Route exact path="/" component={HomePage} />
        <Route path="/about" component={AboutPage} />
        <Route component={NotFoundPage} />
      </Switch>
    </Router>
  );
}

export default App;

```

这个示例定义了一个基本的路由配置，包括主页、关于页面和一个 404 页面。导航链接用于切换路由，`Switch` 和 `Route` 组件用于根据 URL 渲染相应的页面。
