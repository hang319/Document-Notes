# React技术2-集成 less 和 css-module

## 前沿

我们沿着上一篇文章的内容进行集成 less 和 css mmodule。

现在应该 `90%` 的前端开发都会使用 `.less` 文件来进行样式编写。可能对于 `css module` 有些陌生。那么就要先来讲解一下使用使用这两个技术的好处了。

在一个多人协同开发的大型项目中，是否会经常因为使用到相同的样式名称(css 命名冲突)而导致的样式错乱，这种问题被称为 `全局样式污染`。

那么怎么解决这个问题呢？

相信很有效的解决办法就是给每个命名都加上前缀，每个文件或者模块都固定一个前缀来保证命名的单一性。

一份较长的 `.css` 文件每个样式层级一样，实在是让人难以看出其中的关联。

那么笔者认为 `less` 和 `css module` 能够很好的解决这个问题。

- `less`：能够进行层级嵌套，能够较直观的让同事看清楚你的样式代码(哪一层包裹哪一层),没有被包裹起来的样式就不会对产生效果。
- `css module`：会对样式名称进行编译，从而保证命名的单一性。

好了讲了这么多，我们继续上个章节的代码开始编辑吧。

## 一、导入 webpack

要使用 `less` 就要在 `webpack.config.js` 的 配置文件下进行编辑。

所以第一步输入以下命令：

```bash
yarn eject

? Are you sure you want to eject? This action is permanent. Yes

This git repository has untracked files or uncommitted changes:

package.json
D src/App.css
D src/App.js
D src/App.test.js
M src/index.css
M src/index.js
M yarn.lock
src/home/
src/index/
src/router.js

Remove untracked files, stash or commit any changes, and try again.
error Command failed with exit code 1.
```

当我们输入 `yarn eject` 并确认后会告诉我们安装错误, `This git repository has untracked files or uncommitted changes:` 这句话的意思是 `此git存储库包含未跟踪的文件或未提交的更改`。

所以我们在执行这条命令之前要先将项目提交至git 仓库(小伙伴们自行操作)

![img](../../gitImg/react/less1.jpg)

新建完仓库后，按着上面步骤走完命令就可以拉。执行完命令，我们仓库就算搭建好了。

继续执行以下命令提交项目中所有代码：

```bash
git add .
git commit -m "demo"
git pull origin master
git push origin master
```

接下来我们只要把 `yarn eject` 重新执行一遍就可以拉。执行完命令后打开 `config` 文件夹就可以看到以下的新增文件。

![img](../../gitImg/react/less2.jpg)

## 二、导入 less 

```bash
yarn add less

yarn add less-loader
```

执行完后，我们打开 `config/webpack.config.js` 文件

共三步：

1、添加 less 规则

```js
const lessRegex = /\.less$/;
const lessModuleRegex = /\.module\.less$/;
```

![img](../../gitImg/react/less3.jpg)

2、添加 less 配置

```js
{
  test: lessRegex,
  exclude: lessModuleRegex,
  use: getStyleLoaders(
    {
      importLoaders: 2,
      sourceMap: isEnvProduction && shouldUseSourceMap,
    },
    'less-loader'
  ),
  sideEffects: true,
},
{
  test: lessModuleRegex,
  use: getStyleLoaders(
    {
      importLoaders: 2,
      sourceMap: isEnvProduction && shouldUseSourceMap,
      modules: true,
      getLocalIdent: getCSSModuleLocalIdent,
    },
    'less-loader'
  ),
},
```

![img](../../gitImg/react/less4.jpg)

好了，接下来我们重启项目，在 `src/index` 文件夹下新建 `index.less` 目录。

开始我们的代码编写（缩写版）：

`src/index/index.js`

```js
import './index.less';
// import './index.css';

return <div className='indexStyle'>
    <p className='text'>this is index</p>
    <Button onClick={goToHome}>跳转到 Home</Button>
  </div>
```

`src/index/index.less`

```css
.indexStyle {
  .text {
    color: blue;
  }
}
```

来看看效果：

![img](../../gitImg/react/less5.jpg)

好，这里我们来个小小测试：

`src/index/index.less`

```css
.indexStyle {}

.homeStyle {
  .text {
    color: blue;
  }
}
```

我们的 `text` 在 `js` 中是被 `indexStyle` 的样式包裹的，如果我们切换成别的样式包裹，小伙伴们看看效果，是不是发现会失效呢。

所以我们在编写页面的时候每个页面最外层都用页面名称进行最外层的包裹，就能够保证本页面的样式命名不会去影响到其他页面的样式。是不是很好的解决了命名冲突的问题。

![img](../../gitImg/react/less6.jpg)

## 四、使用 `css-module`

继续打开 `webpack.config.js`

```diff
{
  test: lessRegex,
  exclude: lessModuleRegex,
  use: getStyleLoaders(
    {
      importLoaders: 2,
      sourceMap: isEnvProduction && shouldUseSourceMap,
+     modules: true,
    },
    'less-loader'
  ),
  sideEffects: true,
},
```

**重新启动项目，会发现样式失效了。**

所以我们来重新修改以下 `src/index/index.js` 下的代码(有省略)：


```js
import styles from './index.less';

return <div className={styles.indexStyle}>
  <p className={styles.text}>this is index</p>
  <Button onClick={goToHome}>跳转到 Home</Button>
</div>
```

![img](../../gitImg/react/less7.jpg)

这是被编译过的 css 命名。

那么小伙伴们可能会问了，什么情况下开启 `css-module`？

我们业务项目中，都会开启 `css module`，来避免样式污染的问题。

如果我们在开发组件库的情况下，那么别人可能会修改我们的底层代码，如果组件库的样式被编译了，那么使用者就根本无法得知组件下的命名，所以开发组件库时，我们会关闭 `css module` 并且会在每个样式增加前缀来保证不会和业务组件命名起冲突。

## 五、在 css module 下修改组件库的底层样式

```css
:global {}
```

`css module` 开启后，要修改底层样式外面要使用 `:global` 包裹才会生效哦。













