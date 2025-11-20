---
title: "部署MySQL5.7.44的docker compose"
published: 2024-09-23
updated: 2024-09-29
description: "使用Docker Compose部署MySQL 5.7.44数据库的完整指南"
tags: ["Docker", "MySQL", "数据库"]
category: "软件相关"
draft: false
---

# 部署MySQL5.7.44的docker compose

## 🧑‍💻 User

编写一个docker-compose用于部署5.7.44

---
## 🤖 Assistant

好的，这里为您提供一个用于部署 MySQL 5.7.44 的 `docker-compose.yml` 文件，并附上详细的说明和不同级别的配置方案。

---

### 方案一：基础实用版

这个版本包含了运行一个 MySQL 5.7.44 实例所需的核心配置，适合快速开发和测试。

**文件结构：**

```
mysql-5.7/
└── docker-compose.yml
```

**`docker-compose.yml` 文件内容：**

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:5.7.44
    container_name: mysql57_server
    restart: unless-stopped
    environment:
      # --- 关键配置：请务必修改密码 ---
      MYSQL_ROOT_PASSWORD: your_strong_root_password  # 设置 root 用户的密码
      MYSQL_DATABASE: my_app_db                     # [可选] 启动时创建的数据库名
      MYSQL_USER: my_app_user                       # [可选] 启动时创建的普通用户名
      MYSQL_PASSWORD: your_strong_user_password     # [可选] 普通用户的密码
    ports:
      # 将主机的 3306 端口映射到容器的 3306 端口
      # 如果主机 3306 端口被占用，可以修改为 "3307:3306"
      - "3306:3306"
    volumes:
      # 将数据库数据持久化到本地，防止容器删除后数据丢失
      # 'mysql-data' 是一个 Docker 托管的卷 (named volume)
      - mysql-data:/var/lib/mysql
    networks:
      - my-network

# 定义 Docker 托管的卷，用于持久化数据
volumes:
  mysql-data:

# 定义网络，方便其他容器连接
networks:
  my-network:
```

#### 文件解析

* `version: '3.8'`: 指定 Docker Compose 文件的版本。
* `services`: 定义所有服务的容器。
* `mysql`: 服务的名称。
* `image: mysql:5.7.44`: 指定使用的镜像及其版本。
* `container_name: mysql57_server`: 为容器指定一个固定的、易于识别的名称。
* `restart: unless-stopped`: 除非手动停止，否则容器总是在退出后自动重启。
* `environment`: 设置环境变量，用于初始化 MySQL。
 * `MYSQL_ROOT_PASSWORD`: **必须设置**，这是 root 用户的密码。
 * `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`: 这三个是可选的。如果设置了，Docker 会在初始化时自动创建指定的数据库和用户，并授予该用户对该数据库的完全权限。这是非常推荐的做法，避免在应用中使用 root 账户。
* `ports`: 端口映射。`"主机端口:容器端口"`。这里将本地的 3306 端口映射到容器内部的 3306 端口，这样你就可以通过 `localhost:3306` 来访问数据库了。
* `volumes`: 数据卷挂载。
 * `mysql-data:/var/lib/mysql`: 这是最关键的部分。它将容器内的 `/var/lib/mysql` 目录（MySQL 存储数据的地方）挂载到一个名为 `mysql-data` 的 Docker 卷上。这样即使容器被删除 (`docker-compose down`)，数据也会被保留。下次启动时会重新加载。
* `networks`: 定义一个自定义网络。让 MySQL 和其他应用（如你的后端服务）在同一个网络中，它们就可以通过服务名（例如 `mysql`）直接通信，而不需要暴露端口。

#### 如何使用

1. 创建一个目录，例如 `mysql-5.7`。
2. 在目录中创建 `docker-compose.yml` 文件，并将上面的内容复制进去。
3. **重要：** 将 `your_strong_root_password` 和 `your_strong_user_password` 修改为你自己的强密码。
4. 在终端中，进入该目录，然后运行：
```bash
    docker-compose up -d
    ```
    `-d` 参数表示在后台运行。
5.  检查容器是否成功运行：
    ```bash
    docker-compose ps
    ```
6.  **连接数据库：**
    *   **主机地址:** `127.0.0.1` 或 `localhost`
    *   **端口:** `3306`
    *   **用户名:** `root` 或 `my_app_user`
    *   **密码:** 你设置的密码

---

### 方案二：生产环境推荐版

这个版本在基础版之上，增加了字符集、时区和自定义配置文件的支持，更适合生产环境。

**文件结构：**

```
mysql-5.7/
├── docker-compose.yml
└── config/
 └── my.cnf
```

**`config/my.cnf` 文件内容：**

```ini
[mysqld]
# 设置默认字符集为 utf8mb4
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
# 开启查询缓存（在 5.7 中仍然可用，但 8.0 已移除）
# query_cache_type = 1
# query_cache_size = 64M

[client]
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4
```

**`docker-compose.yml` 文件内容：**

```yaml
version: '3.8'

services:
mysql:
 image: mysql:5.7.44
 container_name: mysql57_server
 restart: unless-stopped
 environment:
 # --- 关键配置：请务必修改密码 ---
 MYSQL_ROOT_PASSWORD: your_strong_root_password
 MYSQL_DATABASE: my_app_db
 MYSQL_USER: my_app_user
 MYSQL_PASSWORD: your_strong_user_password
 # 设置时区为亚洲/上海
 TZ: Asia/Shanghai
 ports:
 - "3306:3306"
 volumes:
 # 数据持久化
 - mysql-data:/var/lib/mysql
 # 挂载自定义配置文件
 - ./config/my.cnf:/etc/mysql/conf.d/my_custom.cnf
 # [可选] 挂载初始化脚本
 # - ./init:/docker-entrypoint-initdb.d
 networks:
 - my-network

volumes:
mysql-data:

networks:
my-network:
```

#### 新增配置解析

* `environment.TZ: Asia/Shanghai`: 设置容器的时区，确保数据库中的时间戳是正确的。你可以根据需要修改为其他时区。
* `volumes`:
 * `./config/my.cnf:/etc/mysql/conf.d/my_custom.cnf`: 这一行将你本地的 `config/my.cnf` 文件挂载到容器的 `/etc/mysql/conf.d/` 目录下。MySQL 启动时会自动加载这个目录下的所有 `.cnf` 配置文件。通过这种方式，你可以轻松地自定义 MySQL 的各项参数，如字符集、缓冲池大小、最大连接数等。
 * `./init:/docker-entrypoint-initdb.d`: 这是一个可选的挂载。如果你需要在数据库第一次初始化时执行一些 SQL 脚本（例如创建复杂的表结构、插入初始数据），可以把 `.sql`、`.sql.gz` 或 `.sh` 文件放在本地的 `init` 目录下，Docker 会自动执行它们。

#### 如何使用

1. 创建 `mysql-5.7` 目录，并在其中创建 `config` 子目录。
2. 在 `mysql-5.7` 目录下创建 `docker-compose.yml` 文件。
3. 在 `config` 目录下创建 `my.cnf` 文件。
4. 将上面对应的内容复制到文件中，并修改密码。
5. 在 `mysql-5.7` 目录下运行 `docker-compose up -d`。

这个方案提供了更大的灵活性和更好的实践，强烈推荐在正式项目中使用。

---
## 🧑‍💻 User

继续修改utf8mb4，数据库名和用户名为xboard,密码为AtB7b8MwCJy3xDeh

---
## 🤖 Assistant

好的，我们来根据您的新要求进行修改。

这个版本将直接配置好 `utf8mb4` 字符集，并使用您指定的数据库名 `xboard`、用户名 `xboard` 和密码 `AtB7b8MwCJy3xDeh`。

这是一个更贴近实际项目使用的配置。

---

### 最终配置方案

**文件结构：**

建议您这样组织文件，方便管理：

```
xboard-mysql/
├── docker-compose.yml
└── config/
    └── my.cnf
```

#### 第 1 步：创建 `config/my.cnf` 文件

这个文件专门用来强制 MySQL 使用 `utf8mb4` 字符集，这是现代应用的最佳实践，可以完美支持 Emoji 等特殊字符。

**`config/my.cnf` 内容：**

```ini
[mysqld]
# 设置服务端默认字符集为 utf8mb4
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
# 开启 sql_mode 的一些严格模式，有助于保证数据一致性
# 如果旧系统不兼容，可以注释掉下面这行
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

[client]
# 设置客户端连接默认字符集
default-character-set=utf8mb4

[mysql]
# 设置 mysql 命令行工具默认字符集
default-character-set=utf8mb4
```

---

#### 第 2 步：创建 `docker-compose.yml` 文件

这个文件是核心，它定义了如何运行 MySQL 容器，并使用了您提供的凭据。

**`docker-compose.yml` 内容：**

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:5.7.44
    container_name: mysql_xboard_57
    restart: unless-stopped
    environment:
      # ### 警告：请务必为生产环境修改 root 密码！###
      MYSQL_ROOT_PASSWORD: CHANGE_ME_TO_A_VERY_STRONG_SECRET_PASSWORD

      # --- 根据您的要求配置 ---
      MYSQL_DATABASE: xboard
      MYSQL_USER: xboard
      MYSQL_PASSWORD: AtB7b8MwCJy3xDeh
      
      # 设置时区为中国上海
      TZ: Asia/Shanghai
    ports:
      # 将主机的 3306 端口映射到容器的 3306 端口
      - "3306:3306"
    volumes:
      # 数据持久化，防止容器删除后数据丢失
      - mysql-data:/var/lib/mysql
      # 挂载自定义配置文件，确保使用 utf8mb4
      - ./config/my.cnf:/etc/mysql/conf.d/charset.cnf
    networks:
      - xboard-net

# Docker 托管的数据卷
volumes:
  mysql-data:

# 自定义网络，方便其他服务（如后端应用）连接
networks:
  xboard-net:
```

#### 配置解析

* `MYSQL_ROOT_PASSWORD`: **非常重要！** 我在这里设置了一个占位符 `CHANGE_ME_TO_A_VERY_STRONG_SECRET_PASSWORD`。请务必将其修改为一个**与应用密码不同**的、非常强大的密码。这是数据库的最高权限，泄露风险极高。
* `MYSQL_DATABASE: xboard`: 启动时自动创建名为 `xboard` 的数据库。
* `MYSQL_USER: xboard`: 启动时自动创建名为 `xboard` 的用户。
* `MYSQL_PASSWORD: AtB7b8MwCJy3xDeh`: 为 `xboard` 用户设置密码。该用户将自动拥有对 `xboard` 数据库的完全权限。
* `./config/my.cnf:/etc/mysql/conf.d/charset.cnf`: 将我们创建的 `my.cnf` 文件挂载到容器中，MySQL 启动时会加载它，从而应用 `utf8mb4` 配置。

---

### 如何使用

1. 创建一个名为 `xboard-mysql` 的文件夹。
2. 在 `xboard-mysql` 文件夹内，创建一个 `config` 子文件夹。
3. 将 `my.cnf` 的内容保存到 `xboard-mysql/config/my.cnf` 文件中。
4. 将 `docker-compose.yml` 的内容保存到 `xboard-mysql/docker-compose.yml` 文件中。
5. **在启动前，请务必修改 `docker-compose.yml` 文件中的 `MYSQL_ROOT_PASSWORD` 的值！**
6. 打开终端，进入 `xboard-mysql` 目录，然后运行以下命令启动服务：
 ```bash
 docker-compose up -d
 ```
 * `-d` 参数表示在后台（detached mode）运行。

7. 检查容器状态：
 ```bash
 docker-compose ps
 ```
 您应该能看到 `mysql_xboard_57` 容器正在运行 (`Up`)。

### 连接信息

现在，您可以使用任何数据库客户端工具通过以下信息连接到数据库：

* **主机 (Host):** `127.0.0.1` (或 `localhost`)
* **端口 (Port):** `3306`
* **应用用户名 (User):** `xboard`
* **应用密码 (Password):** `AtB7b8MwCJy3xDeh`
* **数据库 (Database):** `xboard`

* **管理员用户名 (Root User):** `root`
* **管理员密码 (Root Password):** 您为 `MYSQL_ROOT_PASSWORD` 设置的密码。

---
## 🧑‍💻 User

-- MySQL dump 10.13 Distrib 5.7.44, for Linux (x86_64)
--
-- Host: localhost Database: xboard
-- ------------------------------------------------------
-- Server version	5.7.44-log

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `failed_jobs`
--

DROP TABLE IF EXISTS `failed_jobs`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `failed_jobs` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`connection` text COLLATE utf8mb4_unicode_ci NOT NULL,
`queue` text COLLATE utf8mb4_unicode_ci NOT NULL,
`payload` longtext COLLATE utf8mb4_unicode_ci NOT NULL,
`exception` longtext COLLATE utf8mb4_unicode_ci NOT NULL,
`failed_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `failed_jobs`
--

LOCK TABLES `failed_jobs` WRITE;
/*!40000 ALTER TABLE `failed_jobs` DISABLE KEYS */;
/*!40000 ALTER TABLE `failed_jobs` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `migrations`
--

DROP TABLE IF EXISTS `migrations`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `migrations` (
`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
`migration` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`batch` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=30 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `migrations`
--

