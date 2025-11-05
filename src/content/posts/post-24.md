---
title: "Frp实现远程桌面连接"
published: 2023-11-25
updated: 2024-09-20
description: ""
tags: ["Win11","Frp","远程桌面","Ubuntu"]
category: "计算机相关"
draft: false
---

## 前言 ##
  
  frp 是一个可用于内网穿透的高性能的反向代理应用，分为服务端frps和客户端frpc，支持 tcp, udp, http, https 协议。详情可浏览该项目的[Github主页][1][一面名心镜][2]大佬说frp内网穿透这个很好用，让我试试，跟着他的教程弄，排查了好久的问题，还是有点出入，所以还是记录一下吧

## 工作原理 ##

1.服务端运行，监听一个主端口，等待客户端的连接；
2.客户端连接到服务端的主端口，同时告诉服务端要监听的端口和转发类型；
3.服务端fork新的进程监听客户端指定的端口；
4.外网用户连接到客户端指定的端口，服务端通过和客户端的连接将数据转发到客户端；
5.客户端进程再将数据转发到本地服务，从而实现内网对外暴露服务的能力。

## 必备条件 ##

  一台拥有公网IP的Linux服务器，一台win10/11专业版客户端（家庭版转专业版可以看之前的文章），服务器的安全组7000/7500打开（如果使用到宝塔面板，把防火墙7000/7500也打开），安装所需的[frp文件][3]，Window和Linux都按装最新的即可，写文章的时候最新版本为0.52.3，使用的是Ubuntu 22.04 Win11

## 服务器配置（frp-service） ##

 **一、将Linux解压的frp文件夹中保留frps.exe和frps.toml（LICENSE文件无所谓）上传到Linux目录**
 
 我的是在 /www/wwwroot/frps 进入到frps目录下 

    cd /www/wwwroot/frps

 再打开frps.toml

      sudo vim frps.toml

 并修改frps.toml为

    webServer.addr = \"0.0.0.0\"  #本网络中的本机
    bindPort = 7000  #frp服务器监听的端口号，用于与客户端建立连接。
    vhostHTTPPort = 7002  #虚拟主机HTTP访问的端口号，用于通过HTTP协议访问frp服务器上的虚拟主机
    webServer.port = 7500  #frp仪表盘（Dashboard）的端口号，用于通过Web界面管理和监控frp服务器。
    webServer.user = \"admin\"  #frp仪表盘的用户名，用于登录仪表盘进行管理操作。
    webServer.password = \"admin\"  #frp仪表盘的密码，用于与用户名一起进行身份验证。
    auth.method = \"token\"  #用于认证的方法，可以设置为 \"token\"，表示使用令牌进行认证。
    auth.token = \"918822\"  #用于认证的令牌，客户端需要提供正确的令牌才能与服务器建立连接。

  配置完毕后保存退出

**二、启动frps** 

     ./frps -c frps.toml

  提示成功后，能通过 你的IP：7500的frp面板表示frps配置无误

**三、使用 systemd 管理 frps 服务**
  安装 systemd

    apt install systemd

  创建 frps.service 文件，使用vim在 /etc/systemd/system 目录下创建一个 frps.service 文件，用于配置 frps 服务。

    sudo vim /etc/systemd/system/frps.service

  并在 frps.service 中写入

    [Unit]
    Description=Frp Server Service
    After=network.target
    
    [Service]
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart = /www/wwwroot/frps/frps -c /www/wwwroot/frps/frps.toml #注意路径，这里是我的路径
    LimitNOFILE=1048576
    
    [Install]
    WantedBy=multi-user.target

  启动frps

    sudo systemctl start frps

  设置 frps开机自启动

    sudo systemctl enable frps

## Windows配置（frp-client） ##

  Window解压的frp文件夹中保留frpc.exe和frpc.toml（LICENSE文件无所谓）放在一个不容易被删的地方（路径不得含中文）
  使用记事本打开frpc.toml并写入

    serverAddr = \"你的公网IP\"
    serverPort = 7000  #与bindPort保持一致
    log.to = \"./frpc.log\"  #日志路径
    log.level = \"info\"   #日志等级
    log.maxDays = 3  #日志保存天数
    auth.method = \"token\"  #和上文一样
    auth.token = \"918822\"
    [[proxies]]
    name = \"control\"
    type = \"tcp\"
    localIP = \"127.0.0.1\" 
    localPort = 3389  #本地的远程连接映射端口
    remotePort = 7002  #与之前的vhostHTTPPort保持一致

  使用CMD命令启动：

    frpc -c frpc.toml

  不要关闭窗口，打开frpc.log，看是否显示为 [control] start proxy success

  在第三个设备上的 Microsoft远程桌面 应用上加入电脑 你的IP:7002 即可访问

 
## 可能遇到的问题 ##

 - frpc.log中显示start error: port unavailable
   那么你需要更改remotePort和vhostHTTPPort并保持端口一致
 - token in login doesn\'t match token from configuration  
   你可以跳转到[一面明心镜][4]，查看该错误

## 参考资料 ##
   

  [一面明心镜][5]
  [frps_full_example.toml文档][6]


  [1]: https://github.com/fatedier/frp/
  [2]: https://blog.leesong.top/index.php/archives/143/
  [3]: https://github.com/fatedier/frp/releases
  [4]: https://blog.leesong.top/index.php/archives/143/
  [5]: https://blog.leesong.top/index.php/archives/143/
  [6]: https://github.com/fatedier/frp/blob/dev/conf/frps_full_example.toml