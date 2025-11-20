---
title: "VPS一键脚本"
published: 2025-05-12
updated: 2025-07-12
description: ""
category: "VPS相关"
draft: false
---

更新系统

    apt update && apt upgrade -y

融合怪测试

    curl -L https://gitlab.com/spiritysdx/za/-/raw/main/ecs.sh -o ecs.sh && chmod +x ecs.sh && bash ecs.sh

ip质量测试

    bash <(curl -Ls IP.Check.Place)

--------------------------------------------

    bash <(curl -sL https://run.NodeQuality.com)   

网络质量测试

    bash <(curl -Ls Net.Check.Place) -4

hy2

    wget -N --no-check-certificate https://raw.githubusercontent.com/flame1ce/hysteria2-install/main/hysteria2-install-main/hy2/hysteria.sh && bash hysteria.sh

3x-ui

    bash <(curl -Ls https://raw.githubusercontent.com/xeefei/3x-ui/master/install.sh)

启用BBR拥塞控制算法

    # 启用 BBR 拥塞控制算法
    sysctl -w net.ipv4.tcp_congestion_control=bbr
    
    # 使配置永久生效（修改 sysctl 配置文件）
    echo \"net.ipv4.tcp_congestion_control = bbr\" >> /etc/sysctl.conf
    sysctl -p

启用fq队列管理算法

    # 启用 FQ_Codel
    sysctl -w net.core.default_qdisc=fq_codel
    
    # 使配置永久生效（修改 sysctl 配置文件）
    echo \"net.core.default_qdisc = fq_codel\" >> /etc/sysctl.conf
    sysctl -p

----------------------------

    # 检查 TCP 拥塞控制算法
    sysctl net.ipv4.tcp_congestion_control    #成功应该返回“net.ipv4.tcp_congestion_control = bbr”#
    # 检查默认队列调度器
    sysctl net.core.default_qdisc    #成功应该返回“net.core.default_qdisc = fq_codel”#

233boy一键脚本

    bash <(wget -qO- -o- https://github.com/233boy/sing-box/raw/main/install.sh)

xboard后端对接

    wget -N https://raw.githubusercontent.com/wyx2685/V2bX-script/master/install.sh && bash install.sh

    wget -N https://raw.githubusercontent.com/XrayR-project/XrayR-release/master/install.sh && bash install.sh

xrayr相关命令

    XrayR restart
    XrayR log


