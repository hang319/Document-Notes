# 使用 nodeJs 实现 js/ts 文件翻译功能

关于实现翻译(已中翻英为例)的功能，我们可以分成几个步骤：

- 读文件
- 找出文中的中文
- 将文中的出现的中文词组或者句子组成一哥数组
- 调用公共的翻译接口进行异步翻译
- 将翻译出来的英文回填会文件中

## 一、搭建环境

打开终端，键入：`node -v`

如果出现 `-bash: node: command not found`

说明 `node` 的环境没有搭建。请到[官网](http://nodejs.cn/download/)上下载。

如果终端上出现：`v10.16.0` 类似这样的版本号，说明你的 `node` 环境搭建好啦。

## 二、读文件

我们先读取固定的文件，后面我们会尝试实现翻译上传的文件，有需要的小伙伴可以移步到后面的内容。

首先我们在文件夹下创建一个 `replace.js` 文件用来编写我们的代码。

其次创建一个供我们翻译的 `js/ts` 文件，如：`DesUtils.ts`，代码如下：

```js
const desKeyObj = {
  desKey: 'ztesoftbasemobile20160812..',
  ivKey: '01234567890'
}
export default {
  /**
   * 加密
   * @param dataStr
   */
  encrypt: function (dataStr) {
    try {
        console.log("如果这是一段话。并且还有标点符号。");
    } catch (error) {
        console.log('加密报错' + JSON.stringify(error));
    }
  },
  /**
   *
   * @param dataStr 解密
   */
  crypto: function (dataStr) {
    try {
        console.log('');
    } catch (error) {
        console.log('解密报错')
    }
  }
}
```

现在我们开始编写 `replace.js` 文件，来读取 `DesUtil.ts` 文件。

首先我们可以使用 `require` 命令来引入我们需要的模块。

```js
var fs = require('fs');
```

`node` 对于文件的读写操作是基于 `Stream` 流的形式。

`fs` 模块下有四种流类型：

- Readable - 可读操作
- Writable - 可写操作
- Duplex - 可读可写操作
- Transform - 操作被写入数据，然后读出结果

对于流的操作也有四种：

- data - 当有数据可读时触发
- end - 没有更多的数据可读时触发
- error - 在接收和写入过程中发生错误时触发
- finish - 所有数据已被写入到底层系统时触发

是不是看了有点懵呢，不着急，通过下面的代码你就能理解。

创建可读流，并处理流事件：

```js
// 创建可读流
let readerStream = fs.createReadStream('DesUtils.ts');
readerStream.setEncoding('UTF8');

let data = '';

// 处理流
readerStream.on('data', (chunk) => {
   data += chunk;
});

readerStream.on('end',() => {
   console.log(data);
});

// 可不写
readerStream.on('error', (err) => {
   console.log(err.stack);
});
```

![img](gitImg/nodeReplace/1-1.jpg)

读文件的代码我们已经写完了，现在我们尝试运行一下：

终端进入到该文件夹下，键入：`node replace.js`, 不出意外的话文件的内容已经完整的展示在终端上了。

## 三、列出文中的所有中文内容

在 `data` 上进行数据的处理，我们要通过正则表达式将文中的中文取出来。在上面的代码上进行修改：

```js
readerStream.on('data', (chunk) => {
  const reg = /[\u4e00-\u9fa5]/g;
  while(list = reg.exec(chunk)) {
    console.log(list[0]);
  }
})
```

![img](gitImg/nodeReplace/1-2.jpg)

好了现在我们已经把所有的中文全部取出来了，如果我们的需求是删除中文的话，我们只需要将这些中文删除即可。

如果是翻译成英文的话，总不能一个字一个字的翻译吧？

所以需要我们将这个中文分类出来，该是词组的就存成词组，该是句子的就存成句子，用一个数组来进行存放。

我的方式是通过每个字的索引进行判断。获取到中文与前一个中文索引相比是否是相邻关系，如果是，就不能拆开来，应该组成一个词语或者句子进行存储(可能小伙伴看不懂，你们直接分析代码吧～)

如果有更优的方法，小伙伴可以自己尝试哈～

```js
readerStream.on('data', (chunk) => {

  var reg = /[\u4e00-\u9fa5]/g;
  let index = 0;
  let termList = [];    // 遍历获取到的中文数组
  let term = '';
  data = chunk;

  while (list = reg.exec(chunk)) {
    if ((list.index !== index + 1) && term) {
      termList.push(term);
      term = '';
    }
    term += list[0];
    index = list.index;
  }
  termList.push(term);
  console.log(termList);    // 打印中文数据
})
```

![img](gitImg/nodeReplace/1-3.jpg)

通过上面的代码，我们就已经把词组，句子等区分出来了。终端上就能看到数据 `[ '加密', '如果这是一段话', '并且还有标点符号', '加密报错', '解密', '解密报错' ]`。

翻译的功能我们放到下个模块去说，我们现在先把获取到的中文内容替换成随意的内容。

## 四、替换中文

创建一个可以写入的流，将查询出来的中文替换成随便的字符。

```js
// 创建一个可以写入的流，写入到文件 replaceDesUtils.ts 中
let writerStream = fs.createWriteStream('replaceDesUtils.ts');

readerStream.on('data', (chunk) => {

  var reg = /[\u4e00-\u9fa5]/g;
  let index = 0;
  let termList = [];    // 遍历获取到的中文数组
  let term = '';

  while (list = reg.exec(chunk)) {
    ...省略
  }
  if(termList && termList.length) {
    termList.map(item => {
      data = data.replace(item, '112233');      // 至此，已经将中文全部进行替换
    })
  }

  writerStream.end(data); // 将替换后的文件写入到 replaceDesUtils.ts 中去
})
```

现在就可以发现当前包下多了一个 `replaceDesUtils.ts` 文件，快打开看看是否成功了吧。

## 五、实现翻译功能

在调用翻译的接口上，参考了[基于nodejs实现一个有道词典翻译器](https://www.jianshu.com/p/4d5086f0dc52) 百度上的一个教程文档，小伙伴们可以做参考。

我粗略说下创建的步骤(里头有50元的额度，够我们写 demo 测试用就是了～)：

- 打开[有道智云](https://ai.youdao.com/)注册下账号
- 登录后在左边的 `tab` 上找到 `自然语言翻译-翻译实例`，创建新的实例，选择`文本翻译`(或者你可以选择其他的类型，我只尝试了`文本翻译`)
- 打开左边 `tab` 上`应用管理-我的应用`，创建新的应用，`接入类型` 选择 `API`, 点击下一步后绑定刚创建的实例。
- 查看应用详情里就有 `应用ID` 和 `应用密钥` 这两个字段，分别对应一会需要用到的 `appKey` 和 `secretKey`.

![img](gitImg/nodeReplace/1-4.jpg)

接下来在该文件夹下创建一个 `translator.js` 文件，直接 copy [基于nodejs实现一个有道词典翻译器](https://www.jianshu.com/p/4d5086f0dc52)这里的代码就行。

![img](gitImg/nodeReplace/1-5.jpg)

但是这个文件有一些需要用到的库我们没有。需要我们手动安装一下，步骤如下：

- 1、在当前包下创建一个 `package.json` 文件，复制一下的代码：

```json
{
  "dependencies": {
    "request": "^2.88.0",
    "request-promise": "^4.2.5"
  }
}
```

- 2、然后在终端上执行：`npm i` 或者 `yarn`，装上我们需要的库。

好了继续在 `replace.js` 编写我们的代码：

- 引入翻译的文件，并实例化
- 设置翻译的配置
- 编写公共方法，进行翻译(小伙伴如果对 `Promise` 不太理解我下文有解释啊哈～)
- 将中文进行翻译并写入文档中

```js
var Translator = require('./translator');

let translator = new Translator();

translator.config = {
  from: 'zh_CHS',
  to: 'EN',
  appKey: '*********',
  secretKey: '****************'
}

function translateString(str) {
  return new Promise(function (resolve, reject) {
    let resultStr = translator.translate(str);
    resolve(resultStr);
  })
}
```

这里先解释一下 `translateString()` 这个方法，因为我们调用的翻译接口是异步的，如果不使用 `Promise` 会存在延迟问题。

`Promise` 提供了一种异步执行模式。

其提供的两个参数可以理解为 `return`, 通过 `resolve` 将需要的数据传递出去。这里我们来个小小的测试

```js
translateString('加油').then(val => {
  console.log(val);
})
```

![img](gitImg/nodeReplace/1-6.jpg)

出来的数据是一大长串 `JSON` 字符串，我们对此进行转义和截取下。`console.log(JSON.parse(val).translation)`

是不是发现翻译后的内容已经被我们获取到了。

现在我们继续来编写代码：

```js
while (list = reg.exec(chunk)) {
  ...省略
}
termList.push(term);
console.log(termList);    // 打印中文数据
translateString(termList).then(val => {
  let translation = JSON.parse(val).translation;  // 这里可以打印一下数据，翻译出来的内容，就可以知道为什么下文的代码要那样写了。
  let transList = [];     // 翻译后的数组

  if(translation[0] && translation[0].indexOf(',') !== -1) {

    transList = translation[0].split(', '); // 逗号后面有个空格不要漏了哈
    transList.map((item, index) => {
      console.log(termList[index], item);
      const res = data.replace(termList[index], item);
      data = res;
    })

    writerStream.write(data, 'UTF8');
    writerStream.end();
    console.log('写入完成');
  }
})
```

![img](gitImg/nodeReplace/1-7.jpg)

![img](gitImg/nodeReplace/1-8.jpg)


好了小伙伴们，现在我们打开 `replaceDesUtils.ts` 看看效果哈。

后面的文章会用到 `express` 进行文本操作，可以上传自定义的文件进行翻译。





