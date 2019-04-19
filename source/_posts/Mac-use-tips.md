---
title: Mac使用小技巧
date: 2019-01-18 11:27:03
 - Mac
categories:
 - 效率
---

> 保存一些小的tips

* Mac系统语言为英文的情况下，设置office语言为中文

```bash
defaults write com.microsoft.Word AppleLanguages '("zh-cn")'
defaults write com.microsoft.Excel AppleLanguages '("zh-cn")'
defaults write com.microsoft.Powerpoint AppleLanguages '("zh-cn")'
```

* Mac应用无法打开或文件损坏

```bash
sudo spctl --master-disable
```