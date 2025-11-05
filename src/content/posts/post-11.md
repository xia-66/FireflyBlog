---
title: "解决Vue打开页面不是置顶"
published: 2023-11-21
updated: 2024-09-20
description: ""
category: "前端相关"
draft: false
---

不知道啥问题，刚写的新vue页面打开后居然在页面中部，使用体验很差，通过下面的代码实现这个问题

      onMounted(() => {
        window.scrollTo(0, 0);
      });