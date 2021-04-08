# Histories

`React Router` 是建立在 `history` 之上的。 `history` 监听浏览器地址栏的变化， 并解析这个 `URL` 转化为 `location` 对象， 然后 `router` 使用它匹配到路由，最后正确地渲染对应的组件。

常用的 `history` 有三种形式

- `browserHistory`：是使用 `React Router` 的应用推荐的 `history`。它使用浏览器中的 `History API` 用于处理 `URL`，创建一个像`example.com/some/path`这样真实的 `URL` 。但是它**需要服务器的配合**。

- `hashHistory`：`Hash history` 使用 `URL` 中的 `hash`（`#`）部分去创建形如 `example.com/#/some/path` 的路由。不需要服务器任何配置就可以运行，但是不推荐在实际线上环境中使用。

- `createMemoryHistory`：`Memory history` 不会在地址栏被操作或读取。这就解释了我们是如何实现服务器渲染的。同时它也非常适合测试和其他的渲染环境（像 `React Native` ）。

  和另外两种`history`的一点不同是你必须创建它，这种方式便于测试。

  ```js
  const history = createMemoryHistory(location)
  ```

## 实现示例

```jsx
import React from 'react'
import { render } from 'react-dom'
import { browserHistory, Router, Route, IndexRoute } from 'react-router'

import App from '../components/App'
import Home from '../components/Home'
import About from '../components/About'
import Features from '../components/Features'

render(
  <Router history={browserHistory}>
    <Route path='/' component={App}>
      <IndexRoute component={Home} />
      <Route path='about' component={About} />
      <Route path='features' component={Features} />
    </Route>
  </Router>,
  document.getElementById('app')
)
```