---
title: "Vue3+Vite引用微信开放标签"
published: 2023-11-21
updated: 2024-09-20
description: ""
tags: ["Vite"]
category: "学习记录"
draft: true
---

**一、使用前置条件**

微信开放标签使用步骤与微信JS-SDK类似，也需要引入JS文件等步骤。如果是公众号身份的网页，需要绑定安全域名。具体参见[官方文档][1]。

**二、vite.config.js中plugins中的配置**

    export default defineConfig({
    //...其他配置
      plugins: [
        vue({
          template: {
            compilerOptions: {
              isCustomElement: (tag) => tag.startsWith(\"wx-\"),
            },
          },
        }),
      ],
    });

**三、在打包后注入的入口文件中引入微信SDK文件**

  一般为public文件夹下的index.html文件
    <script src=\"http://res.wx.qq.com/open/js/jweixin-1.6.0.js \"></script>

**四、通过config接口注入权限验证配置并申请所需开放标签**

  首先向后端发请求获得页面路径及要跳转的小程序id，再通过后端调用微信接口，返回签名等信息，再调用 wx.config 申请开放标签。
      // 引入微信开放标签
      wx.config({
        debug: false,
        appId: appId,
        timestamp: timestamp,
        nonceStr: nonceStr,
        signature: signature,
        jsApiList: [\"chooseImage\"],
        openTagList: [\"wx-open-launch-weapp\"],
      });

**五、用法**

    <wx-open-launch-weapp class=\"opentag\" username=\"gh_xxxxxx\" path=\"pages/cart/cart\" @launch=\"handleLaunchFn\" @error=\"handleErrorFn\">
        <div v-is=\"\'script\'\" type=\'text/wxtag-template\'>
             <div style=\"width:3rem;height:3rem\"></div>
        </div>
    </wx-open-launch-weapp>

  [1]: https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_Open_Tag.html
