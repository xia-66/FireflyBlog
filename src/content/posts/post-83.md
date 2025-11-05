---
title: "将图片Buffer数据转化为Blob对象，再转化为URL渲染"
published: 2024-07-31
updated: 2024-07-31
description: ""
tags: ["Vue3","Buffer"]
category: "前端相关"
draft: false
---

微信小程序会返回图片Buffer数据，本来想通过base64编码返回给前端，但搜索后了解到Base64编码的图片会增加大约33%的大小，因为Base64使用4个字符来表示3个字节的数据，对于较大的图片，使用Base64编码可能会导致性能问题，因为它会增加页面的加载时间，换种解决方法先转化为Blob对象，再转化为URL渲染，具体代码如下

    <template>
      <img :src=\"imageSrc\" alt=\"Buffer Image\" style=\"width:200px;height:200px\">
    </template>
    <script setup>
    import { ref } from \'vue\';
    const imageSrc= ref(null)
    const bufferToBlob = (buffer, mimeType) => {
      // 创建一个包含Buffer数据的Uint8Array
      const uint8Array = new Uint8Array(buffer);
      // 使用Blob构造函数创建Blob对象
      const blob = new Blob([uint8Array], { type: mimeType });
      return blob;
    }
    // 指定MIME类型
    const mimeType = \'image/png\';
    // 调用方法转换Buffer为Blob
    const buffer =  [] 
    const blob = bufferToBlob(buffer, mimeType);
    // 现在你可以使用这个Blob对象了，例如用于FileSaver.js等
    console.log(blob);
    // 将Blob转换为URL
    imageSrc.value = URL.createObjectURL(blob);
    </script>

**常见MIME类型：**

 - text/html：HTML文档。
 - text/plain：纯文本文件。
 - text/css：CSS样式表。
 - application/json：JSON格式数据。
 - application/javascript：JavaScript代码。
 - image/jpeg：JPEG图片格式。
 - image/png：PNG图片格式。
 - image/gif：GIF图片格式。
 - audio/mpeg：MP3音频格式。
 - video/mp4：MP4视频格式。
