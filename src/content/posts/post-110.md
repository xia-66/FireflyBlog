---
title: "版权年份及版本号自动更新"
published: 2024-12-31
updated: 2024-12-31
description: ""
category: "软件相关"
draft: false
---

**版权年份**

    <span>&copy; {{ copyrightYears }} All rights reserved. </span>
    
    <script setup>
    import { computed } from \'vue\'
    
    const START_YEAR = 2024
    const currentYear = new Date().getFullYear()
    
    const copyrightYears = computed(() => {
      return currentYear > START_YEAR 
        ? `${START_YEAR}-${currentYear}`
        : START_YEAR
    })
    </script>


**版本号**

    <span> &nbsp; {{ RELEASE }}</span>
    
    import * as pkg from \'../../package.json\'
    
    const RELEASE = `v${pkg.version}`

**使用 npm version 命令自动更新版本号**

例如：
npm version patch 更新补丁版本 (1.0.0 -> 1.0.1)
npm version minor 更新次版本 (1.0.0 -> 1.1.0)
npm version major 更新主版本 (1.0.0 -> 2.0.0)