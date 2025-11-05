---
title: "Git常用合集"
published: 2023-11-22
updated: 2024-09-20
description: ""
tags: ["Git"]
category: "计算机相关"
draft: false
---

**一、克隆远端分支到本地并建立映射（克隆哪个分支branch就是哪个）**

      git clone -b <branch> 仓库的url

**二、获取远端最新代码**

      git fetch

**三、拉取远端当前分支到本地分支（适用于多人开发一个分支）**

      git pull

**四、合并其他分支到当前所在分支(branch为远端分支的一个，和上面一个)**

      git merge origin/<branch>

**五、查看当前工作台状态**

      git status

**六、提交到暂存区**

      git add .
      git commit -m \"更新日志\"

**七、提交到远端分支**

      git push


------------------------------------------------------------
***如果VSCode控制台用的好的话，上面的五六七用不到*** 

