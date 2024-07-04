# 使用 nodeJs 的 express 框架实现 js/ts 文件翻译功能

我有写了一篇不基于 `express` 框架的文档实现翻译功能.

若你对 `node` 还不是很熟悉，请先快速浏览下文章。 

- 搭建环境
- 搭建服务，并完成附件上传的界面
- 读取附件中的内容
- 获取附件中的中文
- 对中文进行翻译
- 将翻译后的内容回填到附件上

## 搭建环境

若需要安装 `node` 请移步[官网](http://nodejs.cn/download/)

首先创建 `expressReplace.js` 文件。其次创建 `package.json` 文件。

```json
{
  "dependencies": {
    "body-parser": "^1.19.0",
    "express": "^4.17.1",
    "multer": "^1.4.2",
    "request": "^2.88.0",
    "request-promise": "^4.2.5"
  }
}
```

因为我们需要引入一些库来使用，当然小伙伴们也可以自己 `yarn add` 或者 `npm i`.

写好 `package.json` 后在终端上执行 `yarn`。




