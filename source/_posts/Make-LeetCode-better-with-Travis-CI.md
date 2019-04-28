---
title: Travis CI 让你愉快的刷 LeetCode
date: 2019-04-20 13:17:00
categories:
 - 效率
tags:
 - LeetCode
 - Travis CI
---

对，感觉自己最近有点魔怔了。感觉跟 [Travis CI](https://travis-ci.org/) 干上了。
我前两天刚把 Hexo 的博客重新整理完，今天早起打开 Travis CI 发现我前两天做的一个 LeetCode 的项目构建失败，对，是因为在此之前我一直没有配置好。

事实上我大概半个月之前无意间在 github 上看到了 [leetcode-cli](https://github.com/skygragon/leetcode-cli) 这个项目，当时好奇就玩了一下，其中 `leetcode show` 这个命令很有趣，show 命令可以让我们快速的预览某一道题的详细信息。

比如说我们查看一下第一题的详情，那就很简单 `leetcode show 1 -g -l swift` 同时指定使用 swift 语音来预览题目详情。

你会发现当前目录下多了一个 1.two-sum.swift 文件，诶，我就在想，如果我一直从1 show 到100那岂不是会有100个 swift 文件。既然已经把题目 down 到了本地我就不用每次都在线看了，然后我们打开 1.two-sum.swift，只有一个类下面的一个方法，我就在想时间久了之后我可能就忘了这个题到底是要解决什么问题了，所以必须要把题目的要求同时保存下来，当然这也不难，我们把命令行的输出直接写到一个临时文件里，然后把这个文件的内容添加到 1.two-sum.swift 中就好了。

然后，我有了一个大胆的想法...
既然一道题可以这也搞，那所有的也都没有什么问题啊，所以就有了这篇文章，把所有的工作交给 CI，每天去 LeetCode 帮我同步一下最新的题库，然后帮我 push 到我指定的仓库，等我想刷的时候，就在本地 pull 一下，然后所有的题就回来了。
这样岂不是美滋滋？虽然我也不知道我到猴年马月才能把这些东西看完...
算了，我不想写了，代码都在 [这里](https://github.com/Agenric/LeetCode)，就这样。