---
title: "Npm镜像源操作"
published: 2024-03-04
updated: 2024-09-23
description: ""
tags: ["Npm"]
category: "软件相关"
draft: false
---

**打开PowerShell**

**查看当前使用的镜像地址命令**

    npm config get registry

**设置为淘宝镜像源**

    npm config set registry https://registry.npmmirror.com/

**还原默认镜像**

    //方法一
    npm config delete registry
    //方法二
    npm config set registry https://registry.npmjs.org