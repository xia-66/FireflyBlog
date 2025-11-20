---
title: "Vant4库中修改样式"
published: 2023-11-21
updated: 2024-09-20
description: ""
tags: ["Vant4"]
category: "代码相关"
draft: false
---

有时候内置的样式并不美观，需要自定义修改

**一、!importance修改（部分适用）**

  拿到节点的class或id属性，通过css中的!importance提高权重覆盖样式

**二、根据官方提供的[ConfigProvider 覆盖][1]进行修改（好用）**

  在template中需要修改样式外包裹

      <van-config-provider :theme-vars=\"themeVars\">
      ...
      </van-config-provider>

  在js或者ts中引用

    import type { ConfigProviderThemeVars } from \'vant\';
    
    const themeVars: ConfigProviderThemeVars = {
      //修改原先的--van-padding-base: 4px; 为10px需要进行驼峰命名法
      paddingBase: \'10px\',
    };


  [1]: https://vant-contrib.gitee.io/vant/#/zh-CN/config-provider