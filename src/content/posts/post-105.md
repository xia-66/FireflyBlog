---
title: "Element-ui el-select不更新问题"
published: 2024-10-16
updated: 2024-10-25
description: ""
tags: ["ElementUI"]
category: "前端相关"
draft: false
---

问题：在下拉框中选择某个值后，el-select绑定的值更新了但是页面显示没有更新。
2409:894a:110:37f9:3c1b:a69:b16c:f411
原因：
因为下拉框数据是循环别的接口得来的，因为数据层次太多，render函数没有自动更新，需手动强制刷新所以我直接强制刷新了值，而forceUpdate就是重新render。

解决方法：
在el-select标签中添加 @change=\"$forceUpdate();\" 进行强制渲染