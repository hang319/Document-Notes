# @wufengteam/core 统一中心注册器

## 一、功能解析

统一中心注册器，顾名思义可以理解为将需要具备的资产汇总到统一的地方实现实现管控。并且能够做到资产的统一性、完整性和可配置性。

这里的资产指的是组件、图标、方法、配置等可支持统一管控的东西。

拿我们之前做产品预演的一个 demo "无锋零代码平台"来举个例子。

零代码平台最基础的就是要提供可拖拽配置的组件。因为做的是平台，所以我们要尽可能的支持让外部的资产适配我们的平台，且要保证统一的规则。那么如何做到统一和开放的平衡就变得尤为重要。

## 二、使用场景

### 1、组件注册

通过 `import { wufengController } from '@wufengteam/core';` 来引出统一中心注册器的主角。

当用户开发完自定义的组件，就可以通过以下的配置将组件注册进去。

```js
wufengController.registerComponent(Text, { ...config }, "pc");
```

从上述的代码中我们可以看出这个注册组件的能力需要传入三个参数。首先是开发人员研发的定制组件。

其次是该组件的配置表，比如：组件对应的图标、组件的名称、所在分组、传入的默认参数、样式编辑器等配置。这里会在下个部分：源码解析中来分析。这里只需要了解一个大致的能力即可。

那么最后就是组件对应的设备，分为 `pc` 和 `h5` 两个参数，来区分 pc 端和移动端的网页。

很多组件在 pc 和 h5 两个场景下并不能共用，因为相同的组件名可能需要开发两套组件来解决场景问题。但是对于配置表来说，应该要做到一致性。配置一个组件的配置表，要能够支持多个场景的使用。

通过类似的能力，可以通过 `registerIcons`、`setLang`、`setPlatform` 等来注册图标、国际化、平台标识等等。

### 2、方法注册

对于复杂的组件来说，可能会在内部调用特定的方法来达到组件的功能完整性。比如选址组件，我们需要不断的调用接口，来获取对应的下一层级的数据。

那么就需要通过 `wufengController.registerAction` 来实现方法的注册。

```js
wufengController.registerAction("apiName", (params) => {
  return reqApiFunction(params).then((res) => {
    return res;
  });
});
```

通过上述的代码，我们定义了方法名，并将异步方法进行注册。最终我们只要保证异步请求的数据回传到组件内部能够和内部逻辑对应得上即可。

所以，同一套组件支持在不同的框架中使用，因为我们提供了开放性的请求逻辑，将方法抛出去给对应的研发人员编写，解决了框架对于注册器的影响。

**这里尤其要注意**

很多场景下，我们引入了外部的组件进行注册，但是往往忽略了组件内部需要用到的方法。在组件的使用中就回导致报错。

因此，在执行组件的某个方法逻辑报错时，我们不妨在浏览器的控制台上打印下 `window.wufengController`。看看 `actions` 是否有被注册的方法。

## 三、源码解析

铺垫了这么多，无非是希望小伙伴们在源码解析前能够了解到一定的前提知识。所以接下来才是我们的重点分析。

### 1、创建

既然是全局的注册器，那么肯定是要在项目启动时(或者可以理解为源码自动在全局引入)。

因此，最重要的我们要创建一个构造函数，并实例化一个对象提供给开发者，来支持注册器的能力。

```js
export class WuFeng {
  constructor() {}
}

let wufeng: WuFeng;
wufeng = new WuFeng();
window.wufengController = wufeng;
export { wufeng as wufengController };
```

### 2、单例模式

我们肯定是希望在全局变量里有且只有一个实例化出来的 `wufengController`。但是我们并不能阻止它实例化多个对象。最好的办法就是让类自身负责，保持它的唯一实例。

那怎么实现单例模式？

并不复杂。用一个变量来缓存一个类实例化生成的对象，然后用这个变量来判断一个类是否是否已经被实例化过。如果变量有值，则在下一次要获取该类生成实例化对象时，直接返回之前的变量即可。

简而言之：没有就 new 一个出来，如果已经有了还 new，就直接返回之前已经 new 过的变量。

```js
export class WuFeng {
  constructor() {
    if (WuFeng.singletonInstance) {
      return WuFeng.singletonInstance;
    }
  }

  static singletonInstance: WuFeng;
}

let wufeng: WuFeng;
if ((window as any).wufengController) {
  wufeng = (window as any).wufengController;
} else {
  wufeng = new WuFeng();
}
WuFeng.singletonInstance = wufeng;
window.wufengController = wufeng;
export { wufeng as wufengController };
```

这里在创建完 `wufeng` 的实例化对象后，就将这个对象赋给静态值 `singletonInstance`。而在构造函数初始化时会判断 `singletonInstance`，如果有值，则直接返回，不再重新构造。

### 3、公有属性

接下来就是就是定义类中的公有成员(属性)。这里的成员都会配套上两个方法。

- `set` 方法：允许用户从外部将数据保存到成员中。
- `get` 方法：允许用户获取内部的成员数据。

比如 `actions` 这个对象，用户存放用户配置给内部组件使用的方法。会有 `registerAction` 和 `getAction` 的方法。

```js
export class WuFeng {
  ...// 忽略部分代码

   // 所有需要翻译的标签
  public actions: any = {};

  // 运行时注册，可以简化配置，不用理会编辑器配置
  public registerAction(type: string, action?: (params?: any) => Promise<any>) {
    if (this.actions[type]) {
      console.log(`事件 [ ${type} ] 已存在，已被在此覆盖注册，请确认\n`);
    }
    this.actions[type] = action;
    return null;
  }

  // 获取存在的事件
  public getAction(type: string, params?: any): Promise<any> {
    if (!this.actions[type]) {
      console.warn(`事件 [ ${type} ] 未找到，请检查您的<事件>注册函数\n`);
      return new Promise((resolve) => {
        resolve(null);
      });
    }
    return new Promise((resolve) => {
      this.actions[type](params).then(
        (res: any) => {
          resolve(res);
        },
        (error: any) => {
          console.warn(error);
          // TODO: 底层 reject 数据是否透传到顶层？
          resolve(null);
        },
      );
    });
  }
}
```

剩下的对象，比如组件、图标、监听事件等，也和上大同小异。

### 4、配置表

对于组件来说，最重要的还是配置表，相同的组件代码，不同的配置可能带来不同的效果。为了避免相似组件二次开发的浪费，我们将配置表开放给开发者来配置，提供了组件的复用性。

而 `class` 中定义的 `components` 也做了清晰的类型定义。

在源码 `src/types.ts` 中，清晰的看到组件配置表的 api。

### 5、完善

罗马不是一天建成的。注册器的库最开始并没有具备如此丰富的能力。有了基础的能力后，都是在业务的驱动下不断的完善注册器的能力以支撑繁杂的需求。

就拿注册器来说。无锋零代码平台一开始对标的是完善的表单能力。而后续针对表单数据的统计，我们衍生出了具备筛选能力的仪表盘模块。筛选模块和仪表盘组件本身不具备通信的能力。为了实现这个能力，才增加 ` public subscriptions = {};` 的成员，并提供 `useSubscription`、`emit` 来实现注册和通知的方法。

## 四、结束语

`@wufengteam/core` 统一中心注册器对于小伙伴们可能无法在日常业务中使用到，且这个注册器对于外部公司亦或者是不同的项目组来说使用的意义并不大。但是整体结构和仓库作者在开发时的思路确是有值得我们参考和借鉴的地方。本篇文章通过源码解析的模块是希望读者能够通过对源码和场景的理解而复用到相似场景中去。
