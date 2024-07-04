# eggjs 实现服务端请求教程文档

## 简介

该教程适合入门级小伙伴，使用 node `eggjs` 框架实现服务端开发，以及处理开发过程中遇到的问题。

教程中的每个章节会附上 **demo(git 分支)**，小伙伴可以自行拉取代码查看。

教程和 demo 会使用基于 `umi` 二次封装的 `alita` 框架进行页面渲染，重点讲解 `eggjs` 的 node 服务端框架。

实现通过 `get` 请求获取的 `user` 列表，并通过 `post` 请求获取详细的 `user` 信息。

~~作为入门级的我们，首先不用太关注过多的 api，技术类的东西建议先掌握了用法，用多了就能慢慢理解其中的奥义。所有光看不实操的小伙伴都是刷流氓。~~

现在我们先来创建 `egg` 框架。

## 一、eggjs 框架搭建和讲解

[eggjs 官网-快速入门](https://eggjs.org/zh-cn/intro/quickstart.html)

通过上面的链接，我们能够快速搭建 `eggjs` 的脚手架。

```bash
# 新建文件夹
mkdir eggDemo && cd eggDemo
# 搭建脚手架命令
npm init egg --type=simple

# 手动输入项目 pkg 信息
？project name egg
? project description
? project author
? cookie security keys 1620311360934_6251

# 进入项目安装依赖
cd init
yarn
```

在安装依赖的时间里，可以打开 `package.json` 文件看下 `script` 的标签。

```js
"scripts": {
    "start": "egg-scripts start --daemon --title=egg-server-egg",
    "stop": "egg-scripts stop --title=egg-server-egg",
    "dev": "egg-bin dev",
    ...
},
```

- `yarn run start`：启动 `egg` 服务
- `yarn run stop`：停止 `egg` 服务
- `yarn run dev`：启动开发模式，支持热部署

装好依赖后执行 `yarn run dev` 脚手架已经给我们写好了最简 demo。实现了一个最简单的接口。

![img](../gitImg/egg/egg1.jpg)

**`eggjs` 奉行『约定优于配置』**，可以简单的理解为：以什么命名的文件和文件夹下就做什么事情(描述不准确，但是八九不离十)。

[完整的目录结构请看官网讲解](https://eggjs.org/zh-cn/basics/structure.html)，看下脚手架为我们搭建的几个目录：

- `/config/config.default.js`: 配置文件
- `/config/plugin.js`: 配置插件

---

- `/app/router.js`: 用于配置 URL 路由规则,即每个接口地址
- `/app/controller/**`: 用于解析用户的输入，处理后返回相应的结果
- `/app/public/**`: 用于放置静态资源

在真实的项目中会存在成百上千个接口，如果全部都挤在 `app/router.js` 里可能效果不佳。一般会分模块或者分后台中心来实现。

我们要实现的 demo 中有两个页面，分别为 `用户列表页` 和 `用户详情页`。我们就按页面来区分，新建两个文件用来存放路由。

```js
module.exports = app => {
  require('./router/list')(app);
  require('./router/detail')(app);
};
```

![img](../gitImg/egg/egg2.jpg)

最后我们快速搭建一个 前端 web 框架。

```bash
# 在 eggDemo 的文件夹下键入 web 脚手架
yarn create alita eggWeb

cd eggWeb
yarn
```

初始化框架可以看下 demo 中的 `/config/config.ts` 和 `/src/app.ts` 文件。

**[feat-frame-building 框架搭建分支](https://github.com/hang1017/eggDemo/tree/feat-frame-building)**

## 二、用户列表接口开发(get 请求)

### egg

每个接口都有独一无二的路由(接口名)，

所以我们先来实现路由的编写：

`/router/list.js`

```js
'use strict';

module.exports = app => {
  const { router, controller } = app;
  router.get('/api/getUserList', controller.home.userList);
};
```

- `router.get` 代表 get 请求。
- `controller.home.userListController`: 代表 `controller` 文件夹下存在 `home` 文件，文件内有 `userListController` 的方法。

`/app/controller/home.js`

```js
const Controller = require('egg').Controller;

class HomeController extends Controller {
  async userListController() {
    const { ctx } = this;
    const data = Array.from(new Array(9)).map((_val, i) => ({
      name: `name${i}`,
      id: i,
    }));
    ctx.body = {
      data,
      success: true,
    };
  }
}

module.exports = HomeController;
```

- `async`: 可以理解为约定，代表了异步。

`this` 内参数说明：

| 参数         | 说明                                                                                                                                |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| this.ctx     | 当前请求的上下文 Context 对象的实例，通过它我们可以拿到框架封装好的处理当前请求的各种便捷属性和方法(能够更好的拿到请求上下文的数据) |
| this.app     | 当前应用 Application 对象的实例，通过它我们可以拿到框架提供的全局对象和方法                                                         |
| this.service | 通过它我们可以访问到抽象出的业务层，等价于 `this.ctx.service`, service 用于服务端的底层封装                                         |
| this.config  | 应用运行时的配置项                                                                                                                  |

写完服务端的 `get` 请求，启动服务，打开 `http://127.0.0.1:7001/api/getUserList`，能够看到请求的数据已经显示在界面上。

![img](../gitImg/egg/egg3.jpg)

### web

开发 web 端代码，请求 `/api/getUserList` 接口。

`/src/pages/index/service.ts`: 使用 `useRequest` 请求参数。

```js
import { request } from 'alita';

export async function getUserList(): Promise<any> {
  return request('/api/getUserList', { method: 'get' });
}
```

`/src/pages/index/index.tsx`:

```js
import React, { FC, Fragment } from 'react';
import { useRequest, history } from 'alita';
import { Button } from 'antd-mobile';
import { getUserList } from './service';
import styles from './index.less';

interface HomePageProps {}

const HomePage: FC<HomePageProps> = () => {
  const { data, run } = useRequest(getUserList, {
    manual: true,
  });
  return (
    <div className={styles.center}>
      {data && (
        <Fragment>
          <div>调用 getUserList 请求： 出参：</div>
          {data.map((item: any) => (
            <div
              onClick={() => {
                history.push({
                  pathname: '/userDetail',
                  query: {
                    id: item.id,
                  },
                });
              }}
              key={item?.id}
              style={{
                height: '1rem',
              }}
            >
              {item.name}
            </div>
          ))}
        </Fragment>
      )}
      <Button onClick={() => run()}>get 请求</Button>
    </div>
  );
};
export default HomePage;
```

### 跨域问题

启动 web 项目访问：`http://localhost:8000/#/` 点击按钮调用接口发现数据并未展示出来，接口状态码 `304 Not Modified`。

![img](../gitImg/egg/egg4.jpg)

细看接口发现，服务接口和 web 并不同源，应该是存在跨域问题，我们需要增加代理来解决跨域问题：

`/config/config.js`

```js
import { defineConfig } from 'alita';

const baseUrl = 'http://127.0.0.1:7001';

export default defineConfig({
  appType: 'h5',
  mobileLayout: true,
  proxy: {
    '/api': {
      target: baseUrl,
      changeOrigin: true,
    },
  },
});
```

![img](../gitImg/egg/egg5.jpg)

至此，一个完整的 get 请求接口从开发，到 web 端界面渲染就完成了。

**[feat-user-list 分支](https://github.com/hang1017/eggDemo/tree/feat-user-list)**

## 三、用户详情接口开发(post 请求)

### egg

有了 `get` 请求的示例，`post` 实现同理。

`/app/router/detail.js`

```js
'use strict';

module.exports = app => {
  const { router, controller } = app;
  router.post('/api/getUserDetail', controller.home.userDetailController);
};
```

`/app/controller/home.js`

```js
...
class HomeController extends Controller {
  async userListController() { ... }

  async userDetailController() {
    const { ctx } = this;
    ctx.body = {
      data: 'userDetail query success',
      success: true,
    };
  }
}
...
```

因为 `post` 请求无法通过 url 实现接口调用，所以我们首要目的是成功调用接口：

### web

```bash
# 终端执行，使用命令行创建页面
alita g pages userDetail
```

`/src/pages/userDetail/service.ts`

```js
import { request } from 'alita';

export async function queryUserDetail(params: any): Promise<any> {
  return request('/api/getUserDetail', { method: 'post', data: params });
}
```

`/src/pages/userDetail/index.tsx`

```js
import React, { FC } from 'react';
import { useRequest } from 'alita';
import { queryUserDetail } from './service';
import styles from './index.less';

interface UserDetailPageProps {
  location: any;
}

const UserDetailPage: FC<UserDetailPageProps> = ({ location }) => {
  const { id } = location.query;
  const { data } = useRequest(() => queryUserDetail({ id }));
  return <div className={styles.center}>{data}</div>;
};

export default UserDetailPage;
```

在列表页点击任意一项，即可跳转到用户详情页。url 为 `http://localhost:8000/#/userDetail?id=1`

但是请求似乎并没有想象的那样成功。

接口`403`，报错信息 `{"message":"invalid csrf token"}`。

![img](../gitImg/egg/egg6.jpg)

通过百度得知框架内置了安全系统，默认开启防止 `XSS` 攻击 和 `CSRF` 攻击。参考[Egg post 失败 { message: 'invalid csrf token' } 解决方案](https://blog.csdn.net/weixin_43704471/article/details/90763103)可得到一下的解决方案

### egg

`/config/config.default.js` ：

```js
config.security = {
  csrf: {
    headerName: 'x-csrf-token', // 自定义请求头
  },
};
```

### web

给请求头 `headers` 增加 `x-csrf-token`：

`/src/utils/index.ts`

```js
// 封装获取 cookie 的方法
export const getCookie = (name: any) => {
  var arr,
    reg = new RegExp('(^| )' + name + '=([^;]*)(;|$)');
  if ((arr = document.cookie.match(reg))) return unescape(arr[2]);
  else return '';
};
```

`/src/app.ts`

通过中间件统一给请求接口增加 `header`

```js
const middleware = async (ctx: Context, next: any) => {
  // 可以在这写一些请求前做的事情 操作ctx.req
  ctx.req.options = {
    ...ctx.req.options,
    headers: {
      'x-csrf-token': getCookie('csrfToken'), // 前后端不分离的情况加每次打开客户端，egg会直接在客户端的 Cookie 中写入密钥 ，密钥的 Key 就是 'scrfToken' 这个字段，所以直接获取就好了
    },
  };
  await next();
  // 可以在这里对响应数据做一些操作 操作ctx.res
};
```

![img](../gitImg/egg/egg7.jpg)

### egg

最后我们在 `controller` 拿到接口请求的入参，重新封装出参的数据。

`/app/controller/home.js`

```js
async userDetailController() {
    const { ctx } = this;
    const { id } = ctx.request.body;
    console.log(ctx.request.body);
    ctx.body = {
      data: {
        id,
        name: `name${id}`,
      },
      success: true,
    };
}
```

当有调用到接口时，终端会显示 `ctx.request.body` 的数据。

**[feat-user-detail 分支](https://github.com/hang1017/eggDemo/tree/feat-user-detail)**

## 四、egg 中间件

写了中间件，所有请求都会经过中间件。可以在中间件进行操作处理。

可以参考官网[编写 Middleware](https://eggjs.org/zh-cn/intro/quickstart.html#%E7%BC%96%E5%86%99-middleware)示例。

这里将实现实现一个需求，在请求头 `header` 增加 `requestHeader` 的参数。接口侧在中间件进行校验 `requestHeader` 的正确性。若正确，则流程继续。若不正确，则无法请求接口。

### egg

在 `/app` 下新建 `/middleware` 文件夹。这也是约定，用来存放中间件。

`/app/middleware/request.js`

不着急实现功能，要先试试中间件是否增加成功。

通过下面的代码可以看出，若中间件实现成功，每当有接口请求，终端就会展示 `options`(config.default.js) 的参数，以及请求的入参参数。

```js
'use strict';
module.exports = options => {
  return async function requestMiddleware(ctx, next) {
    console.log(options.requestHeader, ctx.request.body);
    await next();
  };
};
```

`/config.config.default.js`

```js
module.exports = appInfo => {
    ...//这里省略部分代码

    config.middleware = ['request']; // 将 `request.js` 文件加入中间件中

    // 给 `request.js` 增加配置参数
    exports.request = {
        requestHeader: 'hang',
    };

    return {
        ...config,
        ...userConfig,
    };
};
```

访问 `http://localhost:8000/#/userDetail?id=2`

![img](../gitImg/egg/egg8.jpg)

那么说明中间件配置成功。开始来编写功能代码

`/app/middleware/request.js`

```js
module.exports = options => {
  return async function requestMiddleware(ctx, next) {
    const { requestHeader } = ctx.request.header;
    if (ctx.request.header.requestheader === options.requestHeader) {
      await next();
    } else {
      ctx.status = 403;
      ctx.body = {
        data: 'access was denied',
        success: false,
      };
    }
  };
};
```

![img](../gitImg/egg/egg9.jpg)

### web

若接口请求失败，在 web 中间件统一显示处理。

`/config/config.ts`

```js
export const request = {
  prefix: '', // 统一的请求头
  middlewares: [middleware],
  errorHandler: (error: ResponseError) => {
    if (error.response.status === 403) {
      Toast.fail('access was denied');
    }
  },
};
```

![img](../gitImg/egg/egg10.jpg)

`/config/config.ts`

```js
const middleware = async (ctx: Context, next: any) => {
  // 可以在这写一些请求前做的事情 操作ctx.req
  ctx.req.options = {
    ...ctx.req.options,
    headers: {
      'x-csrf-token': getCookie('csrfToken'), // 前后端不分离的情况加每次打开客户端，egg会直接在客户端的 Cookie 中写入密钥 ，密钥的 Key 就是 'scrfToken' 这个字段，所以直接获取就好了
      requestHeader: 'hang',
    },
  };
  await next();
};
```

web 接口请求统一增加 `header` `requestHeader` 后，接口又重新正常请求。

**[feat-middleware 分支](https://github.com/hang1017/eggDemo/tree/feat-middleware)**

## 五、访问 MySQL 数据库

![img](../gitImg/egg/egg11.jpg)

以上是新建的数据库内容，小伙伴可以自行搭建下。

### 1、连接数据库

根据[官网教程](https://eggjs.org/zh-cn/tutorials/mysql.html#egg-mysql)

在项目中安装对应的插件

```bash
npm i --save egg-mysql

or

yarn add egg-mysql
```

**开启插件：**

`/config/plugin.js`

```js
'use strict';

module.exports = {
  mysql: {
    enable: true,
    package: 'egg-mysql',
  },
};
```

**配置数据库信息**

`/config/config.default.js`

```js
'use strict';

/**
 * @param {Egg.EggAppInfo} appInfo app info
 */
module.exports = appInfo => {

  ... // 忽略重复代码

  config.mysql = {
    client: {
      // host
      host: 'localhost',
      // 端口号
      port: '3306',
      // 用户名
      user: 'root',
      // 密码
      password: '******',
      // 数据库名
      database: 'BMS',
    },
    // 是否加载到 app 上，默认开启
    app: true,
    // 是否加载到 agent 上，默认关闭
    agent: false,
  };

  return {
    ...config,
    ...userConfig,
  };
};

```

那么至此我们的数据库配置就已经完成。

### 2、查询数据库

web 列表页中的 `user` 数据是在 `egg` 上写死的。

现在创建了 `user` 表，就通过查表来获取数据。

`/app/controller/home.js`

修改 `userListController` 的方法：

```js
 async userListController() {
  const { ctx, app } = this;
  const sql = 'select * from user'; // sql 语句

  // 这句是重点，圈起来要考
  const res = await app.mysql.query(sql); // 查询 sql 数据

  ctx.body = {
    data: res,
    success: true,
  };
}
```

通过 web 界面，可以确信数据库的数据是被我们查询出来没错了。

![img](../gitImg/egg/egg12.jpg)

完成了列表页，就顺手把 `userDetail` 的查询一起实现了吧。

```js
async userDetailController() {
  const { ctx, app } = this;
  const { id } = ctx.request.body;
  const sql = `select * from user where id=${id}`;

  const res = await app.mysql.query(sql);
  ctx.body = {
    data: res,
    success: true,
  };
}
```

![img](../gitImg/egg/egg13.jpg)

**[feat-mysql 分支](https://github.com/hang1017/eggDemo/tree/feat-mysql)**

## 六、Service 层

将请求数据库的代码提取出来，封装成 `service` 层。

这样做有几个好处：

- 保持 `Controller` 中的逻辑更加简洁
- 保持业务逻辑的独立性，抽象出来的 `Service` 可以被多个 `Controller` 重复调用

在 `Controller` 层，我们专注于业务逻辑处理，`Service` 层就只做最基本的数据存取的功能。

这里有几个细节要注意：

- 文件必须要在 `/app/service` 下(约定优于配置)
- 一个 Service 文件只能包含一个类， 这个类需要通过 `module.exports` 的方式返回

新建文件 `/app/service/user.js`:

```js
'use strict';

const Service = require('egg').Service;

class UserService extends Service {
  async userList() {
    const res = await this.app.mysql.query('select * from user');
    return res;
  }

  async userDetail(id) {
    const res = await this.app.mysql.query(`select * from user where id=${id}`);
    return res;
  }
}

module.exports = UserService;
```

代码很好理解，就是把 `Controller` 层操作数据库的代码迁移到 `Service` 来。

修改下 `Controller` 层的代码，通过 `Service` 操作数据库。

`/app/controller/home.js`

```js
async userListController() {
  const { ctx, service } = this;
  const res = await service.user.userList();
  ctx.body = {
    data: res,
    success: true,
  };
}
```

同理，小伙伴可以自行操作下 `userDetailController`的方法。

代码编写好后访问 `web` 端的页面，能正常请求到数据就说明没问题。

**[feat-service 分支](https://github.com/hang1017/eggDemo/tree/feat-service)**
