---
title: "移动端点击手机号唤醒拨号功能"
published: 2024-03-02
updated: 2024-03-11
description: ""
tags: ["Vue3"]
category: "前端相关"
draft: false
---

**在vue项目中**

  在index.html中加入这一段

    <meta name=\"format-detection\" content=\"telephone=yes\"/>

  创建一个拨号函数

    const callPhone = (phoneNumber) =>  {
      window.location.href = \'tel:\' + phoneNumber
    }

  然后调用即可

   **在H5中（phoneNumber即为要放的电话）**

    <a href=\"tel:phoneNumber\">phoneNumber</a>

  体验链接 [test.lumine.top][1]


  [1]: http://test.lumine.top