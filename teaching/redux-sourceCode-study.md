# Redux 源码学习

## 一、Utils

### 1、actionTypes.ts

```ts
const randomString = () => Math.random().toString(36).substring(7).split('').join('.');

const ActionTypes = {
  INIT: `@@redux/INIT${randomString()}`,
  REPLACE: `@@redux/REPLACE${randomString()}`,
  PROBE_UNKNOWN_ACTION: () => `@@redux/PROBE_UNKNOWN_ACTION${randomString()}`
}

export default ActionTypes;
```

抛出两个初始的 `action`, 其实就是一个命名，增加了7位的随机数。`toString(36)` 代表随机数的范围：数字+26个字母

### 2、warning.ts

```ts
export default function warning(message: string): void {
  if(typeof console !== undefined && typeof console.error === 'function') {
    console.error(message);
  }
  try {
    throw new Error(message)
  } catch(e) {}
}
```

给 `console` 增加一层判断，ie8 一下都不支持 console。

### 3、isPlainObject.ts

对对象做是否是简单对象的判断。**凡不是new Object()或者字面量的方式构建出来的对象都不是简单对象**

所谓的简单对象就是该对象的__proto__等于Object.prototype

即  `={}` 或 `new Object()` 出来的对象才是简单对象。

```ts
export default function isPlainObject(obj: any): boolean {
  if(typeof obj !== 'object' || obj === null) return null;
  let proto = obj;
  if(Object.getPrototypeOf(proto) !== null) {
    proto = Object.getPrototypeOf(proto);
  }
  return proto === Object.getPrototypeOf(obj)
}
```

## 二、src

### 1、index.ts

首页就是 `redux` 的主入口，结尾抛出5种我们常用的方法。在 `index.ts` 中，只有一个判断函数，我们来看下：

防止在开发环境中对代码惊醒压缩。

```ts
function isCrushed() {
  if(
    process.env.NODE.DEV !== 'product' &&
    typeof isCrushed.name === 'string' &&
    isCrushed.name === 'isCrushed';
  ) {
    warning();
  }
}
```

如果是在开发环境中对代码进行压缩的话，isCrushed.name 会为 `isCrushed`。

### 2、createStore.ts

`createStore` 可接受三个参数(reducer, preloadedState, enhancer)

一般 `preloadedState`， 我们用的比较少先不说。第三个参数为增强器。



**浓缩：1个操作，4个判断，5个赋值，6个方法。**

- 判断是否第二、三个参数都是方法
- 判断是否第三个参数未传递，第二个参数传递进来的是方法
- 增强器一定要为方法
- reducer 一定要为方法

- getState()
- dispatch(action)
- ensureCanMutateNextListeners()
- subscribe(listener)
- replaceReducer()
- observable()

#### 4个判断其一：

```ts
if(
  (typeof preloadedState === 'function' && typeof enhancer === `function`) || 
  (typeof enhancer === 'function' && typeof arguments[3] === 'function')
) {
  warning('如果你第二第三个参数传递的都是方法，那么这两个参数会被当作是增强器，而发出警告，提示要将增强器的数据放到一个函数里传递进来')
}
```

#### 4个判断其二：

将第二个参数赋值给第三个参数，将第二个参数设置为 `undefinded`

```ts
if(typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
  enhancer = preloadedState;
  preloadedState = undefined;
}
```

#### 4个判断其三：

```ts
if(typeof enhancer !== 'undefined') {
  if(typeof enhancer !== 'function') {
    warning('增强器一定要为方法');
  }
  return enhancer(createStore)(reducer, preloadedState)
}
```

#### 4个判断其四：

```ts
if(typeof reducer !== 'function')) { warning('reducer 一定要为方法') }
```

#### 5个赋值：

- 重新赋值一遍 reducer
- 重新赋值一遍 第二个参数 state
- 创建监听者列表
- 浅拷贝一份监听者列表
- 锁。若仓库的数据源在修改中，则其他 `action` 将不能进行操作，每次修改只能一个。

```ts
let currentReducer = reducer;                         
let currentState = preloadState;                      
let currentListeners: (() => void)[] | null = [];     
let nextListeners = currentListeners;                     
let isDispatching = false;                            
```

#### 6个方法其一：getState()

```ts
function getState() {
  if(!isDispatching) {
    return currentState;
  }
}
```

