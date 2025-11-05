---
title: "Vue3+Webpack引用微信开放标签"
published: 2023-11-21
updated: 2024-09-20
description: ""
tags: ["Webpack"]
category: "学习记录"
draft: true
---

**一、使用前置条件**

微信开放标签使用步骤与微信JS-SDK类似，也需要引入JS文件等步骤。如果是公众号身份的网页，需要绑定安全域名。具体参见[官方文档][1]。

**二、vue.config.js配置**

  在vue项目中，直接使用微信开放标签会报错，所以要配置忽略对微信标签的校验，在 vue.config.js 中增加如下配置。
    module.exports = {
      //...其他配置
      chainWebpack: config => {
        config.module
        .rule(\'vue\')
        .use(\'vue-loader\')
        .tap(options => {
          options.compilerOptions = {
            isCustomElement: tag => tag.startsWith(\'wx-\')
          }
          return options
        })
      }
    }

**三、在打包后注入的入口文件中引入微信SDK文件**

  一般为public文件夹下的index.html文件
    <script src=\"http://res.wx.qq.com/open/js/jweixin-1.6.0.js \"></script>

**四、通过config接口注入权限验证配置并申请所需开放标签**

  首先给后端传递当前页面路径及要跳转的小程序id，通过后端调用微信接口，返回签名等信息，再调用 wx.config 申请开放标签。
    getwxConfig({
        wxMiniId: \'gh_xxxxxx\',
        signUrl: window.location.href
    }).then(res => {
        const data = res.data.data
        wx.config({
            debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印
            appId: data.appId, // 必填，公众号的唯一标识
            timestamp: data.timestamp, // 必填，生成签名的时间戳
            nonceStr: data.nonceStr, // 必填，生成签名的随机串
            signature: data.signature, // 必填，签名
            jsApiList: [\'updateTimelineShareData\'], // 必填，需要使用的JS接口列表
            openTagList: [\'wx-open-launch-weapp\'] // 可选，需要使用的开放标签列表，例如[\'wx-open-launch-app\']
        })
    })

**五、用法**

    <wx-open-launch-weapp class=\"opentag\" username=\"gh_xxxxxx\" path=\"pages/cart/cart\" @launch=\"handleLaunchFn\" @error=\"handleErrorFn\">
        <div v-is=\"\'script\'\" type=\'text/wxtag-template\'>
             <div style=\"width:3rem;height:3rem\"></div>
        </div>
    </wx-open-launch-weapp>

  [1]: https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_Open_Tag.html