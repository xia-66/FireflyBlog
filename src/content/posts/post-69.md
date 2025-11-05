---
title: "利用IPv6实现远程连接"
published: 2024-04-07
updated: 2024-04-12
description: ""
tags: ["远程桌面","IPv6"]
category: "计算机相关"
draft: false
---

**前言**

之前通过frp实现远程连接，那样需要一个公网IPv4，对一般人确实没必要买一个云服务器，IPv6的搭建没那么复杂，也不需要买其他东西

**准备工作**

Window专业版及用户名和密码 + 一个IPv6网络 + 另一台连了IPv6的设备进行远程连接（家庭版可看之前的文章修改为专业版）[IPv6测试][1]
<p style=\"color:red\">经测试：校园网开放了IPv6，手机热点是IPv6，BJ网未开放IPv6（可联系客服），其他的网络需自行测试</p>

![2024-04-07T11:29:12.png][2]

**演示（该演示仅适用于Window）**

  首先打开Window本地的IPv6协议
![2024-04-07T12:28:37.png][3]
  寝室内的校园网用的是小米路由器（其他路由器自行设置），连接上校园网后，进入[小米路由器的后台][4]，手动开启IPv6

  登陆后，找到上网设置开启IPv6，上网方式设置为NAT6

  打开终端，输入Ipconfig，可以查看到已经连上了IPv6
![2024-04-07T11:48:37.png][5]
  但这只是适用于局域网，接下来拿到IPv6公网IP

  在后台将WiFi的工作模式改变为有线中继工作模式(但是这样的话WiFi就成一个人的了，其他人连上后基本没有速度)，<span style=\"color:red\">此时小米路由器的后台地址变化了，记下这个IP以便以后访问后台</span>（忘记截图了）

![2024-04-07T11:54:27.png][6]
另一种办法就是，关闭IPv6的防火墙，华硕路由器和小米AX6000路由器和红米AX6S路由器支持后台直接关闭，其他的话只能SSH关掉了防火墙了

  再次打开终端，输入Ipconfig，现在的IPv6就是公网IP了，外部直接访问IPv6即可进行远程连接，手机平板可以通过[RD client][7]连接，window则直接通过自带的远程桌面连接，其速度完全取决于你的带宽


  [1]: https://www.test-ipv6.com/index.html.zh_CN
  [2]: https://blog.heiyu.fun/usr/uploads/2024/04/672259141.png
  [3]: https://blog.heiyu.fun/usr/uploads/2024/04/2654474940.png
  [4]: http://www.miwifi.com
  [5]: https://blog.heiyu.fun/usr/uploads/2024/04/1757761981.png
  [6]: https://blog.heiyu.fun/usr/uploads/2024/04/58988269.png
  [7]: https://learn.microsoft.com/zh-cn/windows-server/remote/remote-desktop-services/clients/remote-desktop-android#download-the-remote-desktop-client