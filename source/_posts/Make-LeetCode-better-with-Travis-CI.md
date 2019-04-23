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

事实上我大概半个月之前无意间在 github 上看到了 [leetcode-cli](https://github.com/skygragon/leetcode-cli) 这个项目，当时觉得很好玩就安装上用了一下，期间我发现 `leetcode show` 这个命令很有趣，show 命令可以让我们快速的预览某一道题的详细信息。

当我们查看某一题的详细信息时，我们发现