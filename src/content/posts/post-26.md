---
title: "微信小程序与Vue的用法的不同写法"
published: 2023-11-27
updated: 2024-09-20
description: ""
tags: ["微信小程序","Vue3"]
category: "前端相关"
draft: false
---

都能使用Mustache语法{{}}

for循环渲染（vue中是v-for，小程序中是wx:for）

条件渲染（vue中是v-if v-show，小程序中是wx:if hidden）

绑定事件（vue中是@click，小程序中是bindtap）

改变数据（vue中是直接赋值a=\'\'，小程序中是

    this.setData({
      a:\'\'
    })
）

发送请求（vue中如果在API中定义了接口函数为

    函数名　（）.then((res) =>｛｝）

，小程序中为

    wx.request({
      url: \'example.php\', //仅为示例，并非真实的接口地址
      data: {
        x: \'\',
        y: \'\'
      },
      header: {
        \'content-type\': \'application/json\' // 默认值
      },
      success (res) {
        console.log(res.data)
      }
    })

）

页面跳转（vue中内部页面跳转用

    router.push({
      name: \"\",
    })
，小程序中常用为　

    wx.redirectTo({
      url: \'pages/index/index\'

})
）