---
title: 借助 Travis CI 使 Hexo 变得更好
date: 2019-04-13 21:27:03
categories:
 - 效率
tags:
 - Hexo
 - Travis CI
---

对，我懒得写博客，却喜欢折腾，这可能是病...
OK，最近又换了电脑，每次换电脑都要迁移。

暂时进入回忆状态：->
最开始玩博客我用的是 WordPress，后来觉得 Hexo 比较好玩，遂换成 Hexo，但是 Hexo 依赖 Node，这就导致一个问题，每次换电脑都需要重新配置一下环境，换成 Hexo 之后保守估计我已经迁移过最少三次了。
不过今天，我动摇了。一个没有技术含量的操作如果需要重复三次以上那可能就是时候考虑让程序帮我们去重复这个操作了。

所以，有了这篇文章...

#### 使用自动构建之前首先要了解手动构建的步骤

既然要自动化，那么首先就必须得了解非自动化的流程，所以我们先来回顾一下 [使用 Hexo 构建博客](https://hexo.io/zh-cn/index.html) 的流程：

``` shell
// 一个新的 Hexo 博客从创建到部署到 github 只需要简单几步

$ npm install hexo-cli -g   // 全局安装 Hexo
$ hexo init blog            // 以 blog 文件夹为根目录初始化一个 Hexo 的博客
$ cd blog                   // 进入到博客根目录
$ npm install               // 安装 Hexo 的依赖
$ hexo new [name].md        // 创建一篇新的博文
$ hexo server               // 本地测试
$ hexo generate             // 生成网站静态文件
$ hexo deploy               // 部署网站到服务器
```

当然，以上流程只是非常简单的步骤，理论上如果要将 Hexo 部署到 github 还需要一个 `hexo-deployer-git` 插件，你可以在 blog 目录执行 `npm install hexo-deployer-git --save` 来安装它。更多关于 deploy 的信息可以来 [这里](https://hexo.io/docs/deployment.html) 查看。

从上面流程我们可以看出来一篇新的博文就从 `hexo new [name]` 开始，执行完这个命令之后我们就会在 source 文件夹下看到 `[name].md` 这个文件。

这样的话也就是说，我们可以把除了 `hexo new` 这个操作放到本地之外其余所有的操作都可以借助 Travis CI 来完成。由此带来的结果就是我们本地其实只需要维护所有博文的 `.md` 文件集合即可。

同时我们还应该注意到 `_config.yml` 文件，这个文件的内容将表现出我们的博客呈现一种什么样的主题、博客的名称、一些个性化的设置等等，因为大概不会有人会使用所有默认的配置。

最后如果说你还有自已的的域名，那么还会有一个 `CNAME` 文件，这个文件其实也是在 source 文件夹当中，伴随着 `hexo generate` 生成静态网页源文件时会自动拷贝到public文件夹中的。

#### 把可以转换成自动操作的步骤交给自动化服务去处理

我们知道 [Travis CI](https://zh.wikipedia.org/zh-cn/Travis_CI) 是一个可以在线进行代码构建的持续集成服务，所以我们把这些步骤交给它去做。

在此之前我的 github 有两个仓库，一个是 `agenric.github.io` 用来存放静态网站的源文件，另一个是 `backup-blog` 用来跟我本地的博文源文件做同步。

但是现在我完全没必要这样做了，我完全可以在同一个仓库的两个分支保存这两份文件，因为 github 的 pages 服务只允许在 master 分支进行构建，所以我在 `agenric.github.io` 中创建一个新的 blog 分支，在这个分之下我维护了一个 `source` 文件夹和 `_config.yml` 、`.travis.yml` 两个文件。

`_config.yml` 事实上就是我之前旧的博客的样式文件，我需要在自动构建时使用该样式文件。
`.travis.yml` 不用解释，用来描述 Travis CI 具体的构建步骤。

下面贴出我的 `.travis.yml` 文件来详细看一下流程：

```yml
language: node_js
node_js:
  - "8.9.1"

branches:
only:
  - blog

cache:
  directories:
    - node_modules

before_install:
  - npm install npm@latest -g
  - npm update -g
  - cd ..

install:
  - npm install hexo-cli -g
  - hexo init blog
  - cd blog
  - /bin/cp -rf ../agenric.github.io/package.json package.json
  - cat package.json
  - npm install

script:
  - /bin/cp -rf ../agenric.github.io/_config.yml _config.yml
  - rm -rf source/
  - /bin/cp -rf ../agenric.github.io/source ./
  - git clone https://github.com/Agenric/hexo-theme-next.git themes/next
  - sed -i'' "/^ *repo/s~github\.com~${GITHUB_TOKEN}@github.com~" _config.yml
  - hexo g

after_success:
  - git init
  - git config user.name "Travis CI"
  - git config user.email "agenricwon@gmail.com"
  - hexo d

```

我们指定构建的默认环境是 Node 8.9.1 版本，只在 blog 分支执行构建操作，同时为了加快构建速度，我们给 node_modules 文件夹添加缓存。

在安装 Hexo 之前，先将本地的 npm 以及 npm 默认包的版本到最新，然后我们覆盖默认的 package.json，package.json 指定了我所需要的所有插件以及版本要求。接着 install，结束之后你会发现所有的准备工作已经做完了。

安装完所需要的所有插件之后，在 script 阶段我会先把网站的配置文件和博文源文件拷贝并粘贴到 blog 的根目录。同时接着安装所需的主题之后，我们看到我需要设置 `_config.yml` 文件中 deploy 到的远程 repo 地址。然后执行构建操作。

最后在所有的操作结束之后，执行 deploy 操作。

至此，我在一台新设备上只需要 clone `agenric.github.io` 这个仓库的 blog 分支，在 `source/_posts` 文件夹中创建新的博文即可。