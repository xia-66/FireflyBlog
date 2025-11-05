---
title: "前后端分离开发（Vue3+TP5）引入微信开放标签注意事项"
published: 2023-12-04
updated: 2024-04-12
description: ""
tags: ["Vue3","前后端分离开发","TP5","微信开放标签"]
category: "后端相关"
draft: false
---

前言
--

该文章由王建辉提供，涉及到vue3、thinkPHP、vite

使用
--

  想要在微信中的H5应用中引入微信开放标签，即引入微信小程序，大体按照微信网页开发中的要求进行即可（教程地址：https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_Open_Tag.html），除了对js接口安全域名、ip白名单的配置，开发中也有一些需要注意的点，本篇文章主要介绍这些需要注意的事项。


----------


  1.微信开发者工具的使用。因为微信浏览器没有控制台等工具供我们开发调试，所以开发时不易定位问题等。此时我们需要联系管理员在微信公众号中将我们加入到开发者人员名单中，此后我们就可以在微信开发者工具中定位、解决问题啦。
  2.因为微信平台的特殊限制，微信开放标签只能在真机中看到效果，即使在微信开发者工具中也不能进行小程序的跳转。如果是使用vue3，我们就需要一遍一遍地打包上传代码到服务器，同时在真机查看微信开放标签的效果，这一点相比较原生html引入微信开放标签是比较繁琐的。
  3.前端开发注意事项（本文是在vite下引入微信开放标签，基本同vue-cli流程一致，配置方式可能有些许不同）：
  **（1）首先在vite.config.ts/js中引入相关的配置来忽略微信开放标签，否则Vue3会忽略这些非官方标签。**
   
![图片1.png][1]
    // 对应代码
    plugins: [vue({
        template:{
          compilerOptions:{
            isCustomElement: tag => tag.startsWith(\'wx-\')
          }
        }
      })],

  **（2）在根目录下的index.html文件引入相关微信js文件和配置（vue-cli是在public目录下）**
![图片2.png][2]
 

    // 对应代码，下方meta主要解决https下http请求失败
    <meta http-equiv=\"Content-Security-Policy\" content=\"upgrade-insecure-requests\">
    <script src=\"http://res.wx.qq.com/open/js/jweixin-1.6.0.js\"></script>

   **(3)引入后端返回的微信开放标签相关的一些参数**
 ![图片3.png][3]
 ![图片4.png][4]
  **（4）在.vue文件的template中引入微信开放标签**（三个需要注意的点，一是vue3中的template里不能有script标签，需要使用v-is=”’script’”来代指script标签；二是微信开放标签外部的css样式不能生效，需要在其内部进行书写行内样式，三是不能使用前端本地的图片资源，必须是可以访问的线上资源）。
![图片5.png][5]
 
至此，前端配置完成。
  **4.后端需要引入JS-SDK使用权限签名算法**（官方文档：https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html#62），配置好后在接口返回给前端对应的配置信息。需要注意的，后端需要配置好引入微信开放标签的前端页面url（不包含#及其后面内容）。
![图片6.png][6]
以上就是配置微信开放标签需要注意的点，望大家在开发中多多注意！


  [1]: https://blog.heiyu.fun/usr/uploads/2023/12/2512616880.png
  [2]: https://blog.heiyu.fun/usr/uploads/2023/12/1338175431.png
  [3]: https://blog.heiyu.fun/usr/uploads/2023/12/1868722129.png
  [4]: https://blog.heiyu.fun/usr/uploads/2023/12/1054266257.png
  [5]: https://blog.heiyu.fun/usr/uploads/2023/12/3872377285.png
  [6]: https://blog.heiyu.fun/usr/uploads/2023/12/328070092.png