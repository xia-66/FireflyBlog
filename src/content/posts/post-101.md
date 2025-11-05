---
title: "Docker清理操作"
published: 2024-09-23
updated: 2024-09-29
description: ""
tags: ["Docker"]
category: "计算机相关"
draft: true
---

一、删除镜像

    //查看所有镜像
    docker images
    //删除镜像（应先将使用该镜像的容器停掉，不然报错）
    docker rmi 镜像ID(只取“IMAGE ID”的前3个字符即可)

二、删除容器

    //查看运行中的容器
    docker ps
    //停止容器
    docker stop 容器ID(只取“CONTAINER ID”的前3个字符即可)
    //查看所有容器的状态
    docker ps -a
    //删除容器
    docker rm 容器ID

