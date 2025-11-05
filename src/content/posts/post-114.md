---
title: "文件下载删除shell脚本"
published: 2025-04-24
updated: 2025-04-24
description: ""
category: "默认分类"
draft: false
---

#!/bin/bash
    
    # 定义下载链接和目标目录
    URL=\"http://27.106.122.68:12501/down/xF2JdgzsM6qI.mp4\"
    TARGET_DIR=\"/www/wwwroot/data\"
    COUNT_DIR=\"/www/wwwroot/download_count\"
    COUNT_FILE=\"$COUNT_DIR/download_count.txt\"
    
    # 检查目标目录是否存在，如果不存在则创建
    if [ ! -d \"$TARGET_DIR\" ]; then
        mkdir -p \"$TARGET_DIR\"
    fi
    
    # 检查下载计数目录是否存在，如果不存在则创建
    if [ ! -d \"$COUNT_DIR\" ]; then
        mkdir -p \"$COUNT_DIR\"
    fi
    
    # 清空目标目录内容
    rm -rf \"$TARGET_DIR\"/*
    
    # 使用wget下载文件到目标目录
    wget -O \"$TARGET_DIR/xF2JdgzsM6qI.mp4\" \"$URL\"
    
    # 记录下载次数
    if [ ! -f \"$COUNT_FILE\" ]; then
        echo 0 > \"$COUNT_FILE\"
    fi
    COUNT=$(cat \"$COUNT_FILE\")
    COUNT=$((COUNT + 1))
    echo $COUNT > \"$COUNT_FILE\"
    
    # 打印执行结果
    echo \"文件已下载到 $TARGET_DIR/xF2JdgzsM6qI.mp4\"
    echo \"下载次数: $COUNT\"



--------------------------

    rm -f /www/wwwroot/data/*