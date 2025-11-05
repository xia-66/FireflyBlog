---
title: "解决Vant中更改默认样式"
published: 2024-03-01
updated: 2024-09-20
description: ""
tags: ["Vant4","Vue3"]
category: "前端相关"
draft: false
---

***前言***

  vant组件中，修改组件内部创建的元素，找到类名修改样式没法生效，又不能把scoped去掉（会影响其他页面），只能进行穿透处理

**解决办法：**  `::v-deep深度选择器`

  一、在需要修改的样式中添加新类名覆盖掉或找准精确定位

    <van-cell-group>
         <van-field label=\"密码\" value=\"密码修改\" class=\"password\" />
         <van-field label=\"手机号\" value=\"手机号\" />          
    </van-cell-group>

  二、对指定样式中内容进行穿透设置(注意,此时使用的是scss写法!)

     .password{
        ::v-deep .van-field__control{
         color: #365EEF;
         }
      }