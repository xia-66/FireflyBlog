---
title: "浏览器禁止右键、F12键复制及打开控制台"
published: 2023-12-31
updated: 2024-03-02
description: ""
category: "代码相关"
draft: false
---

一、主流浏览器禁止右键（火狐浏览器除外）
  
      <script language=\"Javascript\">
        document.oncontextmenu=new Function(\"event.returnValue=false\");
        document.onselectstart=new Function(\"event.returnValue=false\");
      </script>

二、火狐浏览器禁止右键

      <script type=\"text/javascript\">
        document.oncontextmenu=function(e){return false;}
      </script>

      <style>
        body {-moz-user-select:none;} //禁止文字选中
      </style>

三、禁用F12键

      <script type=\"text/javascript\">
            document.onkeydown = function () {
                if (window.event && window.event.keyCode == 123) {
                    event.keyCode = 0;
                    event.returnValue = false;
                    return false;
                }
            };
      </script>

