---
title: "Vercel+Railway部署Typecho动态博客"
published: 2024-03-17
updated: 2024-03-17
description: ""
tags: ["Vercel","Railway"]
category: "计算机相关"
draft: false
---

**什么是vercel？**

 - 快速部署：Vercel是一个serverless平台，可以快速部署各种类型的应用程序，而且无需担心服务器的配置和管理问题。
 - 自动构建：Vercel支持自动构建，只需要在Github或Gitlab上提交代码，Vercel就会自动构建和部署应用程序。
 - 免费托管：Vercel提供免费的托管服务，可以免费托管静态网站、单页应用程序和服务器端渲染应用程序，并且没有流量限制。
 - CDN加速：Vercel使用全球CDN加速，可以提高应用程序的访问速度和稳定性。
 - 移动端优化：Vercel支持移动端优化，可以自动适配不同的移动设备和屏幕尺寸，让应用程序在移动端展示更加优美。
 - 支持多种语言和框架：Vercel支持多种编程语言和框架，如React、Vue、Next.js、Gatsby等，可以满足不同开发者的需求。
 - 安全可靠：Vercel提供了高级的安全功能，如SSL证书、DDoS防护、安全扫描等，可以保障应用程序的安全性和可靠性。

**什么是railway?**

 - Railway是一个基于Docker的云原生应用托管平台，可以让开发者轻松地将应用部署到云端。它支持多种编程语言和框架，如Node.js、Python、Ruby等，同时也支持多种数据库和缓存服务，如MySQL、PostgreSQL、Redis等。开发者可以通过简单的命令行操作，快速部署和管理自己的应用程序。Railway还提供了一些有用的功能，如自动扩展、自动备份、SSL证书、日志管理等，可以帮助开发者更好地管理和维护自己的应用程序。此外，Railway还提供了一个应用市场，开发者可以在市场中找到各种有用的应用程序和服务，如Wordpress、Ghost、Discourse等，以及各种API服务，如短信、邮件、推送等。总之，Railway是一个功能强大、易于使用、灵活扩展的云原生应用托管平台，可以帮助开发者更好地构建、部署和管理自己的应用程序。


**修改typecho的install.php，定位到第773行至775行，注释掉下面三行**

    // if (!$writeable) {
    //     $errors[] = _t(\'上传目录无法写入, 请手动将安装目录下的 %s 目录的权限设置为可写然后继续升级\', $uploadDir);
    // }

**在根目录下添加vercel.json**

    {
      \"functions\": {
        \"api/index.php\": {
          \"runtime\": \"vercel-php@0.7.0\"  //注意这里的PHP版本
        }
      },
      \"routes\": [
        {
          \"src\": \"/(.*)\",
          \"dest\": \"/api/index.php\"
        }
      ]
    }

**在根目录下创建 api 目录并在目录下添加 index.php**

    <?php
    $file= __DIR__ . \'/..\'.$_SERVER[\"PHP_SELF\"];
    if(file_exists($file))
    {
       return false;
    }
    else
    {
        require_once __DIR__ . \'/../index.php\';
    }
    #echo $_SERVER[\"PHP_SELF\"];

**在根目录下创建 config.inc.php**

    <?php
    /**
     * Typecho Blog Platform
     *
     * @copyright  Copyright (c) 2008 Typecho team (http://www.typecho.org)
     * @license    GNU General Public License 2.0
     * @version    $Id$
     */
    /** 开启https */
    define(\'__TYPECHO_SECURE__\',true);
    /** 定义根目录 */
    define(\'__TYPECHO_ROOT_DIR__\', dirname(__FILE__));
    /** 定义插件目录(相对路径) */
    define(\'__TYPECHO_PLUGIN_DIR__\', \'/usr/plugins\');
    /** 定义模板目录(相对路径) */
    define(\'__TYPECHO_THEME_DIR__\', \'/usr/themes\');
    /** 后台路径(相对路径) */
    define(\'__TYPECHO_ADMIN_DIR__\', \'/admin/\');
    /** 设置包含路径 */
    @set_include_path(get_include_path() . PATH_SEPARATOR .
    __TYPECHO_ROOT_DIR__ . \'/var\' . PATH_SEPARATOR .
    __TYPECHO_ROOT_DIR__ . __TYPECHO_PLUGIN_DIR__);
    /** 载入API支持 */
    require_once \'Typecho/Common.php\';
    /** 程序初始化 */
    Typecho_Common::init();
    /** 定义数据库参数 */
    $db = new Typecho_Db(\'Pdo_Mysql\', \'typecho_\');
    $db->addServer(array (
      \'host\' => \'数据库地址\', //修改在railway上的对应内容
      \'user\' => \'数据库用户名\',
      \'password\' => \'数据库密码\',
      \'charset\' => \'utf8mb4\',
      \'port\' => \'数据库端口号\',
      \'database\' => \'数据库名称\',
      \'engine\' => \'MyISAM\',
    ), Typecho_Db::READ | Typecho_Db::WRITE);
    Typecho_Db::set($db);

  **直接在vercel一键部署，打开/install.php 一步一步完成安装即可（域名自行修改）**