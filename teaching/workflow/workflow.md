# gitHub Action - workflow 概念和基本操作

## 一、workflow 文件

本文编写参考：

- [GitHub Actions 入门教程 - 阮一峰](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)
- [GithubAction---Workflow 概念和基本操作](https://zhuanlan.zhihu.com/p/377731593)

`github action` 的配置文件叫做 `workflow` 文件，存放在 `.github/workflows` 目录下。

文件采用 `.yml` 或 `.yaml` 的后缀格式。

一个库可以有多个 `workflow` 文件，只要有符合格式的后缀文件就会自动运行该文件。

## 二、基本概念和术语

- workflow(工作流程)：持续集成一次运行的过程，就是一个 workflow。
- job(任务)：一个 workflow 由一个或者多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。
- step(步骤)：每个 job 由多个 step 构成，一步步完成。
- action(动作)：每个 step 可以依次执行一个或者多个命令。

workflow 文件的配置字段很多，详见[官方文档](https://docs.github.com/cn/actions/using-workflows/workflow-syntax-for-github-actions)。下面是一些基本字段。

### 1、name

workflow 的名称，如果省略该字段，默认为当前 workflow 的文件名。

```yml
name: GitHub Actions Demo
```

### 2、on

指定触发 workflow 的条件，通常是某些事件。

```yml
# push 事件触发 workflow
on: push

# 可以是事件的数组
on: [push, pull_request]

# 指定触发事件时，可以限定分支或标签
on:
  push:
      branches:
       - master
```

### 3、jobs

workflow 的主体就是 job 字段，表示要执行的一项或者多项任务。

#### jobs.<job_id>.name

```yml
jobs:
  my_first_job:
    name: My first job
  my_second_job:
    name: My second job
```

上面代码 `jobs` 包含两个任务，`job_id` 分别是 `my_first_job`、`my_second_job`。

`job_id` 里面的 `name` 就是任务的说明。

#### jobs.<job_id>.needs

指定当前任务的依赖关系，运行顺序。

```yml
jobs:
  job1:
  job2:
    needs: job1
  job3:
    needs: [job1, job2]
```

job1 必须先于 job2 完成，job3 等待 job1 和 job2 的完成才能运行。所以这个执行顺序是 job1、job2、job3。

#### jobs<job_id>.runs-on

指定运行所需要的虚拟机环境。**必填项**

```yml
ubuntu-latest，ubuntu-18.04或ubuntu-16.04

windows-latest，windows-2019或windows-2016

macOS-latest或macOS-10.14
```

#### jobs<job_id>.steps

指定每个 job 的运行步骤，可以包含一个或者多个步骤。

- jobs.<job_id>.steps.name：步骤名称。
- jobs.<job_id>.steps.run：该步骤运行的命令或者 action。
- jobs.<job_id>.steps.env：该步骤所需的环境变量。

```yml
name: Greeting from Mona
on: push

jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest
    steps:
      - name: Print a greeting
        env:
          MY_VAR: Hi there! My name is
          FIRST_NAME: Mona
          MIDDLE_NAME: The
          LAST_NAME: Octocat
        run: |
          echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```

### 模拟实例

模拟一个场景，当某个分支发生 push 事件后，会构建整个项目，并将部署后的产物推送到某个我们配置好的链接下

参考 [alita 主库的配置](https://github.com/alitajs/alita/blob/master/.github/workflows/deploy.yml)

```yml
env:
  NODE_OPTIONS: --max-old-space-size=6144
```

设置 Node 的内存上限。

```yml
- name: "Checkout"
  uses: actions/checkout@v3
  with:
    fetch-depth: 0
```

在 job 的步骤中，第一步 `action/checkout` 意在获取源码。

`uses`: 使用一些官方或者第三方的 actions 来执行。

```yml
- name: "Setup Node.js"
  uses: actions/setup-node@v3
  with:
    node-version: 14
```

在当前操作系统安装 node

```yml
- name: install
  run: yarn

- name: Build
  run: yarn run docs:build
```

安装以来后执行打包命令

```yml
- name: Deploy
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./docs-dist
    cname: dform.alitajs.com
```

- `publish_dir`: 打包后生成的包
- `cname`: 配置好的文档链接
