---
title: "Shell偶尔有用的脚本"
published: 2024-09-18
updated: 2024-09-20
description: ""
category: "计算机相关"
draft: false
---

在Linux下（通常用于宝塔面板的定时任务清空缓存日志或者下载文件）

    #/*表示清空data目录下的所有内容
  
    rm -f /www/wwwroot/data/*

    #从某一url下载文件到本地

    #!/bin/bash
    # 定义下载链接和目标目录
    URL=\"http://xx.xx.xx.xx:8888/down/U6rGuLsIcNy6.tar.gz\"
    TARGET_DIR=\"/www/wwwroot/data\"
    
    # 检查目标目录是否存在，如果不存在则创建
    if [ ! -d \"$TARGET_DIR\" ]; then
      mkdir -p \"$TARGET_DIR\"
    fi
    
    # 使用wget下载文件到目标目录
    wget -O \"$TARGET_DIR/U6rGuLsIcNy6.tar.gz\" \"$URL\"
    
    # 打印执行结果
    echo \"文件已下载到 $TARGET_DIR/U6rGuLsIcNy6.tar.gz\"

在Window桌面下

    @echo off
    set \"url=http://xx.xx.xx.xx:8888/down/VfHT6ZUIIhuq.zip\"
    set \"localPath=E:\\test\\111.zip\"
    
    echo Checking if local path exists...
    if not exist \"%localPath%\\..\" (
        echo Creating local path...
        mkdir \"%localPath%\\..\"
    )
    
    echo Attempting to download file from %url% to %localPath%...
    powershell -Command \"Invoke-WebRequest -Uri \'%url%\' -OutFile \'%localPath%\' -MaximumRedirection 10 -TimeoutSec 60\"
    
    if %errorlevel% neq 0 (
        echo Download failed. Please check the URL and your network connection.
    ) else (
        echo Download complete.
    )
    
    pause

