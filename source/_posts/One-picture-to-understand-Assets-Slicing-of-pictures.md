---
title: 一张图看懂 Assets 中图片的 Slicing
date: 2016-12-19 21:29:00
categories:
  - 技术
tags:
  - iOS小知识
  - Xcode
---

我使用ps自己画了一张图片如下图：
![First](One-picture-to-understand-Assets-Slicing-of-pictures/One-picture-to-understand-Assets-Slicing-of-pictures-1.png)
这是一张2倍像素的图片，我把它拖入Assets.xcassets中。选择右下角Slices的类型为`Horizontal and Vertical`：
![Second](One-picture-to-understand-Assets-Slicing-of-pictures/One-picture-to-understand-Assets-Slicing-of-pictures-2.jpg)
左侧ShowSlicing显示如下：
![Third](One-picture-to-understand-Assets-Slicing-of-pictures/One-picture-to-understand-Assets-Slicing-of-pictures-3.jpg)
在storyboard拖入一个ImageView，上下左右距离边框均为20。则显示如下：
![Fourth](One-picture-to-understand-Assets-Slicing-of-pictures/One-picture-to-understand-Assets-Slicing-of-pictures-4.png)
 简单来记就是纵横各三条线，线之外的四个角不会拉伸。六条线内部交汇处，亮色区域为拉伸区域，灰色蒙版区域被截取掉，不显示。仅此而已。
