# Redux 入门级教程文档

在这里我将用最浅显易懂的语言给小伙伴们解释一下 `Redux`, 并且一步步的教大家如何使用 `Redux`。

## 一、概念说明

### 1、什么是 Redux, 为什么要使用 Redux

官网说明：Redux 是 JavaScript 状态容器，提供可预测化的状态管理。

什么意思呢？就是说随着页面复杂化，页面上的很多数据需要保存，若路由跳转后，前一个页面的数据被销毁会导致数据的丢失。

若通过 url 传递，十分的麻烦，且要将每个数据传入组件中，写起来也是十分的无脑浪费时间。

### 2、为什么要使用 Redux

如果有个仓库用来保存整个项目中需要被保留的数据，且组件中可以直接拿到仓库的数据。那么这些数据将时刻保持一致，且清晰。

## 二、环境搭建

### 1、安装
首先我们先用 `create-react-app` 创建一个 `react` 项目。

`create-react-app redux-text`,

安装 `redux`：

`npm install --save redux` 或者 `yarn add redux`

`npm install --save react-redux` 或者 `yarn add react-redux`

### 2、概念简单说明

这里有几个概念需要大家先稍微知道以下：

`store`: 我们存放整个项目的一个仓库，一个项目只能有一个仓库，用来存放需要保存的数据。

`state`: `store` 中保存的数据。

`action`: 用户在页面上的操作是修改不到 `store` 里的 `state`。页面上的 `view` 要发生变化，就要在页面上通过 `action`，去告诉 `state` 你要变化了。**注意**：这里的 `action` 只是告诉仓库的数据要变化了，而不是去变化数据！！

`reducer`: 接收 `action` 的请求，执行修改 `store` 里的 `state` 的变化。

### 3、目录结构

接下来我们来创建几个项目中需要用到的文件夹和文件，以下是目录结构：

```
.
├── .gitignore
├── README.md
├── src                         
│   └── action                  	
│     ├── oneAction.js		
│     ├── twoAction.js		
│   └── components			
│   └── container
│			├── pageOne
|       ├──index.js
│   └── reducer	
│     ├── oneReducer.js
│     ├── twoReducer.js
│     ├── index.js
│   └── App.css		
│   └── App.js		
│   └── .....       
├── node_modules
├── package.json                
├── public            
```

## 三、搭建数据源

创建两个数据源：

`src/reducer/oneReducer.js` 和 `src/reducer/twoReducer.js`

```js
const oneReducer = (
  state = {
    name: '航航',
    address: '福州'
  },
  action
) => {
  switch(action.type) {
    case 'setName': 
      return {
        ...state,
        name: action.payload.name
      }
    default: 
    return state;
  }
}

export default oneReducer;
```

这里的纯函数 `oneReducer(state, action)` 的两个参数，分别代表了 `store` 下, 名为 `oneReducer` 的 `state` 和 `action`。

`state`: 存放了 `oneReducer` 下的所有数据源和初始值。

`action`: 通过不同的 `action.type` 去执行不同的操作，修改 `state` 数据

**注意**：每次修改 `state`, `redux` 并不是去修改原来的 `state`，而是返回一个新的 `state`, 用新的 `state`, 去替换旧的 `state`。

当 `action.type` 为 `setName` 时，我们先将原先的 `state` 解构出来，并给 `name` 附上新值。

`twoReducer.js` 也是如此：

```js
const twoReducer = (
  state = {
    age: 10,
  },
  action
) => {
  switch(action.type) {
    case 'setAge':
      return {
        ...state,
        age: 11,
      }
    default: 
    return state;
  }
}

export default twoReducer;
```

最后一步，整合所有的 `reducer`

```js
import { combineReducers } from 'redux';

import OneReducer from './oneReducer';
import TwoReducer from './twoReducer';

export default combineReducers({
  OneReducer,
  TwoReducer,
})
```

`combineReducers`: 将所有的子 `reducer` 函数组成对象，提供一个新的 `Reducer` 函数

## 四、编写页面

我们写一个简单的页面，将两个数据源的数据都展示出来。

### 1、创建仓库

打开 `src/index.js`

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import * as serviceWorker from './serviceWorker';
import PageOne from './container/pageOne';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import Reducer from './reducer';

const store = createStore(Reducer);

ReactDOM.render(
  <Provider store={store}>
    <PageOne/>
  </Provider>
, document.getElementById('root'));

serviceWorker.unregister();
```

`createStore`: 创建仓库，用来存放 `recuder` 下的所有数据源。

`Provider`: redux 提供的一个组件，将 `store` 传递给其自组件。简单说就是在整个项目的最外层包上一个组件，放进一个 `store`, 这样就将 `store` 绑定进了项目中。

### 2、展示页面

那么现在我们来专注于页面 `/src/container/pageOne/index.js` 的编写：

```js
import React from 'react';
import { connect } from 'react-redux';

class PageOne extends React.Component {
  render() {
    const { oneReducer, twoReducer } = this.props;
    const { name = '', address = '' } = oneReducer;
    const { age = 0 } = twoReducer;
    console.log(oneReducer, twoReducer);
    return (
      <div>
        <div>name: {name}</div>
        <div>address: {address}</div>
        <div>age: {age}</div>
      </div>
    )
  }
}

const mapStateToProps = state => {
  const { oneReducer, twoReducer } = state;
  return {
    oneReducer,
    twoReducer,
  }
}

