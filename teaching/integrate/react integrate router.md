# React技术1-集成react router

## 前沿

前端各种架构技术的飞速发展，百家齐放，大部分框架都自动帮助我们集成了一些技术，方便我们使用。

很容易盲目的往期走，却忘记了这些基本的操作是如何集成的。

## create-react-app 快速启动一个项目(简单介绍)

使用 `create-react-app reactdemo` 快速新建一个名为 `reactdemo` 的项目，进入项目目录启动。

```
cd reactdemo

yarn start
```

## react-router

router 是每个项目必不可少的点。我们不能仅仅只用 `Link` 标签去进行跳转页面，很多流程下我们是通过执行完某部分代码后才进行的页面跳转。

所以第一部分，我们试着引入路由。

### 1、新建页面

首先页面跳转就要先有页面，我们在 `src` 下新建两个文件夹 `index` 和 `home`， 并且在每个文件夹下都新建 `index.jsx`、`index.css` 文件。

![img](../../gitImg/react/router1.jpg)

### 2、导入路由

打开命令行工具，我们需要导入几个包 `react-router` 和 `react-router-dom`:

```bash
yarn add react-router

yarn add react-router-dom
```

或者可以使用 `npm`（后续不再展示）

```bash
npm i react-router --save

npm i react-router-dom --save
```

### 3、路由编写

在 `src` 下新建 `router.js` 文件

```js
import React from 'react';
import { HashRouter, Route, Switch } from 'react-router-dom';

import Index from './index/index';
import Home from './home';

const Router = ({ history }) => (
  <HashRouter history={history}>
    <Switch>
      <Route exact path="/" component={Index} />
      <Route exact path="/home" component={Home} />
    </Switch>
  </HashRouter>
);
export default Router;
```

- `exact`: 代表严格匹配模式，若置为 `false` 时，根据路由匹配所有组件， 如 `/` 匹配 `/`、`/home`、`/home/menu`， 所以请将此带上
- `path`: 为路径名称

后续每新增一个页面，都要在这里进行路由配置。

这时我们就可以删除 `App.js`、`App.css`、`App.test.js` 三个文件了，没什么用，留着感觉占位置。

路由文件写好后，我们要在 `src/index.js` 文件引入它：

```diff
...
- import App from './App';
+ import Router from './router';
import * as serviceWorker from './serviceWorker';

ReactDOM.render(
  <React.StrictMode>
-    <App />
+    <Router />
  </React.StrictMode>,
  document.getElementById('root')
);
...
```

这里我们来验证下效果

打开 `http://localhost:3000/#/home` 和 `http://localhost:3000/#/`

![img](../../gitImg/react/router2.jpg)

### 4、实现在事件中的路由跳转

这里来个小插曲，我们快速的导入一下 `antd` 库，这样按钮比较好看～

```bash
yarn add antd
```

在 `src/index.css` 第一行加上如下代码：

```css
@import '~antd/dist/antd.css';
```

好了，我们继续打开 `src/index/index.js` 文件。

```js
import React from 'react';
import { Button } from 'antd';
import './index.css';

const IndexPage = ({ history }) => {

  const goToHome = () => {
    history.push({
      pathname: '/home'
    });
  }

  return <div>
    <p>this is index</p>
    <Button onClick={goToHome}>跳转到 Home</Button>

  </div>
}

export default IndexPage;
```

一个完美的路由跳转事件就完成了， 小伙伴自己尝试一下呢。

剩下最后一个问题：怎么通过路由进行传参呢？

```js
const goToHome = () => {
  history.push({
    pathname: '/home',
    state: {
      name: 'this is state'
    }
  });
}
```

可以通过 `state` 进行传参，我们在 `home` 页面可以 `console.log(props)` 看下数据是否有过来。

![img](../../gitImg/react/router3.jpg)

可以看到我们能够通过 `props.location.state` 取到页面传递过来的数据。

如果我们希望通过`url` 传值呢？ 

```js
const goToHome = () => {
  history.push({
    pathname: '/home',
    search: '?name=search'
  });
}
```

当点击完跳转按钮，我们就可以看到路由变成 `http://localhost:3000/#/home?name=search`, 并且通过 `console.log(props.location.search)` 能够取到参数。

最后小补充：

如果你是使用 `umi` 项目，可以使用一下代码进行路由跳转和参数传递

```js
history.push({
  pathname: '/home',
  query: {
    name: 'this is name',
  }
})
```

在 `/home` 页面能够使用 `props.location.query` 取道对象参数，小伙伴们就不需要再去截取数据拉。







