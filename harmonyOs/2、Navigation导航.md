
# Navigation 组件导航

[官网内容]('https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-navigation-navigation')

想实现页面间的跳转就需要使用 `Navigation` 的跳转，就需要在入口根目录上接入。

通过导航栏控制器`NavPathStack` 来实现页面。所以需要创建一个属性，放置到 `Navigation` 上。

每个页面都需要在组件 `onReady` 的时候，将 `context` 传递给 `NavPathStack` 实现管理，保证跳转。

## 一、增加路由表配置

```js
{
    "module": {
        "routerMap": "$profile:route_map"
    }
}
```

## 二、增加路由表

```js
  {
    "routerMap": [
      {
        "name": "PageOne",
        "pageSourceFile": "src/main/ets/pages/PageOne.ets",
        "buildFunction": "PageOneBuilder",
        "data": {
          "description" : "this is PageOne"
        }
      }
    ]
  }
```

## 三、根组件创建 `NavPathStack` 实现路由管理

```js
  @Entry
  @Component
  struct Index {
    pageStack : NavPathStack = new NavPathStack();

    build() {
      Navigation(this.pageStack){
      }.onAppear(() => {
        this.pageStack.pushPathByName("PageOne", null, false);
      })
      .hideNavBar(true)
    }
  }
```

## 子页面保持页面路由配置

```js
  // 跳转页面入口函数
  @Builder
  export function PageOneBuilder() {
    PageOne();
  }

  @Component
  struct PageOne {
    pathStack: NavPathStack = new NavPathStack();

    build() {
      NavDestination() {
      }
      .title('PageOne')
      .onReady((context: NavDestinationContext) => {
         this.pathStack = context.pathStack;
      })
    }
  }
```