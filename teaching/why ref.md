# React ref 多场景使用教程

## 什么是 ref？为什么要使用 ref？

> `ref` 是 React 提供的用来操纵 React 组件实例或者 DOM 元素的接口。

上面的文字很官方，通俗易懂的解释就是通过 `ref` 来操纵我们所创建的自定义组件的内部属性或者事件，或者操纵基础 DOM 元素的能力。

在 React 的数据流中，`props` 是父子组件交互的唯一途径。通过传递一个新的 `props` 值使子组件重新 `re-render`，亦或者通过子组件内部的操作，触发父组件的事件，从而达到父子组件的通信。

当然，存在特殊情况，比如：

- 在父组件中操作子组件的 DOM 元素

在不操纵子组件的能力下，是无法在父组件上拿到子组件定义的值。有小伙伴可能会想只需要将子组件内部的值或者元素提取到父组件即可。没错，这么做当然可以。但是在某些场景下，这个元素放在子组件会更合适，并且有些已经研发成熟的子组件被广泛的使用，不适合再做逻辑的修改。

- `<input />` 外部设置 DOM 元素的获取焦点能力

当外部的按钮点击后需要让 DOM 元素具备一定的能力时，这时候就体现出 `ref` 的必要性。

## ref 在四种场景下的实现

React16.8 后，组件化开发的方式已经被大部分小伙伴使用。那么在组件化研发和类(class)研发中，我们怎么能够实现 ref 的能力呢？

### 父组件：function，子组件：function

```js
import React, { useRef, useState, useImperativeHandle } from "react";

const Son = ({ cRef }) => {
  const [sonValue] = useState("value");

  const SonEvent = () => {
    console.log("this is son event");
  };

  /**
   * 这里是重点！！！！
   * 此处注意useImperativeHandle方法的的第一个参数是目标元素的ref引用
   */
  useImperativeHandle(cRef, () => ({
    // click 就是暴露给父组件的方法
    click: () => {
      SonEvent();
    },
    value: { sonValue },
  }));

  return <div>this is no dva son</div>;
};

const Father = () => {
  const childRef = useRef();
  const getSonEvent = () => {
    childRef.current.click(); // 调用子组件的方法
    console.log(childRef.current.value); // 获取子组件的数据
  };

  return (
    <div>
      <Button onClick={getSonEvent}>调用子组件的方法</Button>
      {/* 子组件引入 cRef 这个名字可随便定义 */}
      <Son cRef={childRef} />
    </div>
  );
};

export default Father;
```

#### useRef

React 提供 `useRef` 的钩子，返回一个可变的 `ref` 对象。其 `.current` 属性被初始化为传递的参数。

**返回的对象将在组件的整个生命周期内保持不变，并且更改 `.current` 属性不会触发 `re-render`**

所以我们在 `Father` 组件创建一个 `useRef` 的钩子。将 `ref` 传递给 `Son` 组件。并通过 `.current` 来获取子组件的值或者执行子组件的事件。

#### useImperativeHandle

正常情况下 `ref` 是不能挂到函数组件上的，因为函数组件没有实例。

但是 `useImperativeHandle` 会为我们提供类似实例的能力，通过 `useImperativeHandle` 的第二个参数，将子组件的自定义值挂在到父组件的 `.current` 上，暴露给父组件使用。

#### forwardRef

`ref` 是 class 组件的关键字，默认返回组件实例。想在 class 组件让它返回非组件实例，就需要用到 `forwardRef` 生成高阶组件将 `ref` 关联到其他地方。

简单的说，通过 `forwardRef` 生成高阶组件实现组件的实例，成功绑定 `ref` 属性。

当然我们可以通过其他的命名来代替 `ref` 就不会出现这个问题，比如上面的代码。但是这会产生歧义，在团队项目研发中不统一。或者一些第三方库已经定义好了 `ref` 的命名，不可修改。

因此，我们下面放上 `forwardRef` 的伪代码，让大家参考：

```js
import React, { useState, useImperativeHandle } from "react";
const Son = React.forwardRef((props, ref) => {
  const [sonValue] = useState("value");
  const SonEvent = () => {};
  useImperativeHandle(ref, () => ({
    click: () => SonEvent(),
    value: { sonValue },
  }));
  return <div>this is no dva son</div>;
});

export default Son;
```

### 父组件：class，子组件：class

```js
import React, { Component } from "react";

class Son extends Component {
  state = {
    sonValue: "sonValue",
  };

  sonEvent = () => {
    console.log("this is son event");
  };

  render() {
    return <div>son</div>;
  }
}

class Father extends Component {
  sonChild;

  getSonEvent = () => {
    this.sonChild.sonEvent();
    console.log(this.sonChild.state);
  };

  render() {
    return (
      <div>
        <Button onClick={this.getSonEvent}>调用子组件的方法</Button>
        <Son
          ref={(e) => {
            this.sonChild = e;
          }}
        />
      </div>
    );
  }
}

export default Father;
```

在类子组件中，有独立的实例，所以父组件通过 `ref` 可以直接调用子组件下的属性和方法。

### 父组件：function，子组件：class

对于子组件使用 class 的场景。子组件已经拥有独立的实例，所以子组件不需要有任何改动。

而父组件通过 `useRef` 创建 `ref` 对象，传递给子组件，即可通过 `.current` 实现父子组件数据通信。请参考 `父组件：class，子组件：class` 的演示代码。

### 父组件：class，子组件：function

这里的代码和 `父组件：function，子组件：function` 类似。子组件需要通过 `useImperativeHandle` 提供类似实例的能力。将自定义的属性暴露给父组件。

## 使用 dva 造成的 ref 报错

当使用 dva 的 `connect` 对子组件进行绑定时，父组件传入的 `ref` 会被过滤掉。导致功能失败。

### 子组件 function 绑定 dva

因为 `ref` 通过 `forwardRef` 生成的高阶组件再被包上一层 `connect` 会失效。导致内部实例无法传递到外部。

所以这里需要调整高阶组件的使用顺序。先用 `connect` 包裹，再使用 `forwardRef` 包裹实现。

```js
import React, { useState, useImperativeHandle, forwardRef } from "react";
import { connect } from "alita";
const Son = ({ refInstance }) => {
  const [sonValue] = useState("value");

  const SonEvent = () => {};

  useImperativeHandle(refInstance, () => ({
    click: () => SonEvent(),
    value: { sonValue },
  }));

  return <div>this is no dva son</div>;
};

const Component = connect()(Son);

export default forwardRef((props, ref) => <Component {...props} refInstance={ref} />);
```

### 子组件 class 绑定 dva

和上述报错逻辑一致。需要通过 `forwardRef:true` 来实现。

```js
import React, { Component } from "react";
import { connect } from "alita";

const mapDispatchToProps = (dispatch, ownProps) => {
  return { dispatch };
};

@connect(({ classclass }) => ({ classclass }), mapDispatchToProps, null, { forwardRef: true })
class Son extends Component {
  state = {
    sonValue: "sonValue",
  };

  sonEvent = () => {};

  render() {
    return <div> son</div>;
  }
}

export default Son;
```

至此 `ref` 的使用场景示例和思路已经表达相对清晰，有没有提及的场景或者表达不清的内容，欢迎小伙伴们评论区留言。
