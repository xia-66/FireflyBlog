---
title: "极致远程桌面的几种方法"
published: 2024-09-20
updated: 2024-09-29
description: ""
category: "软件相关"
draft: false
---

前提：受控端和主控端网络质量良好且稳定
须知：以下所有方法均在Window及Android端实现，macOs下未测试，步骤细节待补充！！！

**一、GameViewer**

不想折腾的，直接无脑入，网易的远程桌面，可调画面质量，需注册登录，什么时候收费未知

**二、双端IPv6环境下（对于网络要求较高）**

sunshine（受控端）+ moonlight（主控端）
由于运营商给受控端分配的ipv6会发生变化，需要用到[ddns-go][1]域名动态解析到变化的ipv6

**三、IPv4环境下（目前国内长时间存在的状态）**
sunshine + 皎月连/ZeroTier（内网穿透）+ moonlight 

**四、Frp、Parsec、ZeroTier搭建Moon中转服务器（可选）**
frp需要大带宽云服务器，parsec被国内墙，可能连不通，moon上手太高

GameViewer：https://gv.163.com/
皎月连：https://www.natpierce.cn/
ZeroTier：https://www.zerotier.com/
sunshine：https://github.com/LizardByte/Sunshine/releases/tag/v0.23.1
moonlight：https://github.com/moonlight-stream


  [1]: https://github.com/jeessy2/ddns-go