---
title: "Css中溢出文字用省略号代替"
published: 2023-12-29
updated: 2024-09-20
description: ""
tags: ["Css"]
category: "前端相关"
draft: false
---

一、单行文字溢出

    text-overflow: ellipsis; //文本溢出用省略号表示
    overflow: hidden;  // 溢出隐藏
    white-space: nowrap;  // 溢出不换行

二、多行文字溢出

    -webkit-line-clamp: 5; //表示显示5行
    -webkit-box-orient: vertical;  // 从上到下垂直排列子元素
    display: -webkit-box;  //  将对象作为弹性盒子模型显示。
    white-space: nowrap; // 溢出不换行
    text-overflow: ellipsis; //文本溢出用省略号表示
    overflow：hidden；// 溢出隐藏

不生效原因

  没有给容器添加固定宽度、容器不可以是flex布局，若使用flex布局，可以将flex包裹