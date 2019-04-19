---
title: 批量删除Git远程分支
date: 2017-12-08 22:17:41
categories:
 - 技术
tags:
 - Git
---

多人协作开发的过程中如果没有一个很好很规范的Git操作权限的管理，那么就很难避免不出现这样一种情况，所有人可以任意push、任意create branch、各种merge等等。 [我不管，我最酷。🙅🏻‍♂️]

这样一段时间之后你就会发现无论是自己本地还是远程都会出现一堆垃圾的分支，看着简直难受，强迫症根本受不了啊。所以，很显然我们不会去手动的一个一个分支的去删除，当然如果你一定要 `git push origin :branchName` 那你随意。

OK，批量删除，如何批量，首先我们需要列出我们需要删除的一系列分支。`git branch -r` 可以列出当前仓库下所有的远程分支，也就是说我们在这个列表里挑选出我们需要删除的某些分支。

所以，接着我们先列出符合自己想要删除符合某些规则的分支名。假如我的git远程地址默认是origin，那么其实在上一步 `git branch -r` 之后大概会长这样：

```bash
$ git branch -r
  origin/aaa
  origin/aaa/sss
  origin/bbb
  origin/ccc/ddd
  origin/eee/fff/ggg
```

比如我想删除所有a开头的分支，可以使用Linux的awk命令，先输出符合所有a开头的分支，可以这样：

```bash
$ git branch -r| awk -F '[/]' '/origin\/a/ {printf "%s\n", $2}'
  origin/aaa
  origin/aaa/sss
```

至于 `awk` 命令的使用，请自行Google，我一般也是用到才会去查，翻手册。🤦‍♂️

获得了想要删除的列表，最后一步就是执行删除操作了。所以完整命令：

```bash
$ git branch -r| awk -F '[/]' '/origin\/a/ {printf "%s\n", $2}' |xargs -I{} git push origin :{}
To xxx.xxx.xxx.git
 - [deleted]             aaa
To xxx.xxx.xxx.git
 - [deleted]             aaa/sss
```

另外，多一句。删除本地分支无非就是最后的操作不一样：`git branch -d` 即可。

事实上，我感觉正规的协作流程就是在主仓库禁止所有developer执行push、create branch操作，所有开发人员从主库fork代码到自己的仓库，每个人维护自己的feature即可。需要进主库的feature提交mr即可。

Done !