export default connect(mapStateToProps)(PageOne);
```

细细讲解以下上面的知识点哈：

`mapStateToProps`: 获取 `store` 仓库下的数据源，这里可以打印以下 `state`, 看下输出。

`connect`: 由 React Redux 库提供的方法，将当前 Redux store state 映射到展示组件 props 中。

`connect` 做了性能优化，可以避免很多不必要的重复渲染，比如，当 `state` 数据变动时，不必编写 `shouldComponentUpdate` 方法来更新展示数据。

那么至此，仓库中的数据我们就可以通过 `this.props` 获取到。

### 3、修改仓库数据

`store` 的数据是无法被修改的，这个保证了数据的稳定性。所以 redux 抛出一个 `store.dispatch(action)` 的事件，提供用户修改 `store` 数据。

所以我们继续修改上面的 `pageOne/index.js` 页面(简写)：

```js
class PageOne extends React.Component {
  
  changeName = (val) => {
    this.props.dispatch({
      type: 'setName',
      payload: {
        name: val
      }
    })
  }

  render() {
    return (
      <div>
        <div>name: {name}</div>
        <div>address: {address}</div>
        <div>age: {age}</div>
        <button onClick={ () => { this.changeName('change_name') }}>修改名字</button>
      </div>
    )
  }
}
```

现在去尝试以下执行按钮点击事件吧。

好了，那么至此一个状态管理的操作就完成了。细心的小伙伴会发现 `action` 好像没有用到？

那么这个 `action` 到底是做什么的？

在我理解，就是把 `dispatch` 中的内容放到 `action` 中。

## 五、编写 action

编写 `src/action/oneAction.js`

```js
export const setName = (name) => ({
  type: 'setName',
  payload: {
    name,
  }
})

export const setAge = (age) => ({
  type: 'setAge',
  age
})
```

修改下 `pageOne/index.js` 页面(简写)：

```js
import { setName } from '../../action/oneAction';

class PageOne extends React.Component {
  changeName = (val) => {
    this.props.dispatch(setName(val))
  }
  ...
}
```

执行以下操作是不是发现也可以呢？那么为什么我们还要来编写 `action` 呢？

在我理解：是为了更加注重 `MVC` 的模式，View 就应该注重 View 的展示逻辑，所以与 UI 无关的逻辑操作就交给 redux 来处理，体现代码分层、职责分离的编程思想。

## 六、中间件-异步

### 1、代码操作(第2点是知识点讲解，小伙伴们要看哈)

由于很多时候执行dispatch并不仅仅是立即去更新reducer,这时需要执行其他函数来满足项目需求，这些函数就是中间件，最后执行过一系列中间件后再去执行reducer

如果我们调取服务端的接口，存在时间的延迟；或者说我想在 reducer 中也去调取其他 reducer 的操作，行不行？

我们来实验一下：

我们 `oneAction.js` 文件中再增加一个方法：

```js
export const allChange = () => dispatch => {
  dispatch(setName('all_hang'));
  dispatch(setAge(10010));
}
```

`pageOne.js` 页面上增加一个点击事件(简写)：

```js
import { setName, allChange } from '../../action/oneAction';

class PageOne extends React.Component {

  changeAll = () => {
    this.props.dispatch(allChange())
  }

  render() {
    return (
      ...
      <div>
        <button onClick={ () => { this.changeAll() }}>修改全部</button>
      </div>
      ...
    )
  }
}
```

当我们点击按钮就发现报错了。看下 `console` 的报错信息: `Use custom middleware for async actions.` 翻译过来的意思是：`使用自定义中间件进行异步操作。`

说明在 `reducer` 中调用别的 `reducer` 的方法是可以的，但是因为我们缺少中间件，所以执行报错，现在我们来把中间件加上：

修改代码前我们要编辑 cli,新增一条命令

`yarn add redux-thunk` 或 `npm install --save redux-thunk` 导入 `redux-thunk` 库。

编辑 `src/index.js`(简写):

以下为简写代码，页面上原先的内容不删除，只是新增了几行代码和修改了 `createStore`

```js
import { createStore, applyMiddleware } from 'redux';
import thunkMiddleware from 'redux-thunk';

const store = createStore(
  Reducer,
  applyMiddleware(
    thunkMiddleware
  )
);
...
...
```

那么现在去执行以下 `allChange()` 事件看看效果，是不是发现页面不报错了，且 `name` 和 `age` 的数据也已经做了修改。

### 2、知识点

那么我们来讲解以下 `applyMiddleware` 和 `thunkMiddleware`

什么是 `middleware`： 

在 `redux` 中 `middleware` 是 发送 `action` 和 `action` 到达 `reducer` 之间的第三方扩展，`middleware` 是架在 `action` 和 `store` 之间的一座桥梁。


`applyMiddleware` 可以看看[官网](https://www.redux.org.cn/docs/api/applyMiddleware.html)的解释。

`Middleware` 可以让你包装 `store` 的 `dispatch()` 方法来达到你想要的目的。

### 3、补充

最后来一个补充：

如果你想每次 `dispatch` 都能够在 `console` 打印日志的话，手写会非常的繁琐。

那么 redux 也提供了这样一个中间件，帮助我们打印日志。

键入 `yarn add redux-logger` 或者 `npm install --save redux-logger` 来导入 `redux-logger` 库。

在 `src/index.js` 下加入以下代码：

```js
import { createLogger } from 'redux-logger'

const loggerMiddleware = createLogger()

const store = createStore(
  Reducer,
  applyMiddleware(
    thunkMiddleware,
    loggerMiddleware
  )
);
```

好了，去尝试一下吧～









