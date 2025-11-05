---
title: "Vue3下载后端传过来的流文件(excel）"
published: 2024-10-16
updated: 2024-10-16
description: ""
tags: ["Vue3"]
category: "前端相关"
draft: false
---

请求接口

    export function outSetExcel(data){
      return request({
          //responseType响应返回的数据类型
          responseType: \'arraybuffer\',
          url:\'/app/warn/warnConfigExcel\',
          method:\"POST\",
          data
      })
    }

**方法一、使用 js-file-download**  
    //安装第三方库
    npm install js-file-download --save

    //this.tableQuery为导出参数、res.data为数据流
      outSetExcel(this.tableQuery).then(res => {
      fileDownload(res.data, \'告警历史.xls\')
    })

**方法二、使用Blob对象**

    const link = document.createElement(\'a\')
    outSetExcel(this.tableQuery).then(res => {
      // 创建Blob对象，设置文件类型
      let blob = new Blob([res.data], {type: \"application/vnd.ms-excel\"})
      let objectUrl = URL.createObjectURL(blob) // 创建URL
      link.href = objectUrl
      link.download = \'xxx\' // 自定义文件名
      link.click() // 下载文件
      URL.revokeObjectURL(objectUrl); // 释放内存
    }）