#### 1、说一下 proxy 代理

- key 的作用是什么：识别作用
- changeOrigin： 控制代理请求时 Host 头的来源，常用于本地开发跨域代理。
- pathRewrite：对别名进行替换

#### 2、如何解决 css 污染

- 命名的规范性
- 通过less 或者 scss 实现样式层级

#### 3、说一下 v-model 的原理，现在需要你写一个 父子组件的 v-model，说一下思路。

父子组件中，实现双向数据绑定的语法糖。

如果没说到子组件的 `model` 可以问下这里面需要写什么？

`props: 'value'` 和 `event: 'change'`。

`value` 还需要在哪里注册一次？`props`

`change` 怎么使用 `$emit('change')`

#### 4、删除对象用delete和Vue.delete有什么区别？

- delete：只是被删除对象成员变为' '或undefined，其他元素键值不变
- Vue.delete： 直接删除对象成员，保证触发更新视图