---
title: "Nvm切换Node.js版本"
published: 2024-07-27
updated: 2024-09-20
description: ""
tags: ["Nvm"]
category: "后端相关"
draft: false
---

**nvm（Node Version Manager）**

Node.js支持多版本共存，可以在同一台电脑上轻松管理和切换多个Node.js版本。
nvm下载地址：https://github.com/coreybutler/nvm-windows/

安装的时候会选择nvm安装目录和node安装目录，两个目录都需要为空目录
安装完成后打开nvm安装目录修改setting.txt的内容
![2024-09-09T12:25:17.png][1]

    node_mirror: https://npmmirror.com/mirrors/node/
    npm_mirror: https://npmmirror.com/mirrors/npm/


    // 查看可安装的Node.js版本
    nvm list available           
    // 安装16.14.0版本的Node.js
    nvm install 16.14.0         
    // 切换使用指定版本的Node.js
    nvm use 16.14.0          
    // 查看已安装的Node.js版本
    nvm list  
    //卸载已安装Node.js版本
    nvm uninstall 16.14.0

常见问题
显示已经切换成功，但实际没有，主要原因是在装nvm之前已经存在node版本了，nvm包里面的版本和已经存在的文件没有关联，找到之前安装的node,删掉文件
nvm use 突然失效，建议卸载nvm重新安装


  [1]: https://blog.heiyu.fun/usr/uploads/2024/09/1943800881.png