这里我们看到 `getState()` 并没有对数据源进行浅拷贝，而是直接返回给我们，所以我们是可能对获取到数据直接修改的。但是修改后不会通知到其他的监听器。

#### 6个方法其二：dispatch(action)

首先先做三个判断：1、是否是简单函数 2、是否类型未定义 3、是否锁开着

```ts
function dispatch(action: A) {
  if(!isPlainObject(action)) throw new Error('');

  if(typeof action.type === `undefined`) throw new Error('');

  if(isDispatching) throw new Error('');

  // 锁起来，执行仓库数据源修改操作，放开锁
  try {
    isDispatching = true;
    currentState = currentReducer(currentState, action);
  } finally {
    isDispatching = false;
  }

  // 通知一个个订阅者
  let listeners = (currentListeners = nextListeners);
  for(let i = 0; i < listeners.length; i++) {
    const listener = listeners[i];
    listener();
  }
  return action;
}
```

#### 6个方法其三：ensureCanMutateNextListeners()

对监听者的一个浅拷贝,防止用户在调度时调用 `subscribe/unsubscribe` 会出错误。

```ts
function ensureCanMutateNextListeners() {
  if(nextListeners === currentListeners) {
    nextListeners = currentListeners.slice();
  }
}
```

#### 6个方法其四：subscribe(listener)

注册订阅者之前，首先先做两个判断：1、监听对象是否是个方法 2、仓库是否锁着

```ts
function subscribe(listener: () => {}) {
  if(typeof listener === 'function') throw new Error('');

  if(isDispatching) throw new Error('');

  let isSubscribed = true;
  ensureCanMutateNextListeners();
  nextListeners.push(listener);

  return function unsubscribe() {
    if(!isSubscribed)  return ;
    if(isDispatching) throw new Error('');

    isSubscribed = false;
    ensureCanMutateNextListeners();

    // 这里为什么要删除呢，因为取消监听了呀
    const index = nextListeners.indexOf(listener);
    nextListeners.splice(index, 1);
    currentListeners = null;
  }
}
```

#### 6个方法其五：replaceReducer(nextReducer)

该方法是用来替换 `reducer` 的，执行操作前，先判断一下 `nextReducer` 是否是方法

```ts
function replaceReducer(nextReducer) {
  if(typeof nextReducer !== 'function') throw new Error('');

  currentReducer = nextReducer;
  dispatch({ type: ActionTypes.REPLACE })
}
```


#### 6个方法其六： observable()

暂不做解释

#### 1个操作

```ts
dispatch({ type: ActionTypes.INIT })
```

`createStore` 方法里的唯一一个操作。做什么呢？

用来初始化整个 `store` 的数据结构， 获取 `reducer` 的默认数据，并结合 `combine`, 生成仓库的初始数据源

如果没有该行代码，仓库中的数据源就为 `undefined`，无法进行接下来的操作。

### 3、combineReducers.ts

如果小伙伴直接看源码可能会比较懵一些，代码并不长，但是有些方法和判断未必看得懂。这里我先给小伙伴举个该文件的作用的最简单的例子。

我们知道 `combineReducers` 就是将所有的子 `reducer` 函数组成对象，转换成一个新的 `reducer` 函数。如：

```ts
const reducer1 = (
  state = {
    name1: 'name1',
    age1: 1,
  },action
) => { ... }

const reducer2 = (
  state = {
    name2: 'name2',
    age2: 2
  }, action
) => { ... }

export default combineReducers({
  reducer1, 
  reducer2
})
```

最后要组合成的 `state` 样子长这个样子：

```json
{
  "reducer1": {
    "name1": "name1",
    "age1": 1
  },
  "reducer2": {
    "name2": "name2",
    "age2": 2
  }
}
```

所以，其实该文件最核心的代码并不多，最大的作用就是将 `state` 进行一个转化。至于该文件中的其他方法和判断，就是增强 `redux` 的健壮性(本人理解，不一定正确)。

**一共有3个方法和一个核心代码**：我将分开讲解：

#### 3个方法其一：getUndefinedStateErrorMessage()

通过方法名我们就可以理解，获取未定义 `state` 的错误信息。

