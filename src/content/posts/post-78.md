---
title: "通过QQ号API获取用户高清QQ头像与QQ昵称"
published: 2024-05-22
updated: 2024-05-22
description: ""
tags: ["API"]
category: "计算机相关"
draft: false
---

普通头像

    http://q1.qlogo.cn/g?b=qq&nk=QQ号码&s=100
    http://q2.qlogo.cn/headimg_dl?dst_uin=QQ号码&spec=100

高清头像

    http://q.qlogo.cn/headimg_dl?dst_uin=QQ号码&spec=640&img_type=jpg

参数介绍

<table>
   <tr>
      <td>规格</td>
      <td>像素</td>
   </tr>
   <tr>
      <td>1</td>
      <td>40 X 40</td>
   </tr>
   <tr>
      <td>2</td>
      <td>100 X 100</td>
   </tr>
  <tr>
      <td>3</td>
      <td>140 X 140</td>
   </tr>   <tr>
      <td>4</td>
      <td>640 X 640</td>
   </tr>
</table>

QQ昵称

    https://api.oioweb.cn/api/qq/info?qq=QQ号码