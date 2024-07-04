# 强大的移动端表单开发方案 @alitajs/dform

## 一、开发设想

从事移动端 H5 开发的小伙伴有没有经历过被长表单支配的恐惧？

是一个个表单项纯手写实现的吗？那一个页面你可能要做一天，欲哭无泪，心力憔悴。为什么表单实现这么难？

是使用 `antd-mobile` 的 `rc-form` 吗？确实会让你提效 `50%`，但一个个的设置 `getFieldProps` 和 `initialValue` 的回填赋值，又感觉做了很多重复的动作。

如果表单中存在 `radio`, `check` 这类 `antd-mobile` 库里没有的样式，那我们还要自己实现UI吗？

再来一种情况，如果是后台提供的动态表单呢？心态崩了，还要自己遍历实现UI，提交取值，回填赋值等需求。

那么能否有一种针对移动端长表单的快捷实现方案呢？比使用 `rc-form` 还要提效 `50%`，方案要覆盖：

- UI 的快速实现
- 能实现一次性全部赋值
- 表单提交取值
- 融合多类型组件表单
- 支持动态表单

针对以上需求，我们开发 `@alitajs/dform` 支撑表单方案，并且在公司内部数十个项目中得到锤炼，不断优化完善。

好了那么是骡子是马拉出来溜溜！

## 二、基础使用

我们借鉴了 `antd@4` 的 `Form` 组件，针对表单使用的 `react-component/field-form` 库进行二次封装。不了解的小伙伴也没事。

想想表单需要哪些属性和事件？

- 表单全部字段
- 表单提交成功事件
- 表单提交失败事件
- 表单赋值回填的字段

```js
import React from 'react';
import DynamicForm, { useForm } from '@alitajs/dform';
import { Button } from 'antd-mobile';

const Page = () => {
  const [form] = useForm(); // 定义 form

  const onFinish = values => console.log('Success:', values);

  const onFinishFailed = errorInfo => console.log('Failed:', errorInfo); 

  const data = [
    {
      type: 'input',
      fieldProps: 'username',
      placeholder: '请输入',
      title: '用户名',
      required: true,
    },
  ];

  const formProps = {
    form, // 表单定义
    data, // 表单全部字段
    formsValues: {}, // 表单赋值回填数据
    onFinish, // 表单提交成功事件
    onFinishFailed, // 表单提交失败事件
  };

  return (
    <>
      <DynamicForm {...formProps} />
      <Button onClick={() => { form.submit();}}>Submit</Button>
    </>
  );
};

export default Page;
```

这个最简 demo 里核心代码应该就 `10` 行。`data` 是数组，要增加表单字段只需要简单的，往 `data` 里添加字段即可。

`dform` 共提供 `15` 种组件。涵盖：

- 文本展示类型: `text`
- 输入类型: `input` 和 `area`
- 选择类型: `picker` 和 `select`
- 多选类型: `multiplePicker`
- 开关类型: `switch`
- 时间选择类型: `date`
- 图片选择类型: `image`
- 选择地址类型: `addressPicker`
- Radio按钮类型: `radio` 和 `coverRadio`
- Check多选类型: `check`
- 时间区间选择类型: `rangeDatePicker`
- 高阶输入类型: `extraInput`

如果这么多的组件还不能满足需求，不着急。我们还提供 `自定义类型: custom` 组件，让用户自己实现，并在文档中提供教程。或者给我们提个 [issues](https://github.com/alitajs/DynamicForm/issues)，我们会根据评估结果进行开发和维护。

## 三、提效点

### 1、`picker` 组件：

`antd-mobile` 提供的 `Select` 组件涵盖了及联的类型，所以 `value` 出参以 `[]` 的形式。

但是在表单对象走接口时，每个字段的值很大情况下都是 `stirng` 或者 `number` 的形式进行传递，在 `[]` 情况下，还要对数据结构进行处理。

`dform` 提供了四种选址组件:

- `picker`: 单选类型，出参为 `string` 或者 `number`，不再需要对数据结构进行多一层的转化。
- `select`: `antd-mobile` 上的 `Select` 组件，出参入参设值保持一致。
- `multiplePicker`: 多选，出参以 list 的形式提供。
- `addressPicker`: 选址，更是帮你大大的提效(**舒服的写业务吧，剩下的事情交给我们**)。

### 2、一行代码配置样式

不同的项目，不同的 ui设计师，针对表单的开发样式肯定不一样，比如：

- 标题的颜色和大小
- 值的颜色和大小
- placeholder 颜色
- ...

在 `.umirc.ts` 和 `config.ts` 下配置：`theme`

![theme](https://img-blog.csdnimg.cn/20200702171633257.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODk3OQ==,size_16,color_FFFFFF,t_70)

一行代码帮你解决整个项目 `dform` 样式问题。**不香吗？**

### 3、不敲一行代码帮你配置 `data` 的JSON数据

如果你连 `JSON` 格式的 `data` 也懒得写，那么 `isDev` 字段开启开发者模式，让你鼠标点一点就能编辑好一串 `JSON`，视频会告诉你用起来多舒服。

<video src="https://v.qq.com/x/page/u3108c1l2o8.html" controls="controls" width="500" height="300">您的浏览器不支持播放该视频！</video>

gitHub: https://github.com/alitajs/DynamicForm（请给个star）

官网文档: https://dform.alitajs.com/























