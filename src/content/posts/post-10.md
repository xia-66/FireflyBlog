---
title: 'Vue关闭微信浏览器'
published: 2023-11-21
updated: 2024-09-20
description: ''
category: '代码相关'
draft: false
---

比如提交表单成功后跳转到一个加群的二维码页面，点击一个按钮后需要关闭微信浏览器

    document.addEventListener(
            \"WeixinJSBridgeReady\",
            function () {
              this.WeixinJSBridge.call(\"closeWindow\"); //安卓手机关闭代码
            },
            false
          );
          this.WeixinJSBridge.call(\"closeWindow\"); //苹果手机关闭代码

