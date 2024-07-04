# React ref 多场景使用教程

React16.8 后，我们就不断的拥抱 Hook 开发，在组件化开发已经成为我们必不可缺的开发方式时，ref 的使用就显得尤为重要。

但是 class 开发依旧被大部分人所使用，所以我们来分析几种使用 ref 的场景。

下面存在几种场景：

- 父组件: class, 子组件: class
- 父组件: hook, 子组件: hook
- 父组件: class, 子组件: hook
- 父组件: hook, 子组件: class


**上面场景如果在 `dva` 中使用会有问题，每种场景会对此进行分析**

下面通过这几种场景分析：如何在 `父组件` 调用 `子组件内部` 的 `方法`以及 `数据`。

## 一、父组件: class, 子组件: class

### 1、子组件不使用 `dva` 情况

子组件不需要任何多余的操作, 直接编写父组件即可。

```js
...
import { Button } from 'antd-mobile';
import { NoDvaSon } from './components'; // 不是用 dva 的子组件

@connect(({ classclass }) => ({ classclass }))
class ClassclassPage extends Component {
  noDvaChild: any;

  getSonEvent = () => {
    this.noDvaChild.noDvaSonEvent(); // 这里调用子组件的方法
    console.log(this.noDvaChild.state); // 这里获取子组件的 state 值
  };

  render() {
    return (
      <div>
        <Button onClick={this.getSonEvent}>调用子组件的方法</Button>
        <NoDvaSon
          ref={(e) => {
            this.noDvaChild = e;
          }}
        />
      </div>
    );
  }
}
export default ClassclassPage;
```

### 2、子组件使用 `dva` 情况

父组件一样按照上面的编写方式不变，只不过子组件加上了 `dva`，保存后打开页面，浏览器的 `Console` 就会有以下的警告。

![1-1](assets/../../gitImg/ref/1-1.jpg)

这时我们点击父组件的按钮调用子组件的方法和数据时，就会报错。

接下来我们来修改 子组件的代码：

修改 connect 的字段：

```js
const mapDispatchToProps = (dispatch, ownProps) => {
  return { dispatch };
};

@connect(({ classclass }) => ({ classclass }), mapDispatchToProps, null, { forwardRef: true }) // 新 react-redux 版本支持
@connect(({ classclass }) => ({ classclass }), mapDispatchToProps, null, { withRef: true }) // 旧 react-redux 版本支持
```

只要引入 `forwardRef` 或者 `withRef` 报错就迎刃而解。

在写 demo 时我使用的是 `withRef` 发现浏览器 `Console` 还是报错，并提示 `withRef` 不能使用，所以就换成了 `forwardRef`。

## 二、父组件: hook, 子组件: hook

在 `hook` 中，使用 `dva` 不会影响到 `ref` 的使用。

父组件代码： 

```js
import React, { FC, useRef } from 'react';
import { Button } from 'antd-mobile';
import { NoDvaSon } from './components';

const HookhookPage: FC = () => {
  const childRef = useRef(); 

  const getSonEvent = () => {
    childRef.current.click(); // 调用子组件的方法
    console.log(childRef.current.value); // 获取子组件的数据
  };

  return (
    <div className={styles.center}>
      <Button onClick={getSonEvent}>调用子组件的方法</Button>
      {/* 子组件引入 cRef 这个名字可随便定义 */}
      <NoDvaSon cRef={childRef} /> 
    </div>
  );
}

export default connect(({ hookhook }) => ({ hookhook }))(HookhookPage);
```

子组件代码：

```js
import React, { FC, useState, useImperativeHandle } from 'react'; // 引入需要的 useImperativeHandle

interface PageProps {
  cRef: any;
}

const Page: FC<PageProps> = ({ cRef }) => {
  const [noDvaValue, setNoDvaValue] = useState('noDvaValue');

  /**
   * 这里是重点！！！！
   * 此处注意useImperativeHandle方法的的第一个参数是目标元素的ref引用
   */
  useImperativeHandle(cRef, () => ({
    // click 就是暴露给父组件的方法
    click: () => {
      noDvaSonEvent();
    },
    value: { noDvaValue },
  }));

  const noDvaSonEvent = () => {
    console.log('this is no dva son event');
  };

  return <div>this is no dva son</div>;
};

export default Page;
```

## 三、父组件: class, 子组件: hook

这种情况下直接参考 **二、父组件: hook, 子组件: hook** 即可。将不在详细说明，demo 中也不做展示。

## 四、父组件: hook, 子组件: class

对于 `class` 类型的子组件来说，实现方法和 `一、父组件: class, 子组件: class` 差不多。这里附上代码供大家参考

对于不使用 `dva` 的的子组件来说，子组件不需要增加代码，只需要在父组件上编写即可：

```js
import React, { FC, useRef } from 'react';
import { HookhookModelState, ConnectProps, connect } from 'alita';
import { Button } from 'antd-mobile';
import { NoDvaClassSon } from './components';

const HookhookPage: FC = ({ hookhook, dispatch }) => {

  let classChild: NoDvaClassSon | null = null;

  const getclassSonEvent = () => {
    classChild.noDvaSonEvent(); // 调用子组件的方法
    console.log(classChild.state); // 获取子组件的数据
  };

  return (
    <div className={styles.center}>
      <Button onClick={getclassSonEvent}>调用class子组件的方法</Button>
      <NoDvaClassSon
        ref={(e) => { // 使用 ref
          classChild = e;
        }}
      />
    </div>
  );
};

export default connect(({ hookhook }) => ({ hookhook }))(HookhookPage);
```

而使用 `dva` 的子组件则像 `一、父组件: class, 子组件: class` 加上 `forwardRef: true` 即可。









