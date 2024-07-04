# keep alice 

沿用项目中 dform、list、cartlist 页面，来实现 keepalive

`config/config.ts`

```diff
export default {
  appType: 'h5',
  mobileLayout: true,
  theme: {
    '@alita-dform-title-font-size': '0.28rem',
    '@alita-dform-title-color': 'gray',
  },
+ keepalive: ['/list', '/dform'],
};
```

增加完 `config` 配置，项目重新启动下。

我们从首页进入 `/list` 加载更多页面，往下滑动页面。回退页面再重新进入 `/list` 页面，保持回退前的位置不变。

当我们需要清除 `/list` 页面的 keepalive,可以从 `alita` 导出 `dropByCacheKey`,

```js
dropByCacheKey('/list');
```

即可实现取消 keepalive 操作

> 注意，keepalive 的配置项，支持正则表达式。但是所有的路由正则匹配应该是全小写的，比如不管你的路由是 `home`、`Home` 还是 `hoMe` ，只有设置 `keepalive:[/home/]` 才有效。而字符串的配置方式就刚好相反，如果你的路由是`home`，你配置 `home`、`Home` 还是 `hoMe` 都有效。

