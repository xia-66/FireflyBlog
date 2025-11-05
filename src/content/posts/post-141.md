---
title: "docker虚拟化的Debian操作系统对接xrayr"
published: 2025-10-05
updated: 2025-10-05
description: ""
category: "默认分类"
draft: false
---

正常安装命令

    wget -N https://raw.githubusercontent.com/XrayR-project/XrayR-release/master/install.sh && bash install.sh

XrayR.service 文件在/etc/systemd/system/XrayR.service
正确配置/etc/XrayR/config.yml文件
重新加载 systemd 配置

    systemctl daemon-reload

启用服务 (设置开机自启)

    systemctl enable XrayR.service

手动启动服务 (立即生效)

    systemctl start XrayR.service

