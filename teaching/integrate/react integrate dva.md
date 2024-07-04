# React技术3-集成dva

本章节为第三部分，教程中的代码也是沿着前两篇文档的代码往下继续编写的，如有需要请小伙伴们移步到：

[React技术1-集成react-router](https://juejin.im/post/5ec8a3aae51d4578885c8f22)

[React技术2-集成 less 和 css-module](https://juejin.im/post/5ec8f0e46fb9a047f0125f24)

对于在 pc 项目中，可能对于数据流管理的依赖还不是那么的深，毕竟一个页面就是一个模块，小伙伴们完全可以在 `state` 里进行数据的保存。

但是对于移动端的H5 开发来说，数据流管理就显得尤为重要，可能单单一个业务流程就涉及到好多页面，如果通过路由传递数据的话，就显得特别的笨重，所以非常需要一个能够储存数据的技术仓库。

`dva` 是一个基于 `redux` 和 `redux-saga` 的数据流方案, 所以相对于 `redux` 来说使用起来就更加的方便和简单，或者说，`dva` 可以算是目前较为主流的一项数据流管理的技术了。

那么现在开始我们的 `dva` 库的引入(demo 是跟着前两部分的文章继续往下写的，如果参考，请前往文章头部查看前两部分的内容)：

## 一、引入 dva

在项目中打开命令行：

```bash
yarn add dva
```

## 二、`src/index.js` 文件编写

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import Router from './router';
import * as serviceWorker from './serviceWorker';

ReactDOM.render(
  <React.StrictMode>
    <Router />
  </React.StrictMode>,
  document.getElementById('root')
);

serviceWorker.unregister();
```

这是我们前两部分用到的代码，现在不需要啦，我们删掉上面的代码使用 dva 重新编写，并引入 `router` 路由。

```js
import dva from 'dva';
import './index.css';

// 初始化 dva
const app = dva();

// 这里可以导入一些 dva 插件，如果不需要就不用写
// app.use({});

// 这里是导入 model 后续我们新建一个dva model 模块就要这里引入才能使用
app.model(require('./models/home').default);

// 导入路由 类似之前代码里的 <Router />
app.router(require('./router').default);

app.start('#root');
```

## 三、编写 model 模版

首先在 `/src` 下新建一个 `models` 文件夹，文件夹下在新建一个 `home.js` 用来存放数据。

**这里有个小小的备注：**

正常情况下，笔者会默认为一个页面新建一个 `model` 或者为一个模块的几个页面去新建一个 `model`。看小伙伴们的习惯。

笔者会把所有的 `model` 统一放在一个文件夹下，当然有些人会放在每个页面的文件夹下，这个也是看个人习惯。

```js
export default {
  namespace: 'home',

  state: {
    value: 'this is models-home',
  },

  effects: {},

  subscriptions: {
    setup({ dispatch, history }) { }
  },

  reducers: {
    save(state, action) {
      return { ...state, ...action.payload };
    },
  }
}
```

上面就是一个最简单的 `model` 的基本模版。笔者默认读者会使用 `dva`,所以这里会略微过一下，如果需要学习 `dva`， 请小伙伴们移步[官网](https://dvajs.com/) 或者参考笔者写的 [dva.js 入门级教程](https://blog.csdn.net/weixin_42278979/article/details/90146080)

- namespace: 为该模版的命名，一般会与页面名称同名
- state: 该模版下的数据
- effects: 异步请求的方法，下文会细说
- subscriptions： 监听事件
- reducers： 同步请求的方法， 下文会细说

## 四、在页面中引入 model

打开 `src/home/index.js`

```js
// 部分代码有省略
import { connect } from 'dva';

const HomePage = ({ home }) => {

  const change = () => {};

  return (
    <div>
      <p>this is home</p>
      <p>{home.value}</p>
      <Button onClick={change}>change value</Button>
    </div>
  )
}

// 如果需要引入多个 model 用 ',' 隔开即可
export default connect(({ home }) => ({ home }))(HomePage);
```

通过 `props.home` 就可以获取到 `model-home` 下的 `state` 数据。

![img](../../gitImg/react/dva1.jpg)

## 五、改变 state 数据

现在有个需求通过点击按钮改变 `state` 下 `value` 的值。

```diff
// 代码有省略
- const HomePage = ({ home }) => {
+ const HomePage = ({ home, dispatch }) => {
  const change = () => {
+    dispatch({
+      type: 'home/save',
+      payload: {
+        value: 'has changed'
+      }
+    })
  };
}
```

因为不需要异步的操作，所以我们直接调用 `reducer` 下的 `save` 函数，来修改 `state` 值。`payload` 用来传递参数。

那么小伙伴们点击下按钮看看是否生效了呢。

## 六、effects 异步请求

如果现在我们需要通过异步请求获取数据并将数据存储在 `state` 中，这时我们就不能调用 `reducer` 下的请求。

在 `effects` 下新写个方法来调用：

```js
effects: {
  *query({ payload }, {  call, put }) {
    // 这里可以通过 request 或者 call 异步请求数据
    ...

    // 然后将请求到的数据调用 reducer 下的 save 方法保存到 state 中。
    yield put({
      type: 'save',
      payload: {
        value: 'effect changed'
      }
    });
  }
}
```

我们修改下 `src/home/index.js` 的代码：

```diff
  const change = () => {
    dispatch({
+      type: 'home/query',
-      type: 'home/home',
-      payload: {
-        value: 'has changed'
-      }
    })
  };
```

好了现在在点击下按钮看看页面会发生什么变化吧(`effects` 如果需要传递参数的话也是通过 `payload` 进行传递)。

正常的流程的是 `effects` 异步请求数据，请求到的数据结构一般无法直接保存至 `state`

而是将请求到的数据传递至 `reducer` 下新建的方法进行数据结构的改变再保存成  `state` 值。

但是笔者为了方便都会直接在 `effects` 下将数据处理好，直接调用 `save` 方法进行保存。

## 七、获取其他model 的数据

这里布置一个小小的作业，写个 `index` 的 model 用在 `index` 页面上。

现在我们在 `home.js` 下取到 `model/index.js` 下的 `state` 数据.

```js
// 代码有省略
effects: {
  *query({ payload }, {  call, put, select }) {
    // 这样就可以取到 index 的 state 数据了
    const indexModel = yield select(_ => _.index);
    console.log(indexModel);
    ...
  }
}
```

编写不易，点点关注给个赞呀～