LOCK TABLES `migrations` WRITE;
/*!40000 ALTER TABLE `migrations` DISABLE KEYS */;
INSERT INTO `migrations` VALUES (1,'2019_08_19_000000_create_failed_jobs_table',1),(2,'2019_12_14_000001_create_personal_access_tokens_table',1),(3,'2023_03_19_000000_create_v2_tables',1),(4,'2023_08_14_221234_create_v2_settings_table',1),(5,'2023_09_04_190923_add_column_excludes_to_server_table',1),(6,'2023_09_06_195956_add_column_ips_to_server_table',1),(7,'2023_09_14_013244_add_column_alpn_to_server_hysteria_table',1),(8,'2023_09_24_040317_add_column_network_and_network_settings_to_v2_server_trojan',1),(9,'2023_09_29_044957_add_column_version_and_is_obfs_to_server_hysteria_table',1),(10,'2023_11_19_205026_change_column_value_to_v2_settings_table',1),(11,'2023_12_12_212239_add_index_to_v2_user_table',1),(12,'2024_03_19_103149_modify_icon_column_to_v2_payment_table',1),(13,'2025_01_01_130644_modify_commission_status_in_v2_order_table',1),(14,'2025_01_04_optimize_plan_table',1),(15,'2025_01_05_131425_create_v2_server_table',1),(16,'2025_01_10_152139_add_device_limit_column',1),(17,'2025_01_12_190315_add_sort_to_v2_notice_table',1),(18,'2025_01_12_200936_modify_commission_status_in_v2_order_table',1),(19,'2025_01_13_000000_convert_order_period_fields',1),(20,'2025_01_15_000002_add_stat_performance_indexes',1),(21,'2025_01_16_142320_add_updated_at_index_to_v2_order_table',1),(22,'2025_01_18_140511_create_plugins_table',1),(23,'2025_06_21_000001_optimize_v2_settings_table',1),(24,'2025_06_21_000002_create_traffic_reset_logs_table',1),(25,'2025_06_21_000003_add_traffic_reset_fields_to_users',1),(26,'2025_07_01_081556_add_tags_to_v2_plan_table',1),(27,'2025_07_01_122908_create_gift_card_tables',1),(28,'2025_07_13_224539_add_column_rate_time_ranges_to_v2_server_table',1),(29,'2025_07_26_000001_add_type_to_plugins_table',1);
/*!40000 ALTER TABLE `migrations` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `personal_access_tokens`
--

DROP TABLE IF EXISTS `personal_access_tokens`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `personal_access_tokens` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`tokenable_type` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`tokenable_id` bigint(20) unsigned NOT NULL,
`name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`token` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
`abilities` text COLLATE utf8mb4_unicode_ci,
`last_used_at` timestamp NULL DEFAULT NULL,
`expires_at` timestamp NULL DEFAULT NULL,
`created_at` timestamp NULL DEFAULT NULL,
`updated_at` timestamp NULL DEFAULT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `personal_access_tokens_token_unique` (`token`),
KEY `personal_access_tokens_tokenable_type_tokenable_id_index` (`tokenable_type`,`tokenable_id`)
) ENGINE=InnoDB AUTO_INCREMENT=18 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `personal_access_tokens`
--

LOCK TABLES `personal_access_tokens` WRITE;
/*!40000 ALTER TABLE `personal_access_tokens` DISABLE KEYS */;
INSERT INTO `personal_access_tokens` VALUES (1,'App\\Models\\User',2,'th2xciWNde16ossHBASM','f0b955acbedb1fb869dd69c285436ec4be6a048f3ec390cc01ca258d03454a6d','[\"*\"]','2025-11-18 06:35:38','2026-11-18 06:35:38','2025-11-18 06:35:38','2025-11-18 06:35:38'),(2,'App\\Models\\User',1,'JM368Z0adq6R9v3TbJ4y','da90df75c575fe98d88e6ec91629c8b7e8ec3fefdc2cb541bdb2d256123241e1','[\"*\"]','2025-11-18 06:48:33','2026-11-18 06:36:06','2025-11-18 06:36:06','2025-11-18 06:48:33'),(3,'App\\Models\\User',1,'LIhAm9bhx7bUAgZrePN4','5ae66021cf4a7ac78ba77293c67d619d78bc3bc75e55fd10f747c4416567ecb0','[\"*\"]','2025-11-18 07:03:53','2026-11-18 06:48:56','2025-11-18 06:48:56','2025-11-18 07:03:53'),(4,'App\\Models\\User',1,'vf0fKGa4hmdRINfoffVg','e81287fa2d375ff03b29f3dee814ae9d5759e9e5b42d4c043643b797bdd0811a','[\"*\"]','2025-11-18 07:13:53','2026-11-18 06:53:13','2025-11-18 06:53:13','2025-11-18 07:13:53'),(5,'App\\Models\\User',1,'SIx0RsG0duFqyaCmoUz5','c6fe5fb6a4bdeabdf4144519f43455e5d3a836fc70cdfe83c8af61714554c1b5','[\"*\"]','2025-11-18 08:26:10','2026-11-18 07:12:12','2025-11-18 07:12:12','2025-11-18 08:26:10'),(6,'App\\Models\\User',1,'I7L3Un9ChnIxBS2ANnZp','362028c34d0ddd06083581c808e72f63459f9910367aefef3927ca4d4f0b4790','[\"*\"]','2025-11-18 09:16:21','2026-11-18 08:23:13','2025-11-18 08:23:13','2025-11-18 09:16:21'),(7,'App\\Models\\User',1,'qoxtc9dzlPCXxbFNsika','e44ebaadd382adbf2ee76bb99106b04c524400886cf584c18d225ced049e9b38','[\"*\"]','2025-11-19 01:48:09','2026-11-18 08:26:45','2025-11-18 08:26:45','2025-11-19 01:48:09'),(8,'App\\Models\\User',1,'MJ6Y5QWs72dU5oPFukKt','b795bbfcad0d6f06f911ac6e946a2b2825c387f2aa67c36d7f24fa1d9922e3b0','[\"*\"]','2025-11-18 23:49:49','2026-11-18 09:03:38','2025-11-18 09:03:38','2025-11-18 23:49:49'),(9,'App\\Models\\User',2,'RGlFbBSwLR7l37Qgun0i','0750bb68703ea84a6e71d783985376c48503b26bc41aa827601625800b18f2ef','[\"*\"]','2025-11-18 09:08:17','2026-11-18 09:08:16','2025-11-18 09:08:16','2025-11-18 09:08:17'),(10,'App\\Models\\User',3,'BEfCI0uowpeXQryEmFu9','c95d258f14849adca52d10e8d45f5a1fb4b1e83998c2f1014b6dc27f33c15a48','[\"*\"]','2025-11-18 09:10:17','2026-11-18 09:09:13','2025-11-18 09:09:13','2025-11-18 09:10:17'),(11,'App\\Models\\User',4,'GALoQYC7hyCH8AGZpp8I','7d9a32ca48c07ec92cb7ac5594cf729c8032cf800ab4d2372712eb08484790ea','[\"*\"]','2025-11-18 09:19:58','2026-11-18 09:18:29','2025-11-18 09:18:29','2025-11-18 09:19:58'),(12,'App\\Models\\User',5,'IXZuoSEWlBf5jsD9HR6d','ff2188e06c9aba9a1f27625d37b9241a5f54cd9e9398036677423735197a22e4','[\"*\"]','2025-11-18 09:48:35','2026-11-18 09:47:28','2025-11-18 09:47:28','2025-11-18 09:48:35'),(13,'App\\Models\\User',2,'jxppyHq93WfwgLBt4zr8','9770e045a7d4985b11fd2e88de91b1ee0c9e15d97b3cabcf0c9149f04f2866b1','[\"*\"]','2025-11-18 15:02:30','2026-11-18 15:02:29','2025-11-18 15:02:29','2025-11-18 15:02:30'),(14,'App\\Models\\User',6,'2JHD0c6ZjdhIHQSARhfK','de51fedd14d8d87784ba2ceefd73eb85e7c5127fedf4dc8a9f3354d003971db5','[\"*\"]','2025-11-18 23:19:29','2026-11-18 23:19:28','2025-11-18 23:19:28','2025-11-18 23:19:29'),(15,'App\\Models\\User',1,'75ocLnH1AsSlwR5oSqgw','b5e6e0f8f89683d602fc5586b4a974ca826f4a7bb113a47cb3c24050249f51a4','[\"*\"]','2025-11-19 00:25:00','2026-11-19 00:24:59','2025-11-19 00:24:59','2025-11-19 00:25:00'),(16,'App\\Models\\User',2,'mIsAPEZ89OYPQZ6WVEgQ','c5cf71045af7be9e37ba61822a95fda566166c60e3a9bfc8fe26c5cea4e062fd','[\"*\"]','2025-11-19 00:46:49','2026-11-19 00:46:47','2025-11-19 00:46:47','2025-11-19 00:46:49'),(17,'App\\Models\\User',1,'e5Va1oOq50DAqYCLcnyg','ccabd7f53516545de7620bc5f4b10650f133d31ff3d0e3f16af35ec2795c3b33','[\"*\"]','2025-11-19 01:55:03','2026-11-19 01:54:40','2025-11-19 01:54:40','2025-11-19 01:55:03');
/*!40000 ALTER TABLE `personal_access_tokens` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_commission_log`
--

DROP TABLE IF EXISTS `v2_commission_log`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_commission_log` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`invite_user_id` int(11) NOT NULL,
`user_id` int(11) NOT NULL,
`trade_no` char(36) COLLATE utf8mb4_unicode_ci NOT NULL,
`order_amount` int(11) NOT NULL,
`get_amount` int(11) NOT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
KEY `v2_commission_log_created_at_index` (`created_at`),
KEY `v2_commission_log_get_amount_index` (`get_amount`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_commission_log`
--

LOCK TABLES `v2_commission_log` WRITE;
/*!40000 ALTER TABLE `v2_commission_log` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_commission_log` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_coupon`
--

DROP TABLE IF EXISTS `v2_coupon`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_coupon` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`code` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`type` int(11) NOT NULL,
`value` int(11) NOT NULL,
`show` tinyint(1) NOT NULL DEFAULT '0',
`limit_use` int(11) DEFAULT NULL,
`limit_use_with_user` int(11) DEFAULT NULL,
`limit_plan_ids` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`limit_period` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`started_at` int(11) NOT NULL,
`ended_at` int(11) NOT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_coupon`
--

LOCK TABLES `v2_coupon` WRITE;
/*!40000 ALTER TABLE `v2_coupon` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_coupon` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_gift_card_code`
--

DROP TABLE IF EXISTS `v2_gift_card_code`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_gift_card_code` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`template_id` int(11) NOT NULL COMMENT '模板ID',
`code` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '兑换码',
`batch_id` varchar(32) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '批次ID',
`status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态：0未使用 1已使用 2已过期 3已禁用',
`user_id` int(11) DEFAULT NULL COMMENT '使用用户ID',
`used_at` int(11) DEFAULT NULL COMMENT '使用时间',
`expires_at` int(11) DEFAULT NULL COMMENT '过期时间',
`actual_rewards` json DEFAULT NULL COMMENT '实际获得的奖励(用于盲盒等)',
`usage_count` int(11) NOT NULL DEFAULT '0' COMMENT '使用次数(分享卡)',
`max_usage` int(11) NOT NULL DEFAULT '1' COMMENT '最大使用次数',
`metadata` json DEFAULT NULL COMMENT '额外数据',
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `v2_gift_card_code_code_unique` (`code`),
KEY `idx_gift_code_template_id` (`template_id`),
KEY `idx_gift_code_status` (`status`),
KEY `idx_gift_code_user_id` (`user_id`),
KEY `idx_gift_code_batch_id` (`batch_id`),
KEY `idx_gift_code_expires_at` (`expires_at`),
KEY `idx_gift_code_lookup` (`code`,`status`,`expires_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_gift_card_code`
--

LOCK TABLES `v2_gift_card_code` WRITE;
/*!40000 ALTER TABLE `v2_gift_card_code` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_gift_card_code` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_gift_card_template`
--

DROP TABLE IF EXISTS `v2_gift_card_template`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_gift_card_template` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '礼品卡名称',
`description` text COLLATE utf8mb4_unicode_ci COMMENT '礼品卡描述',
`type` tinyint(4) NOT NULL COMMENT '卡片类型：1余额 2有效期 3流量 4重置包 5套餐 6组合 7盲盒 8任务 9等级 10节日',
`status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '状态：0禁用 1启用',
`conditions` json DEFAULT NULL COMMENT '使用条件配置',
`rewards` json NOT NULL COMMENT '奖励配置',
`limits` json DEFAULT NULL COMMENT '限制条件',
`special_config` json DEFAULT NULL COMMENT '特殊配置(节日时间、等级倍率等)',
`icon` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '卡片图标',
`background_image` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '背景图片URL',
`theme_color` varchar(7) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '#1890ff' COMMENT '主题色',
`sort` int(11) NOT NULL DEFAULT '0' COMMENT '排序',
`admin_id` int(11) NOT NULL COMMENT '创建管理员ID',
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
KEY `idx_gift_template_type_status` (`type`,`status`),
KEY `idx_gift_template_created_at` (`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_gift_card_template`
--

LOCK TABLES `v2_gift_card_template` WRITE;
/*!40000 ALTER TABLE `v2_gift_card_template` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_gift_card_template` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_gift_card_usage`
--

DROP TABLE IF EXISTS `v2_gift_card_usage`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_gift_card_usage` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`code_id` int(11) NOT NULL COMMENT '兑换码ID',
`template_id` int(11) NOT NULL COMMENT '模板ID',
`user_id` int(11) NOT NULL COMMENT '使用用户ID',
`invite_user_id` int(11) DEFAULT NULL COMMENT '邀请人ID',
`rewards_given` json NOT NULL COMMENT '实际发放的奖励',
`invite_rewards` json DEFAULT NULL COMMENT '邀请人获得的奖励',
`user_level_at_use` int(11) DEFAULT NULL COMMENT '使用时用户等级',
`plan_id_at_use` int(11) DEFAULT NULL COMMENT '使用时用户套餐ID',
`multiplier_applied` decimal(3,2) NOT NULL DEFAULT '1.00' COMMENT '应用的倍率',
`ip_address` varchar(45) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '使用IP地址',
`user_agent` text COLLATE utf8mb4_unicode_ci COMMENT '用户代理',
`notes` text COLLATE utf8mb4_unicode_ci COMMENT '备注',
`created_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
KEY `idx_gift_usage_code_id` (`code_id`),
KEY `idx_gift_usage_template_id` (`template_id`),
KEY `idx_gift_usage_user_id` (`user_id`),
KEY `idx_gift_usage_invite_user_id` (`invite_user_id`),
KEY `idx_gift_usage_created_at` (`created_at`),
KEY `idx_gift_usage_user_usage` (`user_id`,`created_at`),
KEY `idx_gift_usage_template_stats` (`template_id`,`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_gift_card_usage`
--

LOCK TABLES `v2_gift_card_usage` WRITE;
/*!40000 ALTER TABLE `v2_gift_card_usage` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_gift_card_usage` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_invite_code`
--

DROP TABLE IF EXISTS `v2_invite_code`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_invite_code` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`user_id` int(11) NOT NULL,
`code` char(32) COLLATE utf8mb4_unicode_ci NOT NULL,
`status` tinyint(1) NOT NULL DEFAULT '0',
`pv` int(11) NOT NULL DEFAULT '0',
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_invite_code`
--

LOCK TABLES `v2_invite_code` WRITE;
/*!40000 ALTER TABLE `v2_invite_code` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_invite_code` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_knowledge`
--

DROP TABLE IF EXISTS `v2_knowledge`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_knowledge` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`language` char(5) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '語言',
`category` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '分類名',
`title` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '標題',
`body` text COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '內容',
`sort` int(11) DEFAULT NULL COMMENT '排序',
`show` tinyint(1) NOT NULL DEFAULT '0' COMMENT '顯示',
`created_at` int(11) NOT NULL COMMENT '創建時間',
`updated_at` int(11) NOT NULL COMMENT '更新時間',
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_knowledge`
--

LOCK TABLES `v2_knowledge` WRITE;
/*!40000 ALTER TABLE `v2_knowledge` DISABLE KEYS */;
INSERT INTO `v2_knowledge` VALUES (1,'zh-CN','必看的文档','导入失败解决方案','部分软件可能因为兼容性问题，无法正常导入/使用，请使用订阅转换服务：[https://sublink.588235.xyz/](https://sublink.588235.xyz/)',NULL,1,1752307409,1752307483),(2,'zh-CN','必看的文档','FQ必读，防范未然，避免喝茶','## 禁忌 \r\n1.不要在任何外网平台谈论政治相关话题，这是大忌，除非你不在大陆混了；\r\n\r\n2.不要在X（推特）上关注反贼（比如李老师），更不要互动，有人因为这个被请喝茶，X可以看到关注的人；\r\n\r\n3.youtube（油管）可以关注反贼，因为其他人无法查看关注列表，但不要评论点赞视频；\r\n\r\n4.不要安装反诈APP，他不能帮你反诈，但能请你喝茶；\r\n\r\n5.不要在国内的社交平台谈论翻墙，机场，境外的APP等相关话题。特别是QQ，微信群，因为国内平台受到监管影响，会有监控，触发关键词会上报网警，之后会有当地派出所找到你；\r\n\r\n## 尽量避免 \r\n1.尽量避免使用国产输入法，输入法是很多人容易忽略的；\r\n\r\n2.避免DNS泄露，特别是clash用户。如果你不懂，请使用教程中推荐的客户端；\r\n\r\n3.如果你想在“世界遨游”，请尽量避免购买华为手机；\r\n\r\n4.如果你是+86注册的Telegram（TG），请在隐私中隐藏手机号；\r\n\r\n5.尽量避免使用国内邮箱在境外平台注册账号，最好使用Gmail或outlook等海外邮箱；\r\n\r\n## 风险\r\n1.翻墙的人数有很多，一般情况（比如色色，AI，论坛，聊天，学术等）都不会请你喝茶，所以不用太担心；\r\n\r\n2.如果你在学校，请注意有些学校内置的防火墙，能非常轻易找到你，尤其避免DNS泄露问题；\r\n\r\n3.即使知道你在翻墙，大多数情况也没什么事，除非当地公安需要你冲业绩了；\r\n\r\n\r\n## 被请喝茶怎么办\r\n \r\n1.保持冷静，想想你做了什么，叔叔一般都是掌握了你的一些证据才会叫你，叔叔指认的老实承认，但请不要多说；\r\n\r\n2.如果问你为什么翻墙，你可以说因为工作、学术等需要用到AI，MJ，谷歌学术之类的，或者饥渴了，想看色色；\r\n\r\n3.如果问你知道为什么叫你来吗？切忌不要上来全盘托出，装不知道，把问题交给他，有证据的指控就承认，无实质证据的均否认；\r\n\r\n4.如果问翻墙工具怎么来的，不要说朋友推荐或者合租之类的，你就说搜索如何用GPT，看到别人分享的就拿来用了；',NULL,1,1752307548,1752307551);
/*!40000 ALTER TABLE `v2_knowledge` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_log`
--

DROP TABLE IF EXISTS `v2_log`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_log` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`title` text COLLATE utf8mb4_unicode_ci NOT NULL,
`level` varchar(11) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`host` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`uri` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`method` varchar(11) COLLATE utf8mb4_unicode_ci NOT NULL,
`data` text COLLATE utf8mb4_unicode_ci,
`ip` varchar(128) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`context` text COLLATE utf8mb4_unicode_ci,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=142 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_log`
--

LOCK TABLES `v2_log` WRITE;
/*!40000 ALTER TABLE `v2_log` DISABLE KEYS */;
INSERT INTO `v2_log` VALUES (1,'Installed and enabled default plugin: epay','INFO','http://localhost','/','GET','[]','127.0.0.1','[]',1763447648,1763447648),(2,'Installed and enabled default plugin: alipay_f2f','INFO','http://localhost','/','GET','[]','127.0.0.1','[]',1763447648,1763447648),(3,'Installed and enabled default plugin: btcpay','INFO','http://localhost','/','GET','[]','127.0.0.1','[]',1763447648,1763447648),(4,'Installed and enabled default plugin: coinbase','INFO','http://localhost','/','GET','[]','127.0.0.1','[]',1763447648,1763447648),(5,'Installed and enabled default plugin: coin_payments','INFO','http://localhost','/','GET','[]','127.0.0.1','[]',1763447648,1763447648),(6,'Installed and enabled default plugin: mgate','INFO','http://localhost','/','GET','[]','127.0.0.1','[]',1763447648,1763447648),(7,'Installed and enabled default plugin: telegram','INFO','http://localhost','/','GET','[]','127.0.0.1','[]',1763447648,1763447648),(8,'Theme initialized in public directory','INFO','https://588235.xyz','/','GET','[]','23.158.104.250','{\"theme\":\"Xboard\"}',1763447740,1763447740),(9,'POST api/v2/2b1145b9/ticket/fetch','INFO','http://27.106.122.68:7001','/api/v2/2b1145b9/ticket/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[{\"id\":\"status\",\"value\":\"0\"}],\"sort\":[]}','23.158.104.250','[]',1763447772,1763447772),(10,'POST api/v2/2b1145b9/config/save','INFO','http://27.106.122.68:7001','/api/v2/2b1145b9/config/save','POST','{\"app_name\":\"XBoard\",\"app_description\":\"XBoard is best!\",\"app_url\":null,\"force_https\":0,\"logo\":null,\"subscribe_url\":null,\"tos_url\":null,\"stop_register\":1,\"try_out_plan_id\":0,\"currency\":\"CNY\",\"currency_symbol\":\"\\u00a5\",\"try_out_hour\":1}','23.158.104.250','[]',1763447778,1763447778),(11,'POST api/v2/2b1145b9/user/fetch','INFO','http://27.106.122.68:7001','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763447781,1763447781),(12,'POST api/v2/2b1145b9/user/destroy','INFO','http://27.106.122.68:7001','/api/v2/2b1145b9/user/destroy','POST','{\"id\":2}','23.158.104.250','[]',1763447785,1763447785),(13,'POST api/v2/2b1145b9/user/fetch','INFO','http://27.106.122.68:7001','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763447785,1763447785),(14,'POST api/v2/2b1145b9/user/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448540,1763448540),(15,'POST api/v2/2b1145b9/user/update','INFO','https://588235.xyz','/api/v2/2b1145b9/user/update','POST','{\"id\":1,\"password\":\"918822gjy\"}','23.158.104.250','[]',1763448551,1763448551),(16,'POST api/v2/2b1145b9/user/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448552,1763448552),(17,'POST api/v2/2b1145b9/user/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448565,1763448565),(18,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"app_name\":\"HeiYu-Air\",\"app_description\":\"XBoard is best!\",\"app_url\":null,\"force_https\":0,\"logo\":null,\"subscribe_url\":null,\"tos_url\":null,\"stop_register\":1,\"try_out_plan_id\":0,\"currency\":\"CNY\",\"currency_symbol\":\"\\u00a5\",\"try_out_hour\":1}','23.158.104.250','[]',1763448594,1763448594),(19,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"app_name\":\"HeiYu-Air\",\"app_description\":\"\\u81f4\\u656c\\u4f20\\u5947\\u673a\\u957f\\u9ed1\\u7fbd\\uff08\\u5f3a\\u5236\\u9080\\u8bf7\\u5236\\uff0c\\u6ca1\\u6709\\u9080\\u8bf7\\u7801\\u53ef\\u4ee5\\u5148\\u79bb\\u5f00\\u4e86~\\uff09\",\"app_url\":null,\"force_https\":0,\"logo\":null,\"subscribe_url\":null,\"tos_url\":null,\"stop_register\":1,\"try_out_plan_id\":0,\"currency\":\"CNY\",\"currency_symbol\":\"\\u00a5\",\"try_out_hour\":1}','23.158.104.250','[]',1763448604,1763448604),(20,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"app_name\":\"HeiYu-Air\",\"app_description\":\"\\u81f4\\u656c\\u4f20\\u5947\\u673a\\u957f\\u9ed1\\u7fbd\\uff08\\u5f3a\\u5236\\u9080\\u8bf7\\u5236\\uff0c\\u6ca1\\u6709\\u9080\\u8bf7\\u7801\\u53ef\\u4ee5\\u5148\\u79bb\\u5f00\\u4e86~\\uff09\",\"app_url\":\"https:\\/\\/588235.xyz\",\"force_https\":0,\"logo\":null,\"subscribe_url\":null,\"tos_url\":null,\"stop_register\":1,\"try_out_plan_id\":0,\"currency\":\"CNY\",\"currency_symbol\":\"\\u00a5\",\"try_out_hour\":1}','23.158.104.250','[]',1763448612,1763448612),(21,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"app_name\":\"HeiYu-Air\",\"app_description\":\"\\u81f4\\u656c\\u4f20\\u5947\\u673a\\u957f\\u9ed1\\u7fbd\\uff08\\u5f3a\\u5236\\u9080\\u8bf7\\u5236\\uff0c\\u6ca1\\u6709\\u9080\\u8bf7\\u7801\\u53ef\\u4ee5\\u5148\\u79bb\\u5f00\\u4e86~\\uff09\",\"app_url\":\"https:\\/\\/588235.xyz\",\"force_https\":1,\"logo\":null,\"subscribe_url\":null,\"tos_url\":null,\"stop_register\":1,\"try_out_plan_id\":0,\"currency\":\"CNY\",\"currency_symbol\":\"\\u00a5\",\"try_out_hour\":1}','23.158.104.250','[]',1763448613,1763448613),(22,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_verify\":true,\"safe_mode_enable\":false,\"secure_path\":\"2b1145b9\",\"email_whitelist_enable\":false,\"email_whitelist_suffix\":[\"gmail.com\",\"qq.com\",\"163.com\",\"yahoo.com\",\"sina.com\",\"126.com\",\"outlook.com\",\"yeah.net\",\"foxmail.com\"],\"email_gmail_limit_enable\":false,\"captcha_enable\":false,\"captcha_type\":\"recaptcha\",\"recaptcha_key\":null,\"recaptcha_site_key\":null,\"recaptcha_v3_secret_key\":null,\"recaptcha_v3_site_key\":null,\"recaptcha_v3_score_threshold\":\"0.5\",\"turnstile_secret_key\":null,\"turnstile_site_key\":null,\"register_limit_by_ip_enable\":false,\"register_limit_count\":\"3\",\"register_limit_expire\":\"60\",\"password_limit_enable\":true,\"password_limit_count\":\"5\",\"password_limit_expire\":\"60\",\"recaptcha_enable\":false}','23.158.104.250','[]',1763448622,1763448622),(23,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_verify\":false,\"safe_mode_enable\":false,\"secure_path\":\"2b1145b9\",\"email_whitelist_enable\":false,\"email_whitelist_suffix\":[\"gmail.com\",\"qq.com\",\"163.com\",\"yahoo.com\",\"sina.com\",\"126.com\",\"outlook.com\",\"yeah.net\",\"foxmail.com\"],\"email_gmail_limit_enable\":false,\"captcha_enable\":false,\"captcha_type\":\"recaptcha\",\"recaptcha_key\":null,\"recaptcha_site_key\":null,\"recaptcha_v3_secret_key\":null,\"recaptcha_v3_site_key\":null,\"recaptcha_v3_score_threshold\":\"0.5\",\"turnstile_secret_key\":null,\"turnstile_site_key\":null,\"register_limit_by_ip_enable\":false,\"register_limit_count\":\"3\",\"register_limit_expire\":\"60\",\"password_limit_enable\":true,\"password_limit_count\":\"5\",\"password_limit_expire\":\"60\",\"recaptcha_enable\":false}','23.158.104.250','[]',1763448626,1763448626),(24,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_verify\":false,\"safe_mode_enable\":false,\"secure_path\":\"2b1145b9\",\"email_whitelist_enable\":false,\"email_whitelist_suffix\":[\"gmail.com\",\"qq.com\",\"163.com\",\"yahoo.com\",\"sina.com\",\"126.com\",\"outlook.com\",\"yeah.net\",\"foxmail.com\"],\"email_gmail_limit_enable\":true,\"captcha_enable\":false,\"captcha_type\":\"recaptcha\",\"recaptcha_key\":null,\"recaptcha_site_key\":null,\"recaptcha_v3_secret_key\":null,\"recaptcha_v3_site_key\":null,\"recaptcha_v3_score_threshold\":\"0.5\",\"turnstile_secret_key\":null,\"turnstile_site_key\":null,\"register_limit_by_ip_enable\":false,\"register_limit_count\":\"3\",\"register_limit_expire\":\"60\",\"password_limit_enable\":true,\"password_limit_count\":\"5\",\"password_limit_expire\":\"60\",\"recaptcha_enable\":false}','23.158.104.250','[]',1763448627,1763448627),(25,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_verify\":false,\"safe_mode_enable\":false,\"secure_path\":\"2b1145b9\",\"email_whitelist_enable\":true,\"email_whitelist_suffix\":[\"gmail.com\",\"qq.com\",\"163.com\",\"yahoo.com\",\"sina.com\",\"126.com\",\"outlook.com\",\"yeah.net\",\"foxmail.com\"],\"email_gmail_limit_enable\":true,\"captcha_enable\":false,\"captcha_type\":\"recaptcha\",\"recaptcha_key\":null,\"recaptcha_site_key\":null,\"recaptcha_v3_secret_key\":null,\"recaptcha_v3_site_key\":null,\"recaptcha_v3_score_threshold\":\"0.5\",\"turnstile_secret_key\":null,\"turnstile_site_key\":null,\"register_limit_by_ip_enable\":false,\"register_limit_count\":\"3\",\"register_limit_expire\":\"60\",\"password_limit_enable\":true,\"password_limit_count\":\"5\",\"password_limit_expire\":\"60\",\"recaptcha_enable\":false}','23.158.104.250','[]',1763448641,1763448641),(26,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_verify\":false,\"safe_mode_enable\":false,\"secure_path\":\"2b1145b9\",\"email_whitelist_enable\":true,\"email_whitelist_suffix\":[\"gmail.com\",\"qq.com\",\"163.com\",\"yahoo.com\",\"sina.com\",\"126.com\",\"outlook.com\",\"yeah.net\",\"foxmail.com\"],\"email_gmail_limit_enable\":true,\"captcha_enable\":true,\"captcha_type\":\"recaptcha\",\"recaptcha_key\":null,\"recaptcha_site_key\":null,\"recaptcha_v3_secret_key\":null,\"recaptcha_v3_site_key\":null,\"recaptcha_v3_score_threshold\":\"0.5\",\"turnstile_secret_key\":null,\"turnstile_site_key\":null,\"register_limit_by_ip_enable\":false,\"register_limit_count\":\"3\",\"register_limit_expire\":\"60\",\"password_limit_enable\":true,\"password_limit_count\":\"5\",\"password_limit_expire\":\"60\",\"recaptcha_enable\":false}','23.158.104.250','[]',1763448644,1763448644),(27,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_verify\":false,\"safe_mode_enable\":false,\"secure_path\":\"2b1145b9\",\"email_whitelist_enable\":true,\"email_whitelist_suffix\":[\"gmail.com\",\"qq.com\",\"163.com\",\"yahoo.com\",\"sina.com\",\"126.com\",\"outlook.com\",\"yeah.net\",\"foxmail.com\"],\"email_gmail_limit_enable\":true,\"captcha_enable\":false,\"captcha_type\":\"recaptcha\",\"recaptcha_key\":null,\"recaptcha_site_key\":null,\"recaptcha_v3_secret_key\":null,\"recaptcha_v3_site_key\":null,\"recaptcha_v3_score_threshold\":\"0.5\",\"turnstile_secret_key\":null,\"turnstile_site_key\":null,\"register_limit_by_ip_enable\":false,\"register_limit_count\":\"3\",\"register_limit_expire\":\"60\",\"password_limit_enable\":true,\"password_limit_count\":\"5\",\"password_limit_expire\":\"60\",\"recaptcha_enable\":false}','23.158.104.250','[]',1763448648,1763448648),(28,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"invite_force\":false,\"invite_commission\":\"10\",\"invite_gen_limit\":\"5\",\"invite_never_expire\":false,\"commission_first_time_enable\":true,\"commission_auto_check_enable\":false,\"commission_withdraw_limit\":\"100\",\"commission_withdraw_method\":[\"\\u652f\\u4ed8\\u5b9d\",\"USDT\",\"Paypal\"],\"withdraw_close_enable\":false,\"commission_distribution_enable\":false,\"commission_distribution_l1\":null,\"commission_distribution_l2\":null,\"commission_distribution_l3\":null}','23.158.104.250','[]',1763448655,1763448655),(29,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"server_pull_interval\":60,\"server_push_interval\":60,\"server_token\":\"qeSlAnUlWd7AXfctBGt6z1CkDQKXaW\",\"device_limit_mode\":0}','23.158.104.250','[]',1763448665,1763448665),(30,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_host\":\"smtp.qq.com\",\"email_port\":null,\"email_encryption\":null,\"email_username\":null,\"email_password\":null,\"email_from_address\":null,\"email_template\":\"default\",\"remind_mail_enable\":false}','23.158.104.250','[]',1763448675,1763448675),(31,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_host\":\"smtp.qq.com\",\"email_port\":465,\"email_encryption\":null,\"email_username\":null,\"email_password\":null,\"email_from_address\":null,\"email_template\":\"default\",\"remind_mail_enable\":false}','23.158.104.250','[]',1763448689,1763448689),(32,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_host\":\"smtp.qq.com\",\"email_port\":465,\"email_encryption\":null,\"email_username\":\"1425350757@qq.com\",\"email_password\":null,\"email_from_address\":null,\"email_template\":\"default\",\"remind_mail_enable\":false}','23.158.104.250','[]',1763448704,1763448704),(33,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_host\":\"smtp.qq.com\",\"email_port\":465,\"email_encryption\":null,\"email_username\":\"1425350757@qq.com\",\"email_password\":\"vkrlizzwjhnrbadd\",\"email_from_address\":null,\"email_template\":\"default\",\"remind_mail_enable\":false}','23.158.104.250','[]',1763448713,1763448713),(34,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_host\":\"smtp.qq.com\",\"email_port\":465,\"email_encryption\":null,\"email_username\":\"1425350757@qq.com\",\"email_password\":\"vkrlizzwjhnrbadd\",\"email_from_address\":\"1425350757@qq.com\",\"email_template\":\"default\",\"remind_mail_enable\":false}','23.158.104.250','[]',1763448717,1763448717),(35,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_host\":\"smtp.qq.com\",\"email_port\":465,\"email_encryption\":null,\"email_username\":\"1425350757@qq.com\",\"email_password\":\"vkrlizzwjhnrbadd\",\"email_from_address\":\"1425350757@qq.com\",\"email_template\":\"classic\",\"remind_mail_enable\":false}','23.158.104.250','[]',1763448719,1763448719),(36,'POST api/v2/2b1145b9/config/testSendMail','INFO','https://588235.xyz','/api/v2/2b1145b9/config/testSendMail','POST','[]','23.158.104.250','[]',1763448721,1763448721),(37,'POST api/v2/2b1145b9/config/save','INFO','https://588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_host\":\"smtp.qq.com\",\"email_port\":465,\"email_encryption\":null,\"email_username\":\"1425350757@qq.com\",\"email_password\":\"vkrlizzwjhnrbadd\",\"email_from_address\":\"1425350757@qq.com\",\"email_template\":\"classic\",\"remind_mail_enable\":true}','23.158.104.250','[]',1763448721,1763448721),(38,'POST api/v2/2b1145b9/order/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448741,1763448741),(39,'POST api/v2/2b1145b9/user/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448746,1763448746),(40,'POST api/v2/2b1145b9/order/assign','INFO','https://588235.xyz','/api/v2/2b1145b9/order/assign','POST','{\"email\":\"1425350757@qq.com\",\"plan_id\":1,\"period\":\"three_year_price\",\"total_amount\":0}','23.158.104.250','[]',1763448761,1763448761),(41,'POST api/v2/2b1145b9/user/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448766,1763448766),(42,'POST api/v2/2b1145b9/user/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448796,1763448796),(43,'POST api/v2/2b1145b9/order/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448801,1763448801),(44,'POST api/v2/2b1145b9/order/paid','INFO','https://588235.xyz','/api/v2/2b1145b9/order/paid','POST','{\"trade_no\":\"c104e4b1cc8c77e8d35e581d91c7413f\"}','23.158.104.250','[]',1763448805,1763448805),(45,'POST api/v2/2b1145b9/order/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448805,1763448805),(46,'POST api/v2/2b1145b9/user/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763448808,1763448808),(47,'POST api/v2/2b1145b9/server/manage/save','INFO','https://588235.xyz','/api/v2/2b1145b9/server/manage/save','POST','{\"id\":null,\"specific_key\":null,\"code\":\"3\",\"show\":false,\"name\":\"\\u65e5\\u672c-Fla-1G\",\"rate\":\"1\",\"rate_time_enable\":false,\"rate_time_ranges\":[],\"tags\":[],\"excludes\":[],\"ips\":[],\"group_ids\":[\"1\",\"2\"],\"host\":\"2400:cca0:100:4fc::\",\"port\":\"33333\",\"server_port\":\"33333\",\"parent_id\":\"0\",\"route_ids\":[],\"protocol_settings\":{\"tls\":2,\"tls_settings\":{\"server_name\":null,\"allow_insecure\":false},\"reality_settings\":{\"server_port\":443,\"server_name\":\"www.amazon.com\",\"allow_insecure\":false,\"public_key\":\"U7i7tNVOiuJr-ndcNnNyvD3p_oRtUWYm3Lf3sIuQKi0\",\"private_key\":\"dXz18u1dfXK2EkWlKSrvk8IfLqk5u09UatZHNAKf4QU\",\"short_id\":\"12e53dce81928f\"},\"network\":\"tcp\",\"network_settings\":[],\"flow\":\"xtls-rprx-vision\"},\"type\":\"vless\"}','23.158.104.250','[]',1763448985,1763448985),(48,'POST api/v2/2b1145b9/server/manage/update','INFO','https://588235.xyz','/api/v2/2b1145b9/server/manage/update','POST','{\"id\":89,\"type\":\"vless\",\"show\":1}','23.158.104.250','[]',1763448987,1763448987),(49,'POST api/v2/2b1145b9/user/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763449001,1763449001),(50,'POST api/v2/2b1145b9/user/fetch','INFO','https://588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.250','[]',1763449011,1763449011),(51,'POST api/v2/2b1145b9/config/save','INFO','https://x.588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"app_name\":\"HeiYu-Air\",\"app_description\":\"\\u81f4\\u656c\\u4f20\\u5947\\u673a\\u957f\\u9ed1\\u7fbd\\uff08\\u5f3a\\u5236\\u9080\\u8bf7\\u5236\\uff0c\\u6ca1\\u6709\\u9080\\u8bf7\\u7801\\u53ef\\u4ee5\\u5148\\u79bb\\u5f00\\u4e86~\\uff09\",\"app_url\":\"https:\\/\\/x.588235.xyz\",\"force_https\":1,\"logo\":null,\"subscribe_url\":null,\"tos_url\":null,\"stop_register\":1,\"try_out_plan_id\":0,\"currency\":\"CNY\",\"currency_symbol\":\"\\u00a5\",\"try_out_hour\":1}','154.85.17.49','[]',1763454415,1763454415),(52,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','23.158.104.248','[]',1763454528,1763454528),(53,'POST api/v2/2b1145b9/server/manage/save','INFO','https://x.588235.xyz','/api/v2/2b1145b9/server/manage/save','POST','{\"id\":89,\"specific_key\":null,\"code\":\"3\",\"show\":true,\"name\":\"\\u65e5\\u672c-Fla-1G\",\"rate\":\"1.00\",\"rate_time_enable\":false,\"rate_time_ranges\":[],\"tags\":[],\"excludes\":[],\"ips\":[],\"group_ids\":[\"1\",\"2\"],\"host\":\"jp6.cloudflare.182682.xyz\",\"port\":\"8443\",\"server_port\":\"8443\",\"parent_id\":0,\"route_ids\":[],\"protocol_settings\":{\"tls\":1,\"tls_settings\":{\"server_name\":\"jp6.588235.xyz\",\"allow_insecure\":false},\"flow\":null,\"network\":\"ws\",\"network_settings\":{\"path\":\"\\/\",\"headers\":{\"Host\":\"jp6.588235.xyz\"}},\"reality_settings\":{\"allow_insecure\":false,\"server_port\":\"443\",\"server_name\":\"www.amazon.com\",\"public_key\":\"U7i7tNVOiuJr-ndcNnNyvD3p_oRtUWYm3Lf3sIuQKi0\",\"private_key\":\"dXz18u1dfXK2EkWlKSrvk8IfLqk5u09UatZHNAKf4QU\",\"short_id\":\"12e53dce81928f\"}},\"type\":\"vless\",\"sort\":null,\"created_at\":1763448985,\"updated_at\":1763448987,\"groups\":[{\"name\":\"\\u521d\\u7ea7\",\"id\":1},{\"name\":\"\\u72ec\\u4eab\",\"id\":2}],\"parent\":null,\"last_check_at\":\"1763454112\",\"last_push_at\":null,\"online\":0,\"is_online\":0,\"available_status\":0,\"cache_key\":\"vless-89-1763448987-0\",\"load_status\":null}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763455093,1763455093),(54,'POST api/v2/2b1145b9/server/manage/drop','INFO','https://x.588235.xyz','/api/v2/2b1145b9/server/manage/drop','POST','{\"id\":86}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763455446,1763455446),(55,'POST api/v2/2b1145b9/server/manage/drop','INFO','https://x.588235.xyz','/api/v2/2b1145b9/server/manage/drop','POST','{\"id\":85}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763455451,1763455451),(56,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456467,1763456467),(57,'POST api/v2/2b1145b9/user/update','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/update','POST','{\"id\":1,\"plan_id\":3,\"device_limit\":null}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456481,1763456481),(58,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456481,1763456481),(59,'POST api/v2/2b1145b9/plan/save','INFO','https://x.588235.xyz','/api/v2/2b1145b9/plan/save','POST','{\"id\":3,\"group_id\":2,\"name\":\"\\u72ec\\u4eab\",\"tags\":[],\"content\":null,\"transfer_enable\":1024,\"prices\":{\"monthly\":null,\"quarterly\":null,\"half_yearly\":null,\"yearly\":null,\"two_yearly\":null,\"three_yearly\":null,\"onetime\":null,\"reset_traffic\":null},\"speed_limit\":null,\"capacity_limit\":null,\"device_limit\":null,\"force_update\":false,\"reset_traffic_method\":null,\"users_count\":1,\"active_users_count\":1}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456530,1763456530),(60,'POST api/v2/2b1145b9/plan/save','INFO','https://x.588235.xyz','/api/v2/2b1145b9/plan/save','POST','{\"id\":3,\"group_id\":2,\"name\":\"\\u72ec\\u4eab\",\"tags\":[],\"content\":null,\"transfer_enable\":1024,\"prices\":{\"monthly\":null,\"quarterly\":null,\"half_yearly\":null,\"yearly\":null,\"two_yearly\":null,\"three_yearly\":null,\"onetime\":null,\"reset_traffic\":null},\"speed_limit\":null,\"capacity_limit\":null,\"device_limit\":null,\"force_update\":true,\"reset_traffic_method\":null,\"users_count\":1,\"active_users_count\":1}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456534,1763456534),(61,'POST api/v2/2b1145b9/config/save','INFO','https://x.588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"app_name\":\"HeiYu-Air\",\"app_description\":\"\\u81f4\\u656c\\u4f20\\u5947\\u673a\\u957f\\u9ed1\\u7fbd\\uff08\\u5f3a\\u5236\\u9080\\u8bf7\\u5236\\uff0c\\u6ca1\\u6709\\u9080\\u8bf7\\u7801\\u53ef\\u4ee5\\u5148\\u79bb\\u5f00\\u4e86~\\uff09\",\"app_url\":\"https:\\/\\/x.588235.xyz\",\"force_https\":1,\"logo\":null,\"subscribe_url\":null,\"tos_url\":null,\"stop_register\":0,\"try_out_plan_id\":0,\"currency\":\"CNY\",\"currency_symbol\":\"\\u00a5\",\"try_out_hour\":1}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456815,1763456815),(62,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456818,1763456818),(63,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456937,1763456937),(64,'POST api/v2/2b1145b9/order/assign','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/assign','POST','{\"email\":\"937866880@qq.com\",\"plan_id\":1,\"period\":\"three_year_price\",\"total_amount\":0}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456958,1763456958),(65,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456963,1763456963),(66,'POST api/v2/2b1145b9/order/paid','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/paid','POST','{\"trade_no\":\"2746945d4c967af5feb983aa97a99efb\"}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456965,1763456965),(67,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456965,1763456965),(68,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456981,1763456981),(69,'POST api/v2/2b1145b9/order/assign','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/assign','POST','{\"email\":\"3194517886@qq.com\",\"plan_id\":1,\"period\":\"three_year_price\",\"total_amount\":0}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456990,1763456990),(70,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456992,1763456992),(71,'POST api/v2/2b1145b9/order/paid','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/paid','POST','{\"trade_no\":\"e0061b54aaa6772c2bf150fb0da51452\"}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456995,1763456995),(72,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763456995,1763456995),(73,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457000,1763457000),(74,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457048,1763457048),(75,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457059,1763457059),(76,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457091,1763457091),(77,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457113,1763457113),(78,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457115,1763457115),(79,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457178,1763457178),(80,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457215,1763457215),(81,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457217,1763457217),(82,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457275,1763457275),(83,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457299,1763457299),(84,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457340,1763457340),(85,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457364,1763457364),(86,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457373,1763457373),(87,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457395,1763457395),(88,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457397,1763457397),(89,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457399,1763457399),(90,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457404,1763457404),(91,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457407,1763457407),(92,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457423,1763457423),(93,'POST api/v2/2b1145b9/config/save','INFO','https://x.588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"email_verify\":false,\"safe_mode_enable\":false,\"secure_path\":\"2b1145b9\",\"email_whitelist_enable\":true,\"email_whitelist_suffix\":[\"gmail.com\",\"qq.com\",\"163.com\",\"foxmail.com\"],\"email_gmail_limit_enable\":true,\"captcha_enable\":false,\"captcha_type\":\"recaptcha\",\"recaptcha_key\":null,\"recaptcha_site_key\":null,\"recaptcha_v3_secret_key\":null,\"recaptcha_v3_site_key\":null,\"recaptcha_v3_score_threshold\":\"0.5\",\"turnstile_secret_key\":null,\"turnstile_site_key\":null,\"register_limit_by_ip_enable\":false,\"register_limit_count\":\"3\",\"register_limit_expire\":\"60\",\"password_limit_enable\":true,\"password_limit_count\":\"5\",\"password_limit_expire\":\"60\",\"recaptcha_enable\":false}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457441,1763457441),(94,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457447,1763457447),(95,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457449,1763457449),(96,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457497,1763457497),(97,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457508,1763457508),(98,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457516,1763457516),(99,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457518,1763457518),(100,'POST api/v2/2b1145b9/order/assign','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/assign','POST','{\"email\":\"yanku8471@gmail.com\",\"plan_id\":1,\"period\":\"two_year_price\",\"total_amount\":0}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457525,1763457525),(101,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457533,1763457533),(102,'POST api/v2/2b1145b9/order/paid','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/paid','POST','{\"trade_no\":\"d637204667c7b4b7bd7f8b533aefdd8d\"}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457535,1763457535),(103,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457536,1763457536),(104,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457546,1763457546),(105,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457581,1763457581),(106,'POST api/v2/2b1145b9/coupon/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/coupon/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457581,1763457581),(107,'POST api/v2/2b1145b9/gift-card/templates','INFO','https://x.588235.xyz','/api/v2/2b1145b9/gift-card/templates','POST','{\"per_page\":20,\"page\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457582,1763457582),(108,'POST api/v2/2b1145b9/coupon/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/coupon/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457583,1763457583),(109,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457583,1763457583),(110,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457587,1763457587),(111,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:c025:9fb7:5c88:f396','[]',1763457610,1763457610),(112,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459393,1763459393),(113,'POST api/v2/2b1145b9/order/assign','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/assign','POST','{\"email\":\"mizi280012@gmail.com\",\"plan_id\":1,\"period\":\"three_year_price\",\"total_amount\":0}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459411,1763459411),(114,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459422,1763459422),(115,'POST api/v2/2b1145b9/order/cancel','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/cancel','POST','{\"trade_no\":\"2025111817110344351231918\"}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459431,1763459431),(116,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459432,1763459432),(117,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459437,1763459437),(118,'POST api/v2/2b1145b9/order/assign','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/assign','POST','{\"email\":\"mizi280012@gmail.com\",\"plan_id\":1,\"period\":\"three_year_price\",\"total_amount\":0}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459444,1763459444),(119,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459454,1763459454),(120,'POST api/v2/2b1145b9/order/paid','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/paid','POST','{\"trade_no\":\"ea3e6da974d56f874d5b26fb12ac00e1\"}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459466,1763459466),(121,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459467,1763459467),(122,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8421:b40:ec0:5883:65ff:fe2c:e80e','[]',1763459471,1763459471),(123,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470245,1763470245),(124,'POST api/v2/2b1145b9/user/generate','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/generate','POST','{\"email_prefix\":\"2766016387\",\"email_suffix\":\"qq.com\",\"password\":null,\"expired_at\":null,\"plan_id\":null,\"download_csv\":false}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470291,1763470291),(125,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470292,1763470292),(126,'POST api/v2/2b1145b9/order/assign','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/assign','POST','{\"email\":\"2766016387@qq.com\",\"plan_id\":1,\"period\":\"three_year_price\",\"total_amount\":0}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470302,1763470302),(127,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470303,1763470303),(128,'POST api/v2/2b1145b9/order/paid','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/paid','POST','{\"trade_no\":\"415c871c069a6f9aabb5253d78157ec0\"}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470306,1763470306),(129,'POST api/v2/2b1145b9/order/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/order/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470306,1763470306),(130,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470309,1763470309),(131,'POST api/v2/2b1145b9/config/save','INFO','https://x.588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"app_name\":\"HeiYu-Air\",\"app_description\":\"\\u81f4\\u656c\\u4f20\\u5947\\u673a\\u957f\\u9ed1\\u7fbd\\uff08\\u5f3a\\u5236\\u9080\\u8bf7\\u5236\\uff0c\\u6ca1\\u6709\\u9080\\u8bf7\\u7801\\u53ef\\u4ee5\\u5148\\u79bb\\u5f00\\u4e86~\\uff09\",\"app_url\":\"https:\\/\\/x.588235.xyz\",\"force_https\":1,\"logo\":null,\"subscribe_url\":null,\"tos_url\":null,\"stop_register\":1,\"try_out_plan_id\":0,\"currency\":\"CNY\",\"currency_symbol\":\"\\u00a5\",\"try_out_hour\":1}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470355,1763470355),(132,'POST api/v2/2b1145b9/config/save','INFO','https://x.588235.xyz','/api/v2/2b1145b9/config/save','POST','{\"invite_force\":true,\"invite_commission\":\"10\",\"invite_gen_limit\":\"5\",\"invite_never_expire\":false,\"commission_first_time_enable\":true,\"commission_auto_check_enable\":false,\"commission_withdraw_limit\":\"100\",\"commission_withdraw_method\":[\"\\u652f\\u4ed8\\u5b9d\",\"USDT\",\"Paypal\"],\"withdraw_close_enable\":false,\"commission_distribution_enable\":false,\"commission_distribution_l1\":null,\"commission_distribution_l2\":null,\"commission_distribution_l3\":null}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470364,1763470364),(133,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','2408:8221:b12:7480:1195:c4e4:903c:3525','[]',1763470380,1763470380),(134,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','42.236.216.175','[]',1763470623,1763470623),(135,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','42.236.216.175','[]',1763470644,1763470644),(136,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','42.236.216.175','[]',1763478338,1763478338),(137,'统计任务执行完毕。耗时:9.692907333374E-5','INFO','http://localhost','/','GET','[]','127.0.0.1','[]',1763482200,1763482200),(138,'POST api/v2/2b1145b9/server/manage/save','INFO','https://x.588235.xyz','/api/v2/2b1145b9/server/manage/save','POST','{\"id\":null,\"specific_key\":null,\"code\":\"5\",\"show\":false,\"name\":\"\\u7f8e\\u56fd-RN\",\"rate\":\"1\",\"rate_time_enable\":false,\"rate_time_ranges\":[],\"tags\":[],\"excludes\":[],\"ips\":[],\"group_ids\":[\"1\",\"2\"],\"host\":\"74.48.43.241\",\"port\":\"33333\",\"server_port\":\"33333\",\"parent_id\":\"0\",\"route_ids\":[],\"protocol_settings\":{\"tls\":2,\"tls_settings\":{\"server_name\":null,\"allow_insecure\":false},\"reality_settings\":{\"server_port\":443,\"server_name\":\"www.amazon.com\",\"allow_insecure\":false,\"public_key\":\"sFKkW7ODuyM2TrxmstyY-ePhV0xIZHc_wbRaSvOj-js\",\"private_key\":\"So7CcXgh5i0ix_mm5OE4_yh8wrwiuMUO6QT7hN9duYI\",\"short_id\":\"0e828d9eb1ea26\"},\"network\":\"tcp\",\"network_settings\":[],\"flow\":\"xtls-rprx-vision\"},\"type\":\"vless\"}','42.236.216.175','[]',1763516228,1763516228),(139,'POST api/v2/2b1145b9/server/manage/update','INFO','https://x.588235.xyz','/api/v2/2b1145b9/server/manage/update','POST','{\"id\":90,\"type\":\"vless\",\"show\":1}','42.236.216.175','[]',1763516230,1763516230),(140,'POST api/v2/2b1145b9/user/fetch','INFO','https://x.588235.xyz','/api/v2/2b1145b9/user/fetch','POST','{\"pageSize\":20,\"current\":1,\"filter\":[],\"sort\":[]}','42.236.216.175','[]',1763516889,1763516889),(141,'Theme initialized in public directory','INFO','http://74.48.43.241:7001','/','GET','[]','198.176.52.221','{\"theme\":\"Xboard\"}',1763517261,1763517261);
/*!40000 ALTER TABLE `v2_log` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_mail_log`
--

DROP TABLE IF EXISTS `v2_mail_log`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_mail_log` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`email` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
`subject` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`template_name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`error` text COLLATE utf8mb4_unicode_ci,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_mail_log`
--

LOCK TABLES `v2_mail_log` WRITE;
/*!40000 ALTER TABLE `v2_mail_log` DISABLE KEYS */;
INSERT INTO `v2_mail_log` VALUES (1,'1425350757@qq.com','This is xboard test email','mail.classic.notify',NULL,1763448722,1763448722);
/*!40000 ALTER TABLE `v2_mail_log` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_notice`
--

DROP TABLE IF EXISTS `v2_notice`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_notice` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`sort` int(11) DEFAULT NULL,
`title` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`content` text COLLATE utf8mb4_unicode_ci NOT NULL,
`show` tinyint(1) NOT NULL DEFAULT '0',
`img_url` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`tags` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
KEY `v2_notice_sort_index` (`sort`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_notice`
--

LOCK TABLES `v2_notice` WRITE;
/*!40000 ALTER TABLE `v2_notice` DISABLE KEYS */;
INSERT INTO `v2_notice` VALUES (1,NULL,'FQ必读，防范未然，避免喝茶','## 禁忌 \r\n1.不要在任何外网平台谈论政治相关话题，这是大忌，除非你不在大陆混了；\r\n\r\n2.不要在X（推特）上关注反贼（比如李老师），更不要互动，有人因为这个被请喝茶，X可以看到关注的人；\r\n\r\n3.youtube（油管）可以关注反贼，因为其他人无法查看关注列表，但不要评论点赞视频；\r\n\r\n4.不要安装反诈APP，他不能帮你反诈，但能请你喝茶；\r\n\r\n5.不要在国内的社交平台谈论翻墙，机场，境外的APP等相关话题。特别是QQ，微信群，因为国内平台受到监管影响，会有监控，触发关键词会上报网警，之后会有当地派出所找到你；\r\n\r\n## 尽量避免 \r\n1.尽量避免使用国产输入法，输入法是很多人容易忽略的；\r\n\r\n2.避免DNS泄露，特别是clash用户。如果你不懂，请使用教程中推荐的客户端；\r\n\r\n3.如果你想在“世界遨游”，请尽量避免购买华为手机；\r\n\r\n4.如果你是+86注册的Telegram（TG），请在隐私中隐藏手机号；\r\n\r\n5.尽量避免使用国内邮箱在境外平台注册账号，最好使用Gmail或outlook等海外邮箱；\r\n\r\n## 风险\r\n1.翻墙的人数有很多，一般情况（比如色色，AI，论坛，聊天，学术等）都不会请你喝茶，所以不用太担心；\r\n\r\n2.如果你在学校，请注意有些学校内置的防火墙，能非常轻易找到你，尤其避免DNS泄露问题；\r\n\r\n3.即使知道你在翻墙，大多数情况也没什么事，除非当地公安需要你冲业绩了；\r\n\r\n\r\n## 被请喝茶怎么办\r\n \r\n1.保持冷静，想想你做了什么，叔叔一般都是掌握了你的一些证据才会叫你，叔叔指认的老实承认，但请不要多说；\r\n\r\n2.如果问你为什么翻墙，你可以说因为工作、学术等需要用到AI，MJ，谷歌学术之类的，或者饥渴了，想看色色；\r\n\r\n3.如果问你知道为什么叫你来吗？切忌不要上来全盘托出，装不知道，把问题交给他，有证据的指控就承认，无实质证据的均否认；\r\n\r\n4.如果问翻墙工具怎么来的，不要说朋友推荐或者合租之类的，你就说搜索如何用GPT，看到别人分享的就拿来用了；',0,NULL,'[]',1752307149,1752307947),(2,NULL,'手册&用户须知toS','## 使用手册&注解\r\n除 特殊标注 外，节点速率基本为1Gbps\r\n形如 \"x2.0\" 为倍率，即按照两倍的倍率流量进行计算\r\n【0.01】为超低倍率，更专用于下载服务（但是不要用来长时间占用宽带和BT这类违反DMCA的服务）\r\n\r\n## 节点&倍率问题\r\n本站采用 “动态倍率”。\r\n即随时可能变化倍率（节点名末尾后有倍率标注）\r\n哪怕两个相似的节点可能倍率也不相同（因质量&上游不同）\r\n\r\n## ToS内容\r\nHeiYu-Air（下简称Air）\r\n1. 无退款计划（除HeiYu主动退款）\r\n2. 请勿随意转发订阅\r\n3. 请勿滥用节点IP\r\n4. 禁止BT/PT，以及其他违反DMCA等\r\n5. 禁止违法犯罪等\r\n6. 违反规则可能被导致账号/订阅被（包括但不限于）暂停/封禁/永久锁定\r\n7. 本ToS可在不告知用户的前提下进行更新\r\n8. 邀请码不得随意发布\r\n\r\n附：**HeiYu保留对本站(HeiYu-Air)的一切权力（即在需要的情况下，不需要任何理由暂停任意账号/订阅，且无退款计划）**\r\n\r\n注意事项：**此ToS已发布于 Polygon 链（版本更新也会同步推送新版）**\r\n\r\n如果不接受以上条款请自寻出路。\r\n购买订阅视为自动接受条款\r\n\r\nAir运营团队 - （HeiYu）',1,NULL,'[]',1752307924,1752307924);
/*!40000 ALTER TABLE `v2_notice` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_order`
--

DROP TABLE IF EXISTS `v2_order`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_order` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`invite_user_id` int(11) DEFAULT NULL,
`user_id` int(11) NOT NULL,
`plan_id` int(11) NOT NULL,
`coupon_id` int(11) DEFAULT NULL,
`payment_id` int(11) DEFAULT NULL,
`type` int(11) NOT NULL COMMENT '1新购2续费3升级',
`period` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`trade_no` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL,
`callback_no` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`total_amount` int(11) NOT NULL,
`handling_amount` int(11) DEFAULT NULL,
`discount_amount` int(11) DEFAULT NULL,
`surplus_amount` int(11) DEFAULT NULL COMMENT '剩余价值',
`refund_amount` int(11) DEFAULT NULL COMMENT '退款金额',
`balance_amount` int(11) DEFAULT NULL COMMENT '使用余额',
`surplus_order_ids` text COLLATE utf8mb4_unicode_ci COMMENT '折抵订单',
`status` int(11) NOT NULL DEFAULT '0' COMMENT '0待支付1开通中2已取消3已完成4已折抵',
`commission_status` int(11) NOT NULL DEFAULT '0' COMMENT '0待确认1发放中2有效3无效',
`commission_balance` int(11) NOT NULL DEFAULT '0',
`actual_commission_balance` int(11) DEFAULT NULL COMMENT '实际支付佣金',
`paid_at` int(11) DEFAULT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `trade_no` (`trade_no`),
KEY `v2_order_created_at_index` (`created_at`),
KEY `v2_order_status_index` (`status`),
KEY `v2_order_total_amount_index` (`total_amount`),
KEY `v2_order_commission_status_index` (`commission_status`),
KEY `v2_order_invite_user_id_index` (`invite_user_id`),
KEY `v2_order_commission_balance_index` (`commission_balance`),
KEY `v2_order_updated_at_index` (`updated_at`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_order`
--

LOCK TABLES `v2_order` WRITE;
/*!40000 ALTER TABLE `v2_order` DISABLE KEYS */;
INSERT INTO `v2_order` VALUES (1,NULL,1,1,NULL,NULL,1,'three_yearly','c104e4b1cc8c77e8d35e581d91c7413f','manual_operation',0,NULL,NULL,NULL,NULL,NULL,NULL,3,0,0,NULL,1763448805,1763448761,1763448805),(2,NULL,2,1,NULL,NULL,1,'three_yearly','2746945d4c967af5feb983aa97a99efb','manual_operation',0,NULL,NULL,NULL,NULL,NULL,NULL,3,0,0,NULL,1763456965,1763456959,1763456965),(3,NULL,3,1,NULL,NULL,1,'three_yearly','e0061b54aaa6772c2bf150fb0da51452','manual_operation',0,NULL,NULL,NULL,NULL,NULL,NULL,3,0,0,NULL,1763456995,1763456990,1763456995),(4,NULL,4,1,NULL,NULL,1,'two_yearly','d637204667c7b4b7bd7f8b533aefdd8d','manual_operation',0,NULL,NULL,NULL,NULL,NULL,NULL,3,0,0,NULL,1763457535,1763457525,1763457535),(5,NULL,5,1,NULL,NULL,1,'monthly','2025111817114095991559575',NULL,1,NULL,NULL,NULL,NULL,NULL,NULL,2,0,0,NULL,NULL,1763459260,1763459277),(6,NULL,5,1,NULL,NULL,1,'monthly','2025111817110344351231918',NULL,1,NULL,NULL,NULL,NULL,NULL,NULL,2,0,0,NULL,NULL,1763459283,1763459431),(7,NULL,5,1,NULL,NULL,1,'three_yearly','ea3e6da974d56f874d5b26fb12ac00e1','manual_operation',0,NULL,NULL,NULL,NULL,NULL,NULL,3,0,0,NULL,1763459466,1763459444,1763459466),(8,NULL,6,1,NULL,NULL,1,'three_yearly','415c871c069a6f9aabb5253d78157ec0','manual_operation',0,NULL,NULL,NULL,NULL,NULL,NULL,3,0,0,NULL,1763470306,1763470302,1763470306);
/*!40000 ALTER TABLE `v2_order` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_payment`
--

DROP TABLE IF EXISTS `v2_payment`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_payment` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`uuid` char(32) COLLATE utf8mb4_unicode_ci NOT NULL,
`payment` varchar(16) COLLATE utf8mb4_unicode_ci NOT NULL,
`name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`icon` text COLLATE utf8mb4_unicode_ci,
`config` text COLLATE utf8mb4_unicode_ci NOT NULL,
`notify_domain` varchar(128) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`handling_fee_fixed` int(11) DEFAULT NULL,
`handling_fee_percent` decimal(5,2) DEFAULT NULL,
`enable` tinyint(1) NOT NULL DEFAULT '0',
`sort` int(11) DEFAULT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_payment`
--

LOCK TABLES `v2_payment` WRITE;
/*!40000 ALTER TABLE `v2_payment` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_payment` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_plan`
--

DROP TABLE IF EXISTS `v2_plan`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_plan` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`group_id` int(10) unsigned DEFAULT NULL,
`transfer_enable` bigint(20) unsigned DEFAULT NULL COMMENT 'Transfer limit in bytes',
`name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`prices` json DEFAULT NULL COMMENT 'Store different duration prices and reset traffic price',
`sell` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'is sell',
`speed_limit` int(10) unsigned DEFAULT NULL COMMENT 'Speed limit in Mbps, 0 for unlimited',
`device_limit` int(10) unsigned DEFAULT NULL,
`show` tinyint(1) NOT NULL DEFAULT '0',
`sort` int(11) DEFAULT NULL,
`renew` tinyint(1) NOT NULL DEFAULT '1',
`content` text COLLATE utf8mb4_unicode_ci,
`tags` json DEFAULT NULL,
`reset_traffic_method` int(11) DEFAULT '0' COMMENT '重置流量方式:null跟随系统设置、0每月1号、1按月重置、2不重置、3每年1月1日、4按年重置',
`capacity_limit` int(10) unsigned DEFAULT '0' COMMENT '0 for unlimited',
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_plan`
--

LOCK TABLES `v2_plan` WRITE;
/*!40000 ALTER TABLE `v2_plan` DISABLE KEYS */;
INSERT INTO `v2_plan` VALUES (1,1,1024,'虚空套餐','{\"yearly\": null, \"monthly\": \"0.01\", \"onetime\": null, \"quarterly\": null, \"two_yearly\": null, \"half_yearly\": null, \"three_yearly\": null, \"reset_traffic\": null}',1,0,3,1,NULL,1,'- 流量：{{transfer}} GB\r\n- 速度限制：{{speed}} Mbps\r\n- 同时在线设备：{{devices}} 台','[]',NULL,NULL,1752300648,1763131154),(3,2,1024,'独享','{\"yearly\": null, \"monthly\": null, \"onetime\": null, \"quarterly\": null, \"two_yearly\": null, \"half_yearly\": null, \"three_yearly\": null, \"reset_traffic\": null}',0,NULL,NULL,0,NULL,1,NULL,'[]',NULL,NULL,1762784437,1763456534);
/*!40000 ALTER TABLE `v2_plan` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_plugins`
--

DROP TABLE IF EXISTS `v2_plugins`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_plugins` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`code` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`type` varchar(20) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT 'feature' COMMENT '插件类型：feature功能性，payment支付型',
`version` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
`is_enabled` tinyint(1) NOT NULL DEFAULT '0',
`config` json DEFAULT NULL,
`installed_at` timestamp NULL DEFAULT NULL,
`created_at` timestamp NULL DEFAULT NULL,
`updated_at` timestamp NULL DEFAULT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `v2_plugins_code_unique` (`code`),
KEY `v2_plugins_type_is_enabled_index` (`type`,`is_enabled`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_plugins`
--

LOCK TABLES `v2_plugins` WRITE;
/*!40000 ALTER TABLE `v2_plugins` DISABLE KEYS */;
INSERT INTO `v2_plugins` VALUES (1,'EPay','epay','payment','1.0.0',1,'[]','2025-11-18 06:34:08','2025-11-18 06:34:08','2025-11-18 06:34:08'),(2,'AlipayF2F','alipay_f2f','payment','1.0.0',1,'[]','2025-11-18 06:34:08','2025-11-18 06:34:08','2025-11-18 06:34:08'),(3,'BTCPay','btcpay','payment','1.0.0',1,'[]','2025-11-18 06:34:08','2025-11-18 06:34:08','2025-11-18 06:34:08'),(4,'Coinbase','coinbase','payment','1.0.0',1,'[]','2025-11-18 06:34:08','2025-11-18 06:34:08','2025-11-18 06:34:08'),(5,'CoinPayments','coin_payments','payment','1.0.0',1,'[]','2025-11-18 06:34:08','2025-11-18 06:34:08','2025-11-18 06:34:08'),(6,'MGate','mgate','payment','1.0.0',1,'[]','2025-11-18 06:34:08','2025-11-18 06:34:08','2025-11-18 06:34:08'),(7,'Telegram Bot 集成','telegram','feature','1.0.1',1,'{\"help_text\": \"请使用以下命令：\\\\n/bind - 绑定账号\\\\n/traffic - 查看流量\\\\n/getlatesturl - 获取最新链接\", \"start_footer\": \"💡 提示：所有命令都需要在私聊中使用\", \"start_bind_guide\": \"🔗 请先绑定您的 XBoard 账号：\\\\n1. 登录您的 XBoard 账户\\\\n2. 复制您的订阅链接\\\\n3. 发送 /bind + 订阅链接\", \"start_unbind_guide\": \"📋 可用命令：\\\\n/traffic - 查看流量使用情况\\\\n/getlatesturl - 获取订阅链接\\\\n/unbind - 解绑账号\", \"start_bind_commands\": \"📋 可用命令：\\\\n/bind [订阅链接] - 绑定账号\", \"start_welcome_title\": \"🎉 欢迎使用 XBoard Telegram Bot！\", \"enable_ticket_notify\": true, \"enable_payment_notify\": true, \"start_bot_description\": \"🤖 我是您的专属助手，可以帮助您：\\\\n• 绑定您的 XBoard 账号\\\\n• 查看流量使用情况\\\\n• 获取最新订阅链接\\\\n• 管理账号绑定状态\"}','2025-11-18 06:34:08','2025-11-18 06:34:08','2025-11-18 06:34:08');
/*!40000 ALTER TABLE `v2_plugins` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_server`
--

DROP TABLE IF EXISTS `v2_server`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_server` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`type` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT 'Server Type',
`code` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT 'Server Spectific Key',
`parent_id` int(10) unsigned DEFAULT NULL COMMENT 'Parent Server ID',
`group_ids` json DEFAULT NULL COMMENT 'Group ID',
`route_ids` json DEFAULT NULL COMMENT 'Route ID',
`name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT 'Server Name',
`rate` decimal(8,2) NOT NULL COMMENT 'Traffic Rate',
`rate_time_enable` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否启用动态倍率',
`rate_time_ranges` json DEFAULT NULL COMMENT '动态倍率规则',
`tags` json DEFAULT NULL COMMENT 'Server Tags',
`host` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT 'Server Host',
`port` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT 'Client Port',
`server_port` int(11) NOT NULL COMMENT 'Server Port',
`protocol_settings` json DEFAULT NULL,
`show` tinyint(1) NOT NULL DEFAULT '0' COMMENT 'Show in List',
`sort` int(10) unsigned DEFAULT NULL,
`created_at` timestamp NULL DEFAULT NULL,
`updated_at` timestamp NULL DEFAULT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `v2_server_type_code_unique` (`type`,`code`),
KEY `v2_server_sort_index` (`sort`)
) ENGINE=InnoDB AUTO_INCREMENT=91 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_server`
--

LOCK TABLES `v2_server` WRITE;
/*!40000 ALTER TABLE `v2_server` DISABLE KEYS */;
INSERT INTO `v2_server` VALUES (33,'vless','4',0,'[\"1\", \"2\"]','[]','美国-CCS-1G',1.00,0,NULL,'[]','ccs.cloudflare.182682.xyz','8443',8443,'{\"tls\": 1, \"flow\": null, \"network\": \"ws\", \"tls_settings\": {\"server_name\": \"ccs.588235.xyz\", \"allow_insecure\": false}, \"network_settings\": {\"path\": \"/\", \"headers\": {\"Host\": \"ccs.588235.xyz\"}}, \"reality_settings\": {\"short_id\": \"171e5288d3\", \"public_key\": \"lEe1xzMRFfYcppEMbzJG2o7N0Qwcy-IkfS7EVHQPxxc\", \"private_key\": \"d0LIpbY9iOaizWQxrS5w03SMao9P5QRQaEYat0-mUmw\", \"server_name\": \"hydownload.pan.wo.cn\", \"server_port\": \"443\", \"allow_insecure\": false}}',1,3,'2025-10-02 02:32:45','2025-11-17 16:32:48'),(67,'vless','7',0,'[\"1\", \"2\"]','[]','香港-ST-1G',1.00,0,NULL,'[]','hk.cloudflare.182682.xyz','8443',8443,'{\"tls\": 1, \"flow\": null, \"network\": \"ws\", \"tls_settings\": {\"server_name\": \"hk.588235.xyz\", \"allow_insecure\": false}, \"network_settings\": {\"path\": \"/\", \"headers\": {\"Host\": \"hk.588235.xyz\"}}, \"reality_settings\": {\"short_id\": null, \"public_key\": null, \"private_key\": null, \"server_name\": null, \"server_port\": \"443\", \"allow_insecure\": false}}',1,NULL,'2025-10-28 12:26:03','2025-11-17 16:33:09'),(70,'vless','1',0,'[\"2\"]','[]','香港-HW-30M',1.00,0,NULL,'[]','27.106.122.68','33333',33333,'{\"tls\": 2, \"flow\": \"xtls-rprx-vision\", \"network\": \"tcp\", \"tls_settings\": {\"server_name\": \"www.amazon.com\", \"allow_insecure\": false}, \"network_settings\": {\"header\": {\"type\": \"none\"}, \"acceptProxyProtocol\": false}, \"reality_settings\": {\"short_id\": \"94bdd6f661\", \"public_key\": \"aegkIQGR2cCy9ikHuPYYRx5CncIdQqnKRfJoKzuA4jM\", \"private_key\": \"T3ga7x-mEezy-IVXBFecH5IUi3837Qs33BVR0LAEB3Y\", \"server_name\": \"www.amazon.com\", \"server_port\": \"443\", \"allow_insecure\": false}}',1,NULL,'2025-11-06 00:48:45','2025-11-16 13:30:57'),(80,'hysteria','8',0,'[\"1\", \"2\"]','[]','美国-CCS-HY2',1.00,0,NULL,'[]','172.245.45.145','33333',33333,'{\"tls\": {\"server_name\": \"ccs.588235.xyz\", \"allow_insecure\": true}, \"obfs\": {\"open\": false, \"type\": \"salamander\", \"password\": null}, \"version\": 2, \"bandwidth\": {\"up\": null, \"down\": null}, \"hop_interval\": null}',1,NULL,'2025-11-13 10:58:59','2025-11-16 03:06:30'),(83,'vless','8',0,'[\"1\", \"2\"]','[]','美国-HJM-1G',1.00,0,NULL,'[]','hjm.cloudflare.182682.xyz','8443',11951,'{\"tls\": 1, \"flow\": null, \"network\": \"ws\", \"tls_settings\": {\"server_name\": \"us.588235.xyz\", \"allow_insecure\": false}, \"network_settings\": {\"path\": \"/\", \"headers\": {\"Host\": \"us.588235.xyz\"}}, \"reality_settings\": {\"short_id\": null, \"public_key\": null, \"private_key\": null, \"server_name\": null, \"server_port\": \"443\", \"allow_insecure\": false}}',1,NULL,'2025-11-14 09:54:35','2025-11-17 16:32:27'),(84,'vless','9',0,'[\"1\", \"2\"]','[]','新加坡-56IDC-1G',1.00,0,NULL,'[]','185.223.207.16','49700',33333,'{\"tls\": 2, \"flow\": \"xtls-rprx-vision\", \"network\": \"tcp\", \"tls_settings\": {\"server_name\": null, \"allow_insecure\": false}, \"network_settings\": [], \"reality_settings\": {\"short_id\": \"4734ba936674f5\", \"public_key\": \"sz5X30WJv35Ox_2OTvt8jvB5rGuK9b2Lqup9vJ75v0E\", \"private_key\": \"G0dC1ddGvfL0xiOgSBhvrc0tRVBK9WEgEq4rN2JEeUE\", \"server_name\": \"www.amazon.com\", \"server_port\": \"443\", \"allow_insecure\": false}}',1,NULL,'2025-11-14 13:18:46','2025-11-14 13:21:34'),(88,'vless','2',0,'[\"1\", \"2\"]','[]','香港-Fla-1G',1.00,0,NULL,'[]','hk6.cloudflare.182682.xyz','8443',8443,'{\"tls\": 1, \"flow\": null, \"network\": \"ws\", \"tls_settings\": {\"server_name\": \"hk6.588235.xyz\", \"allow_insecure\": false}, \"network_settings\": {\"path\": \"/\", \"headers\": {\"Host\": \"hk6.588235.xyz\"}}, \"reality_settings\": {\"short_id\": \"4718\", \"public_key\": \"cjoDLjz1mKOtkcsZbw1fKbmXI0BIIYSYM4QGnBxjjyA\", \"private_key\": \"rUDEBxyIwvVl_S1l9og_xhis1dw7nPrSNln29p0gVyQ\", \"server_name\": \"www.amazon.com\", \"server_port\": \"443\", \"allow_insecure\": false}}',1,NULL,'2025-11-17 13:34:41','2025-11-17 16:31:40'),(89,'vless','3',0,'[\"1\", \"2\"]','[]','日本-Fla-1G',1.00,0,'[]','[]','jp6.cloudflare.182682.xyz','8443',8443,'{\"tls\": 1, \"flow\": null, \"network\": \"ws\", \"tls_settings\": {\"server_name\": \"jp6.588235.xyz\", \"allow_insecure\": false}, \"network_settings\": {\"path\": \"/\", \"headers\": {\"Host\": \"jp6.588235.xyz\"}}, \"reality_settings\": {\"short_id\": \"12e53dce81928f\", \"public_key\": \"U7i7tNVOiuJr-ndcNnNyvD3p_oRtUWYm3Lf3sIuQKi0\", \"private_key\": \"dXz18u1dfXK2EkWlKSrvk8IfLqk5u09UatZHNAKf4QU\", \"server_name\": \"www.amazon.com\", \"server_port\": \"443\", \"allow_insecure\": false}}',1,NULL,'2025-11-18 06:56:25','2025-11-18 08:38:13'),(90,'vless','5',0,'[\"1\", \"2\"]','[]','美国-RN',1.00,0,'[]','[]','74.48.43.241','33333',33333,'{\"tls\": 2, \"flow\": \"xtls-rprx-vision\", \"network\": \"tcp\", \"tls_settings\": {\"server_name\": null, \"allow_insecure\": false}, \"network_settings\": [], \"reality_settings\": {\"short_id\": \"0e828d9eb1ea26\", \"public_key\": \"sFKkW7ODuyM2TrxmstyY-ePhV0xIZHc_wbRaSvOj-js\", \"private_key\": \"So7CcXgh5i0ix_mm5OE4_yh8wrwiuMUO6QT7hN9duYI\", \"server_name\": \"www.amazon.com\", \"server_port\": \"443\", \"allow_insecure\": false}}',1,NULL,'2025-11-19 01:37:08','2025-11-19 01:37:10');
/*!40000 ALTER TABLE `v2_server` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_server_group`
--

DROP TABLE IF EXISTS `v2_server_group`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_server_group` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_server_group`
--

LOCK TABLES `v2_server_group` WRITE;
/*!40000 ALTER TABLE `v2_server_group` DISABLE KEYS */;
INSERT INTO `v2_server_group` VALUES (1,'初级',1752300525,1752300525),(2,'独享',1762784365,1762784365);
/*!40000 ALTER TABLE `v2_server_group` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_server_route`
--

DROP TABLE IF EXISTS `v2_server_route`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_server_route` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`remarks` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`match` text COLLATE utf8mb4_unicode_ci NOT NULL,
`action` varchar(11) COLLATE utf8mb4_unicode_ci NOT NULL,
`action_value` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_server_route`
--

LOCK TABLES `v2_server_route` WRITE;
/*!40000 ALTER TABLE `v2_server_route` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_server_route` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_settings`
--

DROP TABLE IF EXISTS `v2_settings`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_settings` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`group` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '设置分组',
`type` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL COMMENT '设置类型',
`name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '设置名称',
`value` mediumtext COLLATE utf8mb4_unicode_ci,
`created_at` timestamp NULL DEFAULT NULL,
`updated_at` timestamp NULL DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_setting_name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=61 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_settings`
--

LOCK TABLES `v2_settings` WRITE;
/*!40000 ALTER TABLE `v2_settings` DISABLE KEYS */;
INSERT INTO `v2_settings` VALUES (1,NULL,NULL,'theme_xboard','{\"theme_color\":\"default\",\"background_url\":\"\",\"custom_html\":\"\"}','2025-11-18 06:35:40','2025-11-18 06:35:40'),(2,NULL,NULL,'logo',NULL,'2025-11-18 06:36:18','2025-11-18 06:36:18'),(3,NULL,NULL,'force_https','1','2025-11-18 06:36:18','2025-11-18 06:50:13'),(4,NULL,NULL,'stop_register','1','2025-11-18 06:36:18','2025-11-18 12:52:35'),(5,NULL,NULL,'app_name','HeiYu-Air','2025-11-18 06:36:18','2025-11-18 06:49:54'),(6,NULL,NULL,'app_description','致敬传奇机长黑羽（强制邀请制，没有邀请码可以先离开了~）','2025-11-18 06:36:18','2025-11-18 06:50:04'),(7,NULL,NULL,'app_url','https://x.588235.xyz','2025-11-18 06:36:18','2025-11-18 08:26:55'),(8,NULL,NULL,'subscribe_url',NULL,'2025-11-18 06:36:18','2025-11-18 06:36:18'),(9,NULL,NULL,'try_out_plan_id','0','2025-11-18 06:36:18','2025-11-18 06:36:18'),(10,NULL,NULL,'try_out_hour','1','2025-11-18 06:36:18','2025-11-18 06:36:18'),(11,NULL,NULL,'tos_url',NULL,'2025-11-18 06:36:18','2025-11-18 06:36:18'),(12,NULL,NULL,'currency','CNY','2025-11-18 06:36:18','2025-11-18 06:36:18'),(13,NULL,NULL,'currency_symbol','¥','2025-11-18 06:36:18','2025-11-18 06:36:18'),(14,NULL,NULL,'email_whitelist_enable','1','2025-11-18 06:50:22','2025-11-18 06:50:41'),(15,NULL,NULL,'email_whitelist_suffix','[\"gmail.com\",\"qq.com\",\"163.com\",\"foxmail.com\"]','2025-11-18 06:50:22','2025-11-18 09:17:21'),(16,NULL,NULL,'email_gmail_limit_enable','1','2025-11-18 06:50:22','2025-11-18 06:50:27'),(17,NULL,NULL,'captcha_enable','0','2025-11-18 06:50:22','2025-11-18 09:17:21'),(18,NULL,NULL,'captcha_type','recaptcha','2025-11-18 06:50:22','2025-11-18 06:50:22'),(19,NULL,NULL,'recaptcha_enable','0','2025-11-18 06:50:22','2025-11-18 09:17:21'),(20,NULL,NULL,'recaptcha_key',NULL,'2025-11-18 06:50:22','2025-11-18 06:50:22'),(21,NULL,NULL,'recaptcha_site_key',NULL,'2025-11-18 06:50:22','2025-11-18 06:50:22'),(22,NULL,NULL,'recaptcha_v3_secret_key',NULL,'2025-11-18 06:50:22','2025-11-18 06:50:22'),(23,NULL,NULL,'recaptcha_v3_site_key',NULL,'2025-11-18 06:50:22','2025-11-18 06:50:22'),(24,NULL,NULL,'recaptcha_v3_score_threshold','0.5','2025-11-18 06:50:22','2025-11-18 06:50:22'),(25,NULL,NULL,'turnstile_secret_key',NULL,'2025-11-18 06:50:22','2025-11-18 06:50:22'),(26,NULL,NULL,'turnstile_site_key',NULL,'2025-11-18 06:50:23','2025-11-18 06:50:23'),(27,NULL,NULL,'email_verify','0','2025-11-18 06:50:23','2025-11-18 09:17:21'),(28,NULL,NULL,'safe_mode_enable','0','2025-11-18 06:50:23','2025-11-18 09:17:21'),(29,NULL,NULL,'register_limit_by_ip_enable','0','2025-11-18 06:50:23','2025-11-18 09:17:21'),(30,NULL,NULL,'register_limit_count','3','2025-11-18 06:50:23','2025-11-18 06:50:23'),(31,NULL,NULL,'register_limit_expire','60','2025-11-18 06:50:23','2025-11-18 06:50:23'),(32,NULL,NULL,'secure_path','2b1145b9','2025-11-18 06:50:23','2025-11-18 06:50:23'),(33,NULL,NULL,'password_limit_enable','1','2025-11-18 06:50:23','2025-11-18 06:50:23'),(34,NULL,NULL,'password_limit_count','5','2025-11-18 06:50:23','2025-11-18 06:50:23'),(35,NULL,NULL,'password_limit_expire','60','2025-11-18 06:50:23','2025-11-18 06:50:23'),(36,NULL,NULL,'invite_force','1','2025-11-18 06:50:55','2025-11-18 12:52:44'),(37,NULL,NULL,'invite_commission','10','2025-11-18 06:50:55','2025-11-18 06:50:55'),(38,NULL,NULL,'invite_gen_limit','5','2025-11-18 06:50:55','2025-11-18 06:50:55'),(39,NULL,NULL,'invite_never_expire','0','2025-11-18 06:50:56','2025-11-18 12:52:44'),(40,NULL,NULL,'commission_first_time_enable','1','2025-11-18 06:50:56','2025-11-18 06:50:56'),(41,NULL,NULL,'commission_auto_check_enable','0','2025-11-18 06:50:56','2025-11-18 12:52:44'),(42,NULL,NULL,'commission_withdraw_limit','100','2025-11-18 06:50:56','2025-11-18 06:50:56'),(43,NULL,NULL,'commission_withdraw_method','[\"\\u652f\\u4ed8\\u5b9d\",\"USDT\",\"Paypal\"]','2025-11-18 06:50:56','2025-11-18 06:50:56'),(44,NULL,NULL,'withdraw_close_enable','0','2025-11-18 06:50:56','2025-11-18 12:52:44'),(45,NULL,NULL,'commission_distribution_enable','0','2025-11-18 06:50:56','2025-11-18 12:52:44'),(46,NULL,NULL,'commission_distribution_l1',NULL,'2025-11-18 06:50:56','2025-11-18 06:50:56'),(47,NULL,NULL,'commission_distribution_l2',NULL,'2025-11-18 06:50:56','2025-11-18 06:50:56'),(48,NULL,NULL,'commission_distribution_l3',NULL,'2025-11-18 06:50:56','2025-11-18 06:50:56'),(49,NULL,NULL,'server_token','qeSlAnUlWd7AXfctBGt6z1CkDQKXaW','2025-11-18 06:51:05','2025-11-18 06:51:05'),(50,NULL,NULL,'server_pull_interval','60','2025-11-18 06:51:05','2025-11-18 06:51:05'),(51,NULL,NULL,'server_push_interval','60','2025-11-18 06:51:05','2025-11-18 06:51:05'),(52,NULL,NULL,'device_limit_mode','0','2025-11-18 06:51:05','2025-11-18 06:51:05'),(53,NULL,NULL,'email_template','classic','2025-11-18 06:51:15','2025-11-18 06:51:59'),(54,NULL,NULL,'email_host','smtp.qq.com','2025-11-18 06:51:15','2025-11-18 06:51:15'),(55,NULL,NULL,'email_port','465','2025-11-18 06:51:15','2025-11-18 06:51:29'),(56,NULL,NULL,'email_username','1425350757@qq.com','2025-11-18 06:51:15','2025-11-18 06:51:44'),(57,NULL,NULL,'email_password','vkrlizzwjhnrbadd','2025-11-18 06:51:15','2025-11-18 06:51:53'),(58,NULL,NULL,'email_encryption',NULL,'2025-11-18 06:51:15','2025-11-18 06:51:15'),(59,NULL,NULL,'email_from_address','1425350757@qq.com','2025-11-18 06:51:15','2025-11-18 06:51:57'),(60,NULL,NULL,'remind_mail_enable','1','2025-11-18 06:51:15','2025-11-18 06:52:01');
/*!40000 ALTER TABLE `v2_settings` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_stat`
--

DROP TABLE IF EXISTS `v2_stat`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_stat` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`record_at` int(11) NOT NULL,
`record_type` char(1) COLLATE utf8mb4_unicode_ci NOT NULL,
`order_count` int(11) NOT NULL COMMENT '订单数量',
`order_total` int(11) NOT NULL COMMENT '订单合计',
`commission_count` int(11) NOT NULL,
`commission_total` int(11) NOT NULL COMMENT '佣金合计',
`paid_count` int(11) NOT NULL,
`paid_total` int(11) NOT NULL,
`register_count` int(11) NOT NULL,
`invite_count` int(11) NOT NULL,
`transfer_used_total` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `v2_stat_record_at_unique` (`record_at`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_stat`
--

LOCK TABLES `v2_stat` WRITE;
/*!40000 ALTER TABLE `v2_stat` DISABLE KEYS */;
INSERT INTO `v2_stat` VALUES (1,1763395200,'d',8,2,0,0,6,0,6,0,'1147940306',1763482200,1763482200);
/*!40000 ALTER TABLE `v2_stat` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_stat_server`
--

DROP TABLE IF EXISTS `v2_stat_server`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_stat_server` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`server_id` int(11) NOT NULL COMMENT '节点id',
`server_type` char(11) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '节点类型',
`u` bigint(20) NOT NULL,
`d` bigint(20) NOT NULL,
`record_type` char(1) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT 'd day m month',
`record_at` int(11) NOT NULL COMMENT '记录时间',
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `server_id_server_type_record_at` (`server_id`,`server_type`,`record_at`),
KEY `server_id` (`server_id`),
KEY `record_at` (`record_at`),
KEY `v2_stat_server_server_id_index` (`server_id`),
KEY `v2_stat_server_record_at_index` (`record_at`),
KEY `v2_stat_server_u_index` (`u`),
KEY `v2_stat_server_d_index` (`d`)
) ENGINE=InnoDB AUTO_INCREMENT=711 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_stat_server`
--

LOCK TABLES `v2_stat_server` WRITE;
/*!40000 ALTER TABLE `v2_stat_server` DISABLE KEYS */;
INSERT INTO `v2_stat_server` VALUES (1,84,'vless',13591116,607932475,'d',1763395200,1763448669,1763478535),(4,70,'vless',78829,213261,'d',1763395200,1763449096,1763477547),(5,85,'vless',3951,17547,'d',1763395200,1763449546,1763453148),(6,67,'vless',3832216,41082200,'d',1763395200,1763449549,1763477575),(7,88,'vless',18124170,56842849,'d',1763395200,1763449733,1763453073),(11,33,'vless',23740,108409,'d',1763395200,1763449796,1763477541),(12,86,'vless',2147,14112,'d',1763395200,1763449861,1763450460),(13,83,'vless',7732722,135731010,'d',1763395200,1763449866,1763477560),(19,80,'hysteria',46654,133941,'d',1763395200,1763450098,1763477547),(43,89,'vless',34726297,227702660,'d',1763395200,1763455301,1763477572),(441,89,'vless',15461398,104584340,'d',1763481600,1763508053,1763517009),(442,84,'vless',7514045,59828430,'d',1763481600,1763508054,1763516883),(443,83,'vless',13583,55815,'d',1763481600,1763508068,1763516490),(444,33,'vless',14725,62273,'d',1763481600,1763508080,1763516487),(445,80,'hysteria',16871,76482,'d',1763481600,1763508089,1763516499),(446,70,'vless',15388,60819,'d',1763481600,1763508092,1763516493),(519,88,'vless',6987,36083,'d',1763481600,1763512083,1763516463),(689,90,'vless',139151,1136088,'d',1763481600,1763516439,1763516799);
/*!40000 ALTER TABLE `v2_stat_server` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_stat_user`
--

DROP TABLE IF EXISTS `v2_stat_user`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_stat_user` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`user_id` int(11) NOT NULL,
`server_rate` decimal(10,2) NOT NULL,
`u` bigint(20) NOT NULL,
`d` bigint(20) NOT NULL,
`record_type` char(2) COLLATE utf8mb4_unicode_ci NOT NULL,
`record_at` int(11) NOT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `server_rate_user_id_record_at` (`server_rate`,`user_id`,`record_at`),
KEY `v2_stat_user_user_id_server_rate_record_at_index` (`user_id`,`server_rate`,`record_at`),
KEY `v2_stat_user_u_index` (`u`),
KEY `v2_stat_user_d_index` (`d`)
) ENGINE=InnoDB AUTO_INCREMENT=719 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_stat_user`
--

LOCK TABLES `v2_stat_user` WRITE;
/*!40000 ALTER TABLE `v2_stat_user` DISABLE KEYS */;
INSERT INTO `v2_stat_user` VALUES (1,5,1.00,724720,1428911,'d',1763395200,1763448669,1763448790),(2,8,1.00,38079,22782,'d',1763395200,1763448669,1763448790),(7,1,1.00,75555711,945815907,'d',1763395200,1763449096,1763478535),(102,4,1.00,1843332,122510864,'d',1763395200,1763457644,1763464239),(439,1,1.00,21432570,139579913,'d',1763481600,1763508053,1763517009),(556,2,1.00,1749578,26260417,'d',1763481600,1763513255,1763516402);
/*!40000 ALTER TABLE `v2_stat_user` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_ticket`
--

DROP TABLE IF EXISTS `v2_ticket`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_ticket` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`user_id` int(11) NOT NULL,
`subject` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
`level` int(11) NOT NULL,
`status` int(11) NOT NULL DEFAULT '0' COMMENT '0:已开启 1:已关闭',
`reply_status` int(11) NOT NULL DEFAULT '1' COMMENT '0:待回复 1:已回复',
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
KEY `v2_ticket_status_index` (`status`),
KEY `v2_ticket_created_at_index` (`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_ticket`
--

LOCK TABLES `v2_ticket` WRITE;
/*!40000 ALTER TABLE `v2_ticket` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_ticket` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_ticket_message`
--

DROP TABLE IF EXISTS `v2_ticket_message`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_ticket_message` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`user_id` int(11) NOT NULL,
`ticket_id` int(11) NOT NULL,
`message` text COLLATE utf8mb4_unicode_ci NOT NULL,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_ticket_message`
--

LOCK TABLES `v2_ticket_message` WRITE;
/*!40000 ALTER TABLE `v2_ticket_message` DISABLE KEYS */;
/*!40000 ALTER TABLE `v2_ticket_message` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_traffic_reset_logs`
--

DROP TABLE IF EXISTS `v2_traffic_reset_logs`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_traffic_reset_logs` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`user_id` bigint(20) NOT NULL COMMENT '用户ID',
`reset_type` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '重置类型',
`reset_time` timestamp NOT NULL COMMENT '重置时间',
`old_upload` bigint(20) NOT NULL DEFAULT '0' COMMENT '重置前上传流量',
`old_download` bigint(20) NOT NULL DEFAULT '0' COMMENT '重置前下载流量',
`old_total` bigint(20) NOT NULL DEFAULT '0' COMMENT '重置前总流量',
`new_upload` bigint(20) NOT NULL DEFAULT '0' COMMENT '重置后上传流量',
`new_download` bigint(20) NOT NULL DEFAULT '0' COMMENT '重置后下载流量',
`new_total` bigint(20) NOT NULL DEFAULT '0' COMMENT '重置后总流量',
`trigger_source` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '触发来源',
`metadata` json DEFAULT NULL COMMENT '额外元数据',
`created_at` timestamp NULL DEFAULT NULL,
`updated_at` timestamp NULL DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_user_id` (`user_id`),
KEY `idx_reset_time` (`reset_time`),
KEY `idx_user_reset_time` (`user_id`,`reset_time`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_traffic_reset_logs`
--

LOCK TABLES `v2_traffic_reset_logs` WRITE;
/*!40000 ALTER TABLE `v2_traffic_reset_logs` DISABLE KEYS */;
INSERT INTO `v2_traffic_reset_logs` VALUES (1,1,'manual','2025-11-18 06:53:25',0,0,0,0,0,0,'order',NULL,'2025-11-18 06:53:25','2025-11-18 06:53:25'),(2,2,'manual','2025-11-18 09:09:25',0,0,0,0,0,0,'order',NULL,'2025-11-18 09:09:25','2025-11-18 09:09:25'),(3,3,'manual','2025-11-18 09:09:55',0,0,0,0,0,0,'order',NULL,'2025-11-18 09:09:55','2025-11-18 09:09:55'),(4,4,'manual','2025-11-18 09:18:55',0,0,0,0,0,0,'order',NULL,'2025-11-18 09:18:55','2025-11-18 09:18:55'),(5,5,'manual','2025-11-18 09:51:06',0,0,0,0,0,0,'order',NULL,'2025-11-18 09:51:06','2025-11-18 09:51:06'),(6,6,'manual','2025-11-18 12:51:46',0,0,0,0,0,0,'order',NULL,'2025-11-18 12:51:46','2025-11-18 12:51:46');
/*!40000 ALTER TABLE `v2_traffic_reset_logs` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `v2_user`
--

DROP TABLE IF EXISTS `v2_user`;
/*!40101 SET @saved_cs_client = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `v2_user` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`invite_user_id` int(11) DEFAULT NULL,
`telegram_id` bigint(20) DEFAULT NULL,
`email` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
`password` varchar(64) COLLATE utf8mb4_unicode_ci NOT NULL,
`password_algo` char(10) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`password_salt` char(10) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
`balance` int(11) NOT NULL DEFAULT '0',
`discount` int(11) DEFAULT NULL,
`commission_type` tinyint(4) NOT NULL DEFAULT '0' COMMENT '0: system 1: period 2: onetime',
`commission_rate` int(11) DEFAULT NULL,
`commission_balance` int(11) NOT NULL DEFAULT '0',
`t` int(11) NOT NULL DEFAULT '0',
`u` bigint(20) NOT NULL DEFAULT '0',
`d` bigint(20) NOT NULL DEFAULT '0',
`transfer_enable` bigint(20) NOT NULL DEFAULT '0',
`banned` tinyint(1) NOT NULL DEFAULT '0',
`is_admin` tinyint(1) NOT NULL DEFAULT '0',
`last_login_at` int(11) DEFAULT NULL,
`is_staff` tinyint(1) NOT NULL DEFAULT '0',
`last_login_ip` int(11) DEFAULT NULL,
`uuid` varchar(36) COLLATE utf8mb4_unicode_ci NOT NULL,
`group_id` int(11) DEFAULT NULL,
`plan_id` int(11) DEFAULT NULL,
`speed_limit` int(11) DEFAULT NULL,
`remind_expire` tinyint(4) DEFAULT '1',
`remind_traffic` tinyint(4) DEFAULT '1',
`token` char(32) COLLATE utf8mb4_unicode_ci NOT NULL,
`expired_at` bigint(20) DEFAULT '0',
`next_reset_at` int(11) DEFAULT NULL COMMENT '下次流量重置时间',
`last_reset_at` int(11) DEFAULT NULL COMMENT '上次流量重置时间',
`reset_count` int(11) NOT NULL DEFAULT '0' COMMENT '流量重置次数',
`device_limit` int(11) DEFAULT NULL,
`online_count` int(11) DEFAULT NULL,
`last_online_at` timestamp NULL DEFAULT NULL,
`remarks` text COLLATE utf8mb4_unicode_ci,
`created_at` int(11) NOT NULL,
`updated_at` int(11) NOT NULL,
PRIMARY KEY (`id`),
UNIQUE KEY `email` (`email`),
KEY `v2_user_u_d_expired_at_group_id_banned_transfer_enable_index` (`u`,`d`,`expired_at`,`group_id`,`banned`,`transfer_enable`),
KEY `v2_user_t_index` (`t`),
KEY `v2_user_online_count_index` (`online_count`),
KEY `v2_user_created_at_index` (`created_at`),
KEY `idx_next_reset_at` (`next_reset_at`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `v2_user`
--

LOCK TABLES `v2_user` WRITE;
/*!40000 ALTER TABLE `v2_user` DISABLE KEYS */;
INSERT INTO `v2_user` VALUES (1,NULL,NULL,'1425350757@qq.com','$2y$10$QEVh.ag5VqcDScc9aE027uikDOvRdVvBn.p.4866gYVnQmDYLYFju',NULL,NULL,0,NULL,0,NULL,0,1763517009,96988281,1085395820,1099511627776,0,1,1763517280,0,NULL,'e484d9ff-feb3-417e-b144-3fb0a27b5ce7',2,3,NULL,1,1,'bd418dd18f26307566fb64572a176d41',1858143205,1766040805,1763448805,1,NULL,0,'2025-11-19 01:50:09',NULL,1763447648,1763517323),(2,NULL,NULL,'937866880@qq.com','$2y$10$XkOL3uhPD2xNlq8q1fbVYedMQdCacQS9gFQwDTrBKqQdzJrvM8F6u',NULL,NULL,0,NULL,0,NULL,0,1763516402,1749578,26260417,1099511627776,0,0,1763513207,0,NULL,'37cce889-786d-48a5-83e5-ce167e8a59ba',1,1,0,1,1,'de54bb5c2250899425c82946bdfb8d24',1858151365,1766048965,1763456965,1,3,0,'2025-11-19 00:52:59',NULL,1763456896,1763513881),(3,NULL,NULL,'3194517886@qq.com','$2y$10$SZ.zxq5JezaTMgUkGIKYDOik6JczhABApHI4MUpNUD/xQZAjqtZH6',NULL,NULL,0,NULL,0,NULL,0,0,0,0,1099511627776,0,0,1763456953,0,NULL,'d3d5ce08-bef2-4b75-8541-8753f6d0e8f1',1,1,0,1,1,'5281e2c8d507b7149351ad6cf09a5f4a',1858151395,1766048995,1763456995,1,3,NULL,NULL,NULL,1763456953,1763456995),(4,NULL,NULL,'yanku8471@gmail.com','$2y$10$M9xR66WlAL2yggTojV4dp.YhYCIbfBjDB7CQkZMvTL3gLOdfH6x32',NULL,NULL,0,NULL,0,NULL,0,1763464239,1843332,122510864,1099511627776,0,0,1763457509,0,NULL,'b7bee176-62f1-4f1e-9d61-18d2d624286d',1,1,0,1,1,'04260cf3776aa59a3b71ee17f75b96d6',1826529535,1766049535,1763457535,1,3,0,'2025-11-18 11:05:39',NULL,1763457509,1763464261),(5,NULL,NULL,'mizi280012@gmail.com','$2y$10$zF1YcbtxZzzGo276HPpx3.YZBH.q/DG3wXxOG3oG.1.zM5GQgurH.',NULL,NULL,0,NULL,0,NULL,0,0,0,0,1099511627776,0,0,1763459248,0,NULL,'bf06f442-d175-4c54-8b7d-c84b5ef594a7',1,1,0,1,1,'13285e5a4b725658f7b011b75affcb0e',1858153866,1766051466,1763459466,1,3,NULL,NULL,NULL,1763459248,1763459466),(6,NULL,NULL,'2766016387@qq.com','$2y$10$YYwFqGArF54253QMdr5mOuukjFAxouO9qq8PWEzYLkwYF8pI6BEnW',NULL,NULL,0,NULL,0,NULL,0,0,0,0,1099511627776,0,0,1763507968,0,NULL,'691d2162-a3b1-4dba-afbb-82f16125290c',1,1,0,1,1,'a01e08fecb37b5d5c0f672580a096b9d',1858164706,1766062306,1763470306,1,3,NULL,NULL,NULL,1763470291,1763507968);
/*!40000 ALTER TABLE `v2_user` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Dumping events for database 'xboard'
--

--
-- Dumping routines for database 'xboard'
--
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2025-11-19 10:01:48
解释报错

---
## 🤖 Assistant

好的，根据您提供的 SQL dump 文件，我们来分析一下可能的问题，并提供解决方案。

您提供的 `xboard.sql` 文件看起来是一个完整的数据库备份，包含了建表语句（`CREATE TABLE`）和数据插入语句（`INSERT INTO`）。将这样的文件导入到一个**全新的、空的数据库**中通常不会有问题。

报错通常发生在以下几种情况，让我们逐一排查并解释：

### 可能的报错原因及解释

#### 1. 错误：`ERROR 1050 (42S01) at line ...: Table '...' already exists`

* **错误解释**：这个错误的意思是“表已经存在”。当您尝试创建一个已经存在的表时，MySQL 会报这个错。
* **问题场景**：
 * 您正在尝试将 `xboard.sql` 导入到一个**已经存在** `xboard` 数据库，并且这个数据库里已经有了一些表（例如 `failed_jobs`, `migrations` 等）。
 * 在导入过程中，脚本执行到 `CREATE TABLE` 语句时，发现同名的表已经存在，因此导入失败。
* **解决方案**：
 * **方案A（推荐）：** 在导入之前，先彻底删除旧的数据库。
```sql
        DROP DATABASE IF EXISTS xboard;
        CREATE DATABASE xboard CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
        ```
        然后再执行导入命令。
    *   **方案B（包含在您的SQL文件中）：** 您的 SQL 文件在每个 `CREATE TABLE` 语句前都包含了 `DROP TABLE IF EXISTS table_name;`。这本应能防止此错误。如果仍然报错，可能是因为某些权限问题或数据库状态异常导致 `DROP` 失败。但通常情况下，这个 `DROP` 语句是有效的。

#### 2. 错误：`ERROR 1046 (3D000) at line ...: No database selected`

*   **错误解释**：没有选择要操作的数据库。MySQL 不知道您想把这些表创建在哪个数据库里。
*   **问题场景**：
    *   您在执行导入命令时，没有指定目标数据库。例如，您可能只运行了 `mysql -u root -p < xboard.sql`，而没有加上数据库名。
    *   您的 `xboard.sql` 文件开头缺少 `USE xboard;` 语句。
*   **解决方案**：
    *   **方案A（在命令行中指定）：**
        ```bash
        # 1. 先创建数据库
        mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS xboard CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

        # 2. 导入时指定数据库
        mysql -u root -p xboard < xboard.sql
        ```
    *   **方案B（修改SQL文件）：** 在 `xboard.sql` 文件的最顶部（所有 `CREATE TABLE` 之前）添加一行：
        ```sql
        USE xboard;
        ```
        这样导入时脚本会自动选择数据库。

#### 3. 错误：`ERROR 1044 (42000) at line ...: Access denied for user '...'@'localhost' to database '...'`

*   **错误解释**：权限不足。您用来执行导入操作的数据库用户没有在目标数据库上创建表、删除表或插入数据的权限。
*   **问题场景**：
    *   您使用了我们之前创建的 `xboard` 用户（密码 `AtB7b8MwCJy3xDeh`）来执行导入。这个用户默认只有对 `xboard` 数据库的操作权限，而没有 `CREATE DATABASE` 或 `DROP DATABASE` 的权限。
    *   您使用了 `root` 用户，但可能是在一个权限受限的环境中（例如某些云服务商提供的数据库）。
*   **解决方案**：
    *   **确保使用 `root` 用户或具有同等高权限的用户进行导入操作**，因为导入过程涉及删表、建表等高级权限。
        ```bash
        mysql -u root -p'你的root密码' xboard < xboard.sql
        ```

#### 4. 错误：字符集或排序规则相关错误 (例如 `Illegal mix of collations`)

*   **错误解释**：数据表或字段的字符集/排序规则不匹配或不受支持。
*   **问题场景**：
    *   您手动创建的 `xboard` 数据库默认字符集不是 `utf8mb4`。
    *   您使用的 MySQL 版本过低，不支持 `utf8mb4_unicode_ci` 排序规则。但您使用的是 5.7.44，这应该不是问题。
*   **解决方案**：
    *   确保数据库创建时就指定了正确的字符集和排序规则。我们之前的 `docker-compose.yml` 和 `my.cnf` 文件已经保证了这一点，并且创建数据库时也推荐使用：
        ```sql
        CREATE DATABASE xboard CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
        ```

### 如何在 Docker 环境中正确导入 `xboard.sql`

结合我们之前的 `docker-compose` 设置，这里是**最推荐的、一站式的解决方案**，可以自动完成数据库初始化。

这个方法利用了 MySQL 官方镜像的一个特性：在容器第一次启动时，它会自动执行 `/docker-entrypoint-initdb.d/` 目录下的所有 `.sh`, `.sql`, `.sql.gz` 文件。

**1. 修改您的文件结构：**

将 `xboard.sql` 文件放入一个新的 `init` 文件夹中。

```
xboard-mysql/
├── docker-compose.yml
├── config/
│ └── my.cnf
└── init/
 └── xboard.sql <-- 把您的 SQL 文件放在这里
```

**2. 修改 `docker-compose.yml` 文件：**

在 `volumes` 部分添加一行，将 `init` 文件夹挂载到容器的初始化目录。

```yaml
version: '3.8'

services:
mysql:
 image: mysql:5.7.44
 container_name: mysql_xboard_57
 restart: unless-stopped
 environment:
 MYSQL_ROOT_PASSWORD: CHANGE_ME_TO_A_VERY_STRONG_SECRET_PASSWORD
 MYSQL_DATABASE: xboard # 这一行很重要，初始化脚本会把数据导入这个库
 # 不要在这里定义 MYSQL_USER 和 MYSQL_PASSWORD
 # 因为您的 SQL 文件里可能已经包含了用户创建逻辑
 # 如果没有，您可以在 SQL 文件里添加，或者在初始化后再手动创建
 TZ: Asia/Shanghai
 ports:
 - "3306:3306"
 volumes:
 # 数据持久化
 - mysql-data:/var/lib/mysql
 # 挂载自定义配置文件
 - ./config/my.cnf:/etc/mysql/conf.d/charset.cnf
 # !!! 新增：挂载初始化 SQL 脚本 !!!
 - ./init:/docker-entrypoint-initdb.d
 networks:
 - xboard-net

volumes:
mysql-data:

networks:
xboard-net:
```

**3. 操作步骤：**

1. **清理旧环境（非常重要！）**：如果之前已经启动过容器，必须先彻底删除它和它关联的数据卷，否则初始化脚本不会再次运行。
 ```bash
 # 在 xboard-mysql 目录下执行
 docker-compose down -v
 ```
 ` -v` 参数会删除名为 `mysql-data` 的数据卷，确保一个全新的开始。

2. **重新启动容器：**
 ```bash
 docker-compose up -d
 ```

**工作原理：**
* 容器首次启动时，发现 `MYSQL_DATABASE=xboard`，于是它会先创建一个名为 `xboard` 的空数据库。
* 接着，它会检查 `/docker-entrypoint-initdb.d/` 目录，并按字母顺序执行里面的脚本。
* 它会找到 `xboard.sql` 并执行，将所有的表和数据导入到刚刚创建的 `xboard` 数据库中。

这个方法是最可靠的，因为它避免了所有手动操作可能带来的权限和数据库选择问题，实现了“开箱即用”的数据初始化。

---
## 🧑‍💻 User

Access denied for user 'xboard'@'42.236.216.175' (using password: YES) 连接不上了

---
## 🤖 Assistant

好的，这个报错 `Access denied for user 'xboard'@'42.236.216.175' (using password: YES)` 是一个非常经典的 MySQL 权限问题。让我们来详细分析并解决它。

### 错误信息解读

* `Access denied`: **访问被拒绝**。这是核心问题。
* `for user 'xboard'`: 哪个用户被拒绝了？是 **'xboard'** 这个用户。
* `@'42.236.216.175'`: **从哪里连接时被拒绝了？** 这是最关键的信息！MySQL 服务器收到了一个来自 IP 地址 `42.236.216.175` 的连接请求。
* `(using password: YES)`: 这说明您的应用程序**确实带着密码**来连接了，但“用户名、来源 IP、密码”这三者的组合验证失败了。问题不在于没提供密码，而在于这个组合是无效的。

### 问题根源分析

这个错误 99% 的可能性是由于 **MySQL 的用户授权机制** 引起的。

MySQL 的用户权限并不仅仅是 `用户名` + `密码`，而是 `用户名` + `来源主机(Host)` + `密码`。

当您使用 Docker Compose 的环境变量 `MYSQL_USER` 和 `MYSQL_PASSWORD` 创建用户时，MySQL 官方镜像为了安全和通用性，会创建这样一个用户：`'xboard'@'%'`。这里的 `'%'` 是一个通配符，意思是**允许来自任何 IP 地址的连接**。

正常情况下，这应该能让您从 `42.236.216.175` 连接上。但现在失败了，说明很可能发生了以下几种情况之一：

1. **最常见的原因：** 用户 `'xboard'` 的 host 字段不是 `'%'`，而是被限制为了 `localhost` 或某个 Docker 内部 IP。这可能是在后续操作中被无意修改了，或者在某些特殊的初始化脚本中被设定了。
2. **密码错误：** 您的应用程序（位于 `42.236.216.175`）使用的密码 `AtB7b8MwCJy3xDeh` 与数据库中为 `'xboard'@'...'` 用户设置的密码不匹配。可能存在复制粘贴时的空格、特殊字符转义等问题。
3. **网络问题：** 防火墙阻止了连接，导致连接请求根本没有正确到达 MySQL 服务端口。

### 解决方案：一步步排查

请按照以下步骤进行排查和修复。

#### 第 1 步：确认密码无误

请再三确认您在应用程序的配置文件（例如 `.env` 文件）中填写的数据库密码，与您在 `docker-compose.yml` 中设置的 `AtB7b8MwCJy3xDeh` **完全一致**，没有任何多余的空格或隐藏字符。

#### 第 2 步：进入容器，检查并修复用户权限

这是解决问题的核心步骤。我们需要登录到 MySQL 容器内部，直接查看用户的权限设置。

1. **进入正在运行的 MySQL 容器：**
 打开您服务器的终端，执行以下命令。`mysql_xboard_57` 是我们之前设置的容器名。
 ```bash
 docker exec -it mysql_xboard_57 bash
 ```

2. **登录到 MySQL 服务：**
 进入容器后，您会得到一个新的 shell 提示符。现在，使用 `root` 用户登录 MySQL。系统会提示您输入 `root` 密码（就是您为 `MYSQL_ROOT_PASSWORD` 设置的那个）。
 ```bash
 mysql -u root -p
 ```

3. **查询 `xboard` 用户的 Host 设置：**
 登录成功后，在 MySQL 提示符 (`mysql>`) 下执行以下 SQL 查询语句：
 ```sql
 SELECT user, host FROM mysql.user WHERE user = 'xboard';
 ```

4. **分析查询结果：**
 * **理想情况（结果应该是这样）：**
 ```
 +--------+------+
 | user | host |
 +--------+------+
 | xboard | % |
 +--------+------+
 ```
 如果 `host` 字段确实是 `%`，那么问题 99% 是您应用程序端的密码错误或网络防火墙问题（见第 3 步）。

 * **问题情况（很可能是这样）：**
 ```
 +--------+-----------+
 | user | host |
 +--------+-----------+
 | xboard | localhost |
 +--------+-----------+
 ```
 如果 `host` 字段是 `localhost`、`127.0.0.1` 或者某个 `172.x.x.x` 的 Docker 内部 IP，那么就找到了问题所在！它只允许本地或内部连接，**拒绝了来自公网 IP `42.236.216.175` 的访问**。

5. **修复权限（如果 Host 不对）：**
 在 MySQL 提示符下，执行以下**任一**条修复命令。推荐使用第一条，因为它同时重置了 host 和密码，确保万无一失。

 * **方法一（推荐，授权并重置密码）：**
 ```sql
 GRANT ALL PRIVILEGES ON xboard.* TO 'xboard'@'%' IDENTIFIED BY 'AtB7b8MwCJy3xDeh';
 ```

 * **方法二（仅修改 Host）：**
 ```sql
 UPDATE mysql.user SET host = '%' WHERE user = 'xboard';
 ```

6. **刷新权限并退出：**
 无论使用哪种方法修复，都**必须执行**下面的命令来让更改立即生效！
 ```sql
 FLUSH PRIVILEGES;
 EXIT;
 ```
 执行完 `EXIT` 后，再输入一次 `exit` 退出容器，回到您服务器的 shell。

现在，再次尝试从您的应用程序连接数据库。问题很可能已经解决。

#### 第 3 步：检查网络防火墙

如果完成第 2 步后问题依旧，那么需要检查服务器的网络防火墙。

1. **检查服务器防火墙（以 UFW 为例）：**
 在您的**服务器终端**（不是容器内）运行：
 ```bash
 sudo ufw status
 ```
 检查列表中是否有允许 `3306` 端口的规则。如果没有，请添加它：
 ```bash
 sudo ufw allow 3306/tcp
 sudo ufw reload
 ```

2. **检查云服务商安全组：**
 如果您使用的是阿里云、腾讯云、AWS 等云服务器，请登录到您的云服务控制台，找到对应的“安全组”或“防火墙”规则。确保已经为该服务器实例添加入方向（Inbound）规则，允许 TCP 协议的 `3306` 端口从源 IP `42.236.216.175` (或者更宽松的 `0.0.0.0/0`，但请注意安全风险) 访问。

### 总结排查思路

1. **先确认**：应用程序配置的密码和 IP 地址是否正确。
2. **再检查**：服务器防火墙和云安全组是否放行了 `3306` 端口。
3. **最后深入**：进入 Docker 容器，用 `root` 登录 MySQL，查询 `mysql.user` 表，确认 `xboard` 用户的 `host` 字段是 `%`，如果不是，用 `GRANT` 命令修复并 `FLUSH PRIVILEGES`。

---
## 🧑‍💻 User

无法连接上

---
## 🤖 Assistant

好的，感谢您提供截图，这让问题清晰了很多！我们来一步步分析解决。

从您的截图来看，有几个关键信息点：

1. **文件结构正确**：您在 `/opt/xboard-mysql` 目录下的文件结构是正确的，包含了 `config`、`init` 文件夹和 `docker-compose.yml` 文件。
2. **错误信息依旧**：错误仍然是 `Access denied for user 'xboard'@'42.236.216.175'`。这说明**问题核心还是 MySQL 的用户权限设置**。
3. **连接配置信息**：
 * **主机名**：`74.48.43.241`
 * **端口**：`3306`
 * **用户名**：`xboard`
 * **密码**：`AtB7b8MwCJy3xDeh`
 * **数据库**：`xboard`

现在，让我们把这些信息串起来分析。

### 问题分析

#### 关键矛盾点

* 您的应用程序（截图中的数据库管理工具）是从 IP `42.236.216.175` 发起连接请求的。
* MySQL 服务器运行在 IP `74.48.43.241` 的服务器上。
* MySQL 内部的用户权限系统，**拒绝了**来自 `42.236.216.175` 的 `xboard` 用户的访问请求。

这再次印证了我们之前的判断：**问题不在于 Docker Compose 的配置，而在于 MySQL 数据库内部的用户权限没有正确设置为允许外部 IP 访问。**

当您使用我们的 `docker-compose.yml` 配合 `init/xboard.sql` 启动时，MySQL 会执行 `xboard.sql` 里的所有语句来初始化数据库。**很可能您的 `xboard.sql` 文件中包含了创建用户或修改权限的语句，将 `'xboard'` 用户的 host 限制为了 `localhost` 或其他值，从而覆盖了 Docker 自动创建的 `'xboard'@'%'` 用户。**

### 解决方案（动手修复）

请完全按照下面的步骤操作，这会直接修复权限问题。

**目标：** 我们要强制将 MySQL 里的 `'xboard'` 用户授权给所有 IP (`%`)，并确保密码正确。

---

#### 第 1 步：进入 MySQL 容器

在您的服务器（IP: `74.48.43.241`）上打开终端，执行以下命令进入正在运行的 MySQL 容器：

```bash
docker exec -it mysql_xboard_57 bash
```
* `mysql_xboard_57` 是您 `docker-compose.yml` 里设置的容器名，如果不同请替换。

---

#### 第 2 步：登录 MySQL 服务

进入容器后，使用 `root` 用户登录到 MySQL 数据库。系统会提示您输入密码，这个密码是您在 `docker-compose.yml` 中为 `MYSQL_ROOT_PASSWORD` 设置的值。

```bash
mysql -u root -p
```

---

#### 第 3 步：执行授权命令（核心步骤）

登录成功后，您会看到 `mysql>` 提示符。请**完整复制**并执行下面这条 SQL 命令。这条命令会完成三件事：
1. 找到或创建一个名为 `xboard` 的用户。
2. 允许这个用户从**任何 IP 地址 (`%`)** 连接。
3. 将这个用户对 `xboard` 数据库的**所有权限**授予它，并设置/重置其密码为 `AtB7b8MwCJy3xDeh`。

```sql
GRANT ALL PRIVILEGES ON xboard.* TO 'xboard'@'%' IDENTIFIED BY 'AtB7b8MwCJy3xDeh';
```

**注意**：即使您认为密码是正确的，也请执行这条命令，因为它会强制刷新用户的 host 和密码，确保万无一失。

---

#### 第 4 步：刷新权限并退出

执行完上面的授权命令后，**必须**执行下面的命令来让权限变更立即生效：

```sql
FLUSH PRIVILEGES;
```

然后退出 MySQL 和容器：

```sql
EXIT;
```
现在您回到了容器的 shell，再次输入 `exit` 退出容器，回到服务器的终端。

```bash
exit
```

---

#### 第 5 步：再次尝试连接

现在，回到您的数据库管理工具（截图中的那个），不要修改任何配置，直接点击【连接】按钮。

**99% 的可能性，现在应该可以成功连接了。**

### 如果仍然失败？

如果万一还是失败，请在**执行完上述所有步骤后**，再次重复第 1、2 步进入 MySQL，然后执行查询，并将结果截图或复制给我：

```sql
SELECT user, host FROM mysql.user WHERE user = 'xboard';
```
这条命令的返回结果将是解决问题的最终线索。它必须显示 `host` 为 `%` 才行。

---
## 🧑‍💻 User

mysql> GRANT ALL PRIVILEGES ON xboard.* TO 'xboard'@'%' IDENTIFIED BY 'AtB7b8MwCJy3xDeh';
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> EXIT;
Bye

---
## 🤖 Assistant

太棒了！您提供的这个执行日志是非常关键且积极的信号。

`Query OK, 0 rows affected, 1 warning (0.01 sec)`

这行反馈告诉我们：

* `Query OK`: **命令成功执行了！** MySQL 接受了您的 `GRANT` 授权语句。
* `0 rows affected`: 这很正常。`GRANT` 语句是用来修改权限的，它不会改变表中的行，所以受影响的行数是 0。
* `1 warning`: 这里有一个警告。在 MySQL 5.7 中，`GRANT` 语句自动创建用户并设置密码的方式已经被标记为“不推荐”（deprecated），但它**仍然可以正常工作**。这个警告只是在提醒您，在更新的 MySQL 版本（如 8.0）中，您需要先用 `CREATE USER` 创建用户，再用 `GRANT` 授权。对于 5.7.44 版本，这个警告完全可以忽略，**您的操作是成功的**。

`FLUSH PRIVILEGES;` 也成功执行了，这意味着新的权限规则已经被加载并生效。

### 结论与下一步

**您已经成功地修复了 MySQL 内部的权限问题。**

现在，MySQL 服务器 (`74.48.43.241`) 上的 `xboard` 用户已经被正确配置为：
* **允许来自任何 IP 地址 (`%`) 的连接。**
* **密码被强制重置为 `AtB7b8MwCJy3xDeh`。**

#### **请立即行动：**

1. 回到您的数据库管理工具（您之前截图的那个界面）。
2. **不要修改任何配置信息**（主机、端口、用户名、密码都保持原样）。
3. 直接点击 **【连接】** 按钮。

现在，连接应该可以成功建立。

---

### 如果仍然无法连接（极小概率）

如果此时仍然收到 `Access denied` 错误，那么问题就几乎可以 100% 排除是 MySQL 用户权限配置的问题了。剩下的可能性只有两个：

1. **客户端密码错误**：请您**手动删除**密码框里的所有内容，然后**一个字符一个字符地重新输入** `AtB7b8MwCJy3xDeh`。有时候复制粘贴会带上隐藏的空格或特殊字符。
2. **网络路径问题**：
 * **服务器防火墙**：检查 `74.48.43.241` 这台服务器上的防火墙（如 `ufw`, `firewalld`）是否允许外部访问 `3306` 端口。
 * **云服务商安全组**：检查您云服务商（如阿里云、腾讯云等）的控制台，确保 `74.48.43.241` 实例的安全组规则允许来自您当前 IP (`42.236.216.175`) 对 `3306` 端口的入站访问。

但根据您目前的操作和反馈，我非常有信心您现在重试连接就能成功。请试试看！

---
## 🧑‍💻 User

退出容器

---
## 🤖 Assistant

好的，您已经成功退出了 MySQL 命令行。

现在您应该还在 Docker 容器的命令行界面里。要退出容器，返回到您自己服务器的终端，请直接输入以下命令然后按回车：

```bash
exit
```

执行后，您就会看到您熟悉的服务器命令行提示符了（例如 `root@your-server:~#`）。

退出容器后，您就完成了所有的修复步骤。**现在请立刻去您的数据库管理工具，点击【连接】按钮进行测试。**

期待您的好消息！
