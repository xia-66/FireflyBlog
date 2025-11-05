---
title: "Frp内网穿透Web服务"
published: 2024-12-18
updated: 2024-12-19
description: ""
tags: ["Frp"]
category: "计算机相关"
draft: false
---

**配套工具：**
  公网IPv4的Linux（带宽不是很重要）
  Linux安全组及宝塔面板开放端口
**具体步骤**
Linux装[frps][1]，Win装[frpc][2]，下载对应操作系统的文件，Linux保留frps和frps.toml文件，Win保留frpc.exe和frpc.toml文件（多余文件可删除）

修改frps.toml文件，vhostHTTPPort 为浏览器访问端口

    bindPort = 7000
    vhostHTTPPort = 8080

控制 frps 服务端的启动、停止、配置后台运行以及开机自启动。
1、安装 systemd
    # 使用 yum 安装 systemd（CentOS/RHEL）
    yum install systemd
    
    # 使用 apt 安装 systemd（Debian/Ubuntu）
    apt install systemd
2、创建 frps.service 文件

    $ sudo vim /etc/systemd/system/frps.service

写入内容ExecStart 需要修改成你的路径比如 /www/wwwroot/frps/frps -c /www/wwwroot/frps/frps.toml

    [Unit]
    # 服务名称，可自定义
    Description = frp server
    After = network.target syslog.target
    Wants = network.target
    
    [Service]
    Type = simple
    # 启动frps的命令，需修改为您的frps的安装路径
    ExecStart = /path/to/frps -c /path/to/frps.toml
    
    [Install]
    WantedBy = multi-user.target

3、使用 systemd 命令管理 frps 服务

    # 启动frp
    sudo systemctl start frps
    # 停止frp
    sudo systemctl stop frps
    # 重启frp
    sudo systemctl restart frps
    # 查看frp状态
    sudo systemctl status frps

4、设置 frps 开机自启动

    sudo systemctl enable frps

**启动frps**

    sudo systemctl start frps

修改frpc.toml文件，serverAddr 为Linux公网IP，localPort 为本地跑的服务端口，customDomains 为绑定的域名，测试了几次不使用域名的话有问题

    serverAddr = \"x.x.x.x\"
    serverPort = 7000
    
    [[proxies]]
    name = \"web\"
    type = \"http\"
    localPort = 3000
    customDomains = [\"xx.xx.xx\"]

**启动frpc**

    frpc -c frpc.toml

**客户端快捷方式**
下载地址https://cloudreve.heiyu.fun/f/3yu6/heiyu.exe
![2024-12-19T02:20:24.png][3]
 接下来就可以通过域名+vhostHTTPPort 访问
官方文档：https://gofrp.org/zh-cn/docs/


  [1]: https://github.com/fatedier/frp/releases/tag/v0.61.1
  [2]: https://github.com/fatedier/frp/releases/tag/v0.61.1
  [3]: https://blog.heiyu.fun/usr/uploads/2024/12/3156401418.png