```ts
function getUndefinedStateErrorMessage(key: string, action: Action) {
  const actionType = action && action.type;
  // 这里就是做个判断
  // 如果 actionType 存在就展示，否则就展示 an action
  const actionDescription = ( actionType && `action "${String(actionType)}"` ) || 'an action'

  return (...)  // 返回文字信息
}
```

#### 3个方法其二：assertReducerShape(reducers)

该方法的作用是检查每个 `reducer` 的是否具有默认的返回值，讲解一下步骤：

- 便利 `reducers` 的 `key`。小伙伴如果有看 `combineReducers.ts` 我写的例子就能知道，这里的 `key` 就是 `reducer1` 和 `reducer2`
- 每个 `reducer` 都执行 `init` 的操作。
- 如果执行了第二步，返回回来的是 `undefined`, 则说明该 `reducer` 未定义默认的返回值

```ts
function assertReducerShape(reducers: ReducersMapObject) {
  Object.keys(reducers).forEach(key => {
    const reducer = reducers[key];
    const initialState = reducer(undefined, { type: ActionTypes.INIT });
    if(typeof initalState === 'undefined') throw new Error('');
    if(typeof reducer(undefined, { type: ActionTypes.PROBE_UNKNOWN_ACTION() }) === 'undefined') throw new Error('');
  })
}
```

#### 3个方法其三：getUnexpectedStateShapeWarningMessage(inputState, reducers, action, unexpectedKeyCache) 

不懂的小伙伴可以大致理解下就行。

不要看方法名很长，参数很多。我们慢慢来看。通过方法名，我们大致可以知道这个方法的作用：获取异常 `State` 的 `key`，发出警告。我们来看下步骤：

- 首先保证 `reducers` 不为 `{}` 且 `inputState` 为简单函数。
- 取出 `inputState` 中 `reducers` 不存在的 `key`。
- 如果是替换 `reducer` 的 `action` 就直接跳过。
- 如果存在未定义的 `key` 就将他们打印出来。

```ts
function getUnexpectedStateShapeWarningMessage(inputState, reducers, action, unexpectedKeyCache) {
  const reducerKeys = Object.keys(reducers);

  if(reducerKeys.length === 0) return '';
  if(!isPlainObject(inputState)) return '';

  const unexpectedKeys = Object.keys(inputState).filter(key => {
    !reducers.hasOwnProperty(key) && !unexpectedKeyCache(key);
  })

  unexpectedKeys.forEach(key => unexpectedKeyCache[key] = true )

  if (action && action.type === ActionTypes.REPLACE) return
  
  if (unexpectedKeys.length > 0) return '';
}
```

#### 核心代码

这里就是整合代码，该方法中有些判断的代码我就不做展示，如果小伙伴想看完整源码就去官网上拉取哈，我们来看下步骤：

- 对 `reducers` 做一层浅拷贝
- 返回一个 `combination(state, action)` 方法
- 方法中，遍历 `state` 的 `key` 将每个子 `reducer` 的 `state` 都取出来放到一起去。
- 判断是否有改变，如果改变了就返回新的 `state` 否则就返回旧的 `state`

```ts
const reducerKeys = Object.keys(reducers);
const finalReducers: ReducersMapObject = {}      // 浅拷贝后的 reducers
for(let i = 0; i < reducerKeys.length; i++) {
  const key = reducerKeys[i];

  if(typeof reducers[key] === 'undefined')  warining();
  if(typeof reducers[key] === 'function') finalReducers[key] = reducers[key];
}
const finalReducersKeys = Object.keys(finalReducers);

return function combination(state, action) {
  let hasChanged = false;         // 这里做判断是否
  const nextState = {};
  for(let i = 0; i < finalReducersKeys.length; i++) {
    const key = finalReducerKeys[i];
    const reducer = finalReducers[key];
    const previousStateForKey = state[key];
    const nextStateForKey = reducer(previousStateForKey, action);   // 这里就是取出每个 reducer 的数据源, 这里可以把 nextStateForKey 打印出来看看
    if (typeof nextStateForKey === 'undefined') throw new Error;
    nextState[key] = nextStateForKey;         // 将每个数据源放到新的 state 中
    hasChanged = hasChanged || nextStateForKey !== previousStateForKey  // 判断新的 state 和之前的 state 有不一样。
  } 

  // 这里还判断了一下前后 key 的长度，非常的严谨
  hasChanged = hasChanged || finalReducerKeys.length !== Object.keys(state).length
  
  return hasChanged ? nextState : state
}
```

