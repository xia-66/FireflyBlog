---
title: "NPM更新版本号"
published: 2025-04-12
description: ""
category: "默认分类"
draft: false
---

要使用npm命令更新package.json中的版本号，可以使用npm version命令。以下是几种常见的版本更新方式：

1. 更新补丁版本号（1.0.3 → 1.0.4）:

```
npm version patch
```

2. 更新次版本号（1.0.3 → 1.1.0）:

```
npm version minor
```

3. 更新主版本号（1.0.3 → 2.0.0）:

```
npm version major
```

4. 更新到特定版本号:

```
npm version 1.2.3
```

5. 如果您想添加提交信息:

```
npm version patch -m \"版本升级到 %s\"
```

这些命令会自动更新package.json中的版本号，同时如果项目使用Git管理，还会创建相应的Git标签。执行命令后，版本号会立即更新。
