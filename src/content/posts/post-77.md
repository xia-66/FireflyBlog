---
title: "开源项目如何正确使用"
published: 2024-04-28
updated: 2024-04-28
description: ""
tags: ["开源"]
category: "计算机相关"
draft: false
---

**更新项目配置所需的包，检查不匹配的包**

    npm outdated

![QQ图片20240428201324.png][1]
可以有选择的进行更新（先卸载再安装）

    npm uninstall 包名

    npm i 包名

或者直接全部更新

    npm update

这样更新只会将上面红色的包（低于项目所需的版本更新到最新），黄色的包就只能手动去更新了

**在 package.json 中修改 dependencies 和 devDependencies （影响打包的速度）**

devDependencies： 开发时所依赖的工具包；

    npm i  包名 -D

dependencies：项目正常允许时需要的依赖包；

    npm i  包名 -S

**删除无用的代码，开源项目一般都有很多的功能，留下要用的，扔掉无用的，代码越多越乱**


  [1]: https://blog.heiyu.fun/usr/uploads/2024/04/1799720760.png