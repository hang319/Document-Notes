# alita 初级使用教程

## 一、页面增加 tabLayout

1、检查配置

alita mobile 项目支持配置 `tabBar`，但首先要确保是否开启 `mobileLayout` 的开关。

打开 `config/config.ts` 文件，检查是否有以下代码，若不存在，则手动增加下：

```js
mobileLayout: true,
```

2、业务代码

打开 `/src/app.ts` 文件。

在默认导出的 `mobileLayout` 对象中，存在 `tabBar` 属性，用于配置 `tabLayout`。可在[alita 官网](https://alitajs.com/components/alita-layout) 中看到使用教程。

- `color`: 未选中时的文字颜色
- `selectedColor`: 选中时的文字颜色
- `position`: `tabBar` 的位置，仅支持 `bottom` / `top`
- **`list`**: 数据源。

`list` 数据源以对象数组的形式存在，进一步确认对象里的参数：

- `pagePath`: 页面路径
- `text`: 文字内容
- `iconPath`: 未选中时的图标
- `selectedIconPath`: 选中时的图标
- `iconSize`: 图标大小，默认为 `0.44rem`，
- `badge`: 右上角的红点内容，若无数据则不展示。

以上，成功完成 `tabLayout` 的搭建。

## 二、dform 动态表单

在 `tabLayout` 的首页，完成 `dform` 动态表单的编写。

[dform动态表单官网文档](https://dform.alitajs.com/)

1、命令行

使用前，需要先引入 `dform` 包。在终端上输入以下命令：

```bash
yarn add @alitajs/dform / npm i @alitajs/dform
```

2、代码实现

命令安装好后打开 `/src/pages/index` 的文件进行代码编写

从 `@alitajs/dform` 中引入 `DynamicForm, { useForm }`，

`DynamicForm` 可包含以下属性，若要查看更完整api，可查看[dform文档-api](https://dform.alitajs.com/#api)

- `form`: 从 `useForm` 中引入，定义表单
- `data`: 数据源，表单全部字段
- `onFinish`: 表单提交成功事件
- `onFinishFailed`: 表单提交失败事件
- `formsValues`: 表单赋值回填
a
3、全局样式赋值

表单的字段字段标题颜色大小、值颜色大小、以及不可编辑时的颜色等，都可通过全局配置实现一行代码修改，可查看[dform文档-配置项](https://dform.alitajs.com/setting#%E4%BA%8C%E3%80%81%E8%87%AA%E5%AE%9A%E4%B9%89%E5%B1%9E%E6%80%A7)。

在 `config/config.ts` 文件下编写：

```js
theme: {
  '@alita-dform-title-font-size': '34',
  ...
}
```

`useForm` 提供了几种方法，下面列举几个常用的方法：

- `form.getFieldValue('')`: 获取个别字段的值
- `form.getFieldsValue()`: 获取全部字段的值
- `form.setFieldsValue({})`: 给字段赋值
- `form.getFieldsError()`: 获取所有格式错误的字段

完整最简代码如下：

```ts
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
    formsValues: {
      username: '小明',
    }, // 表单赋值回填数据
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