如果小伙伴还觉得上面的代码麻烦的话，自己写的 demo，可以忽略掉 `hasChanged` 以及对 `nextStateForKey` 的类型判断。

### 3、compose.ts

小伙伴不一定很清楚这个类是干嘛的，如果是在 2019.8月后拉取的官网代码可能会觉得比较这个类量也不少，其实不是的。。最最核心的代码就下面的几行。小伙伴找找：

```ts
export default function compose(...funcs) {
  if(funcs.length === 0) return funcs => funcs;

  if(funcs.length === 1) return funcs[0];

  return funcs.reduce((a, b) => (...args) => a(b(...args))); 
}
```

`redux` 的 `index.ts` 是有把 `compose` 暴露出来，只是可能小伙伴用的比较少。官网也有将 `compose` 作为 api 单独拎出来说。

[点这里打开官网 `compose` 介绍](https://www.redux.org.cn/docs/api/compose.html)

我们先来看看官网给出的示例： 

```ts
import { createStore, combineReducers, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import DevTools from './containers/DevTools'
import reducer from '../reducers/index'

const store = createStore(
  reducer,
  compose(
    applyMiddleware(thunk),
    DevTools.instrument()
  )
)
```

- `compose` 在 `redux` 的作用是什么呢：将多个 `sotre 增强器` 依次执行。

- 那 `compose.ts` 的作用又是什么呢？通过最后一行代码我们可以看出是**从右到左来组合多个函数**。

`es6` 里的 `reduce` 是个累加器, 写个demo 大家参考下。

```ts
const sum = [1,2,3,4].reduce((acc, current) => {
  console.log(acc, current);
  return acc + current;
})
console.log(sum);
```

打印结果为：

```
1 2
3 3
6 4
10
```

即 `acc` 为每次累加后 `return` 的结果，`current` 为当前的数组元素。

由此便可知道入参的方法至少要为两个，否则就会报错了。这也就解释了，为什么代码中 `funcs` 要对长度做 `=== 0` 和 `=== 1` 的判断了。

那么这个方法是如何操作多个 `funcs` 的呢？

官网中已经给出了示例：`compose(funcA, funcB, funcC)` ===> `compose(funcA(funcB(funcC())))`

从右到左把接收到的函数合成后的最终函数。

### 4、applyMiddleware.ts

代码量也不多。我们先来看下源码是怎么写的：

```ts
function applyMiddleware(...middlewares) {
  return createStore = (...args) => {
    const store = createStore(...args);
    let dispatch = () => {}

    const middlewareAPI = {
      state: store.getState;
      dispatch: (...args) => dispatch(...args)
    }

    const chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);
  }
}
```

代码没几行，但是却实在看不懂。很迷是不是？比如 `createStore` 到底哪里来的是不是？

首先我们先打开刚刚讲解过的 `createStore.ts` 里的四个判断里的其中一个：判断 `enhancer 增强器` 一定要为方法。回顾一下代码：

```ts
if(enhancer !== 'undefined') {
  if(enhancer !== 'function') throw new Error('');
  return enhancer(createStore)(reducer, preloadedState);
}
```

小伙伴发现了没有？当在外部执行 `createStore()` 方法时，在做核心代码前会先判断下 `enhancer` 增强器是否存在

如果存在的话就直接返回 `enhancer` 增强器，并且将 `createStore()` 方法传递给了增强器。**所以呀～当 `createStore` 存在增强器时，`createStore` 根本就没走。而是直接被返回出去了。。。**

**所以 `applyMiddleware` 方法中的 `createStore` 是在这里传递进来的。并且一并将 `reducer` 和 `preloadState` 也传递过来了, `createStore` 在是 `applyMiddleware` 被执行的**。

因为中间件是外部的东西(将来小伙伴们厉害了，也可以自己搞一个)，所以在遍历执行每个中间件的时候，将数据 `state` 和 `dispatch` 暴露出去给中间件执行和使用。

整合好后通过 `compose` 重写 `dispatch` 最后将 `store` 和 `dispach` 返回回去。

### 5、bindActionCreators.ts

`redux` 提供的这个方法，其实可以通过 `react-redux` 提供 `dispatch` 去执行。

本人对此没有太深的研究(不想看了)。还请感兴趣的小伙伴自行学习哈。











