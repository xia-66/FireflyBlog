---
title: "微信小程序中引用Vant-weapp"
published: 2023-11-27
updated: 2024-09-20
description: ""
tags: ["Vant-weapp","微信小程序"]
category: "学习记录"
draft: true
---

## 前言 ##

最近开发微信小程序，需要使用vant库，在小程序中只能用vant-weapp，有两个文档，gitee和github，但最需要注意的是版本，写文章时最新的版本为1.11.1

##使用##

    # 通过 npm 安装
    npm i @vant/weapp -S --production

  **app.json 中的 \"style\": \"v2\" 去除，不关闭可能会造成样式冲突**

  **修改 project.config.json（这点vant文档中并没有写清楚**）

  **普通项目修改为**

    {
      ...
      \"setting\": {
        ...
        \"packNpmManually\": true,
        \"packNpmRelationList\": [
          {
            \"packageJsonPath\": \"./package.json\",
            \"miniprogramNpmDistDir\": \"./\"
          }
        ]
      }
    }

  **云开发项目修改为**

    {
      ...
      \"setting\": {
        ...
        \"packNpmManually\": true,
        \"packNpmRelationList\": [
          {
            \"packageJsonPath\": \"./miniprogram/package.json\",
            \"miniprogramNpmDistDir\": \"./miniprogram/\"
          }
        ]
      }
    }

**构建 npm 包**

  点击工具 -> 构建npm   **新版微信开发者工具中没有使用npm模块的选项，不需要处理**

**引入组件**

  在app.json中引用是全局引用，每个页面的json文件中引用则是局部引用（常用引用方式）

    \"usingComponents\": {
      \"van-button\": \"@vant/weapp/button/index\"
    }

##可能遇到的问题##

  局部引用组件没问题，但是报错找不到文件，那就把引用的组件在app.json中全局引用，具体原因不清楚