# DaVinci项目服务器
![Lint](https://github.com/elliotmatson/Docker-Davinci-Resolve-Project-Server/actions/workflows/lint.yml/badge.svg)
![Healthcheck](https://github.com/elliotmatson/Docker-Davinci-Resolve-Project-Server/actions/workflows/stack-healthcheck.yml/badge.svg)
![Docker Build](https://github.com/elliotmatson/Docker-Davinci-Resolve-Project-Server/actions/workflows/docker.yml/badge.svg)

**中文** | [English](https://github.com/fly6516/Docker-Davinci-Resolve-Project-Server/blob/52e3bb81de9b2f814b51523c8ea1e290d955f990/README.md)

带有自动备份功能的DaVinci项目服务器

## 目录
- [DaVinci项目服务器](#DaVinci项目服务器)
  - [目录](#目录)
  - [项目介绍](#项目介绍)
    - [项目功能](#项目功能)
  - [项目设置](#项目设置)
    - [数据库](#数据库)
    - [数据库备份](#数据库备份)
    - [PGAdmin-数据库图形化网页](#PGAdmin-数据库图形化网页)
    - [磁盘位置](#磁盘位置)
  - [安装](#安装)
    - [QNAP 威联通安装](#QNAP-威联通安装)
    - [Synology 群晖安装](#Synology-群晖安装)
    - [Linux安装](#Linux安装)
  - [使用不同的Postgres数据库版本](#使用不同的Postgres数据库版本)
    - [设置一个PostgreSQL 9.5 或 11 版本的项目服务器](#设置一个PostgreSQL-95-或-11-版本的项目服务器)
  - [鸣谢](#鸣谢)

## 项目介绍

有很多方法可以托管一个Resolve项目服务器，但每种方法都有各自的问题。官方的项目服务器需要手动备份，而其他选项对没有IT团队支持的人来说可能会很复杂。希望这能成为小团队更可靠、更简单的解决方案！

### 项目功能
- **轻量级** -  基于Docker，因此不需要完整的macOS或Windows机器或虚拟机。
- **跨平台** -  可安装在Windows、Mac、Linux、威联通、群晖、树莓派，几乎任何能够运行Docker的设备上。
- **兼容 Resolve 的现有备份/还原功能** -  所有备份文件使用标准的Resolve *.backup文件格式，并且可以从Resolve的用户界面进行还原。
- **内置 PGAdmin 服务器** - PGAdmin是用于管理PostgreSQL服务器的工具，对于诊断问题以及迁移/更新整个服务器非常有帮助。

## 项目设置
（请从源文件下载docker-compose.yml，并按以下步骤配置）
我们需要在 docker-compose.yml 文件的顶部进行一些编辑，以配置我们的安装:
```yaml
---
version: '3.8'
x-common:
  database: &db-environment
    POSTGRES_DB: database
    POSTGRES_USER: &pg-user postgres
    POSTGRES_PASSWORD: DaVinci
    TZ: America/Chicago
    POSTGRES_LOCATION: &db-location "???:/var/lib/postgresql/data"
  backup: &backup-environment
    SCHEDULE: "@daily"
    BACKUP_KEEP_DAYS: 7
    BACKUP_KEEP_WEEKS: 4
    BACKUP_KEEP_MONTHS: 6
    BACKUP_LOCATION: &bk-location "???:/backups"
  admin: &admin-environment
    PGADMIN_DEFAULT_EMAIL: admin@admin.com
    PGADMIN_DEFAULT_PASSWORD: root
    PGADMIN_PORT: &pgadmin-port "3001:80"
...
```

### 数据库
要配置服务器本身，我们需要配置以下环境变量：

| 环境变量            | 含义                                                                            |
|---|---|
| POSTGRES_DB       | 数据库名称。可以随意命名(需要在连接时填写)。                                                       |
| POSTGRES_USER     | 用于连接数据库的用户名。Resolve默认是"postgres"。                                             |
| POSTGRES_PASSWORD | 用于连接数据库的密码。Resolve默认是"DaVinci"。                                               |
| TZ                | 您的时区，这里有[一个列表](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)。 |
| POSTGRES_LOCATION | 数据库存储的位置。请参见[卷位置](#volume-locations)。                                         |

### 数据库备份
要配置备份，我们需要设置以下变量：

| 环境变量           | 含义                                                                 |
|---|---|
| SCHEDULE         | 一个[cron字符串](https://www.freeformatter.com/cron-expression-generator-quartz.html)，用于指定备份的创建频率。可以是"@daily"、"@every 1h"等。 |
| BACKUP_KEEP_DAYS    | 保留的每日备份数量，超过此数量后将删除旧备份。                             |
| BACKUP_KEEP_WEEKS   | 保留的每周备份数量，超过此数量后将删除旧备份。                             |
| BACKUP_KEEP_MONTHS  | 保留的每月备份数量，超过此数量后将删除旧备份。                             |
| BACKUP_LOCATION     | 备份文件存储的位置。请参见[卷位置](#volume-locations)。                   |

### PGAdmin-数据库图形化网页
要配置PGAdmin，我们需要设置以下变量：

| 环境变量               | 含义                                   |
|---|---|
| PGADMIN_DEFAULT_EMAIL    | 用于PGAdmin登录的邮箱。默认是"admin@admin.com"。 |
| PGADMIN_DEFAULT_PASSWORD | 用于PGAdmin登录的密码。默认是"root"。            |
| PGADMIN_PORT             | 配置PGAdmin的端口，语法为"对外端口:80"。           |

### 磁盘位置
数据库和备份的位置取决于你安装的平台。你需要知道存储它们的文件夹的完整路径。例如，在 威联通 NAS 上，如果我想将备份存储在名为"Videos"的共享文件夹中的"Backups"文件夹中，路径将是```/shares/Videos/Backups/```，那么我的```BACKUP_LOCATION```值将如下所示:
```yaml
BACKUP_LOCATION: &bk-location "/shares/Videos/Backups/:/backups"
```
在Ubuntu上，如果我想将数据库存储在名为“database”的文件夹中，该文件夹位于名为“johndoe”的用户的主目录中，路径将是```/home/johndoe/database/```，那么我的```POSTGRES_LOCATION```值将如下所示:
```yaml
POSTGRES_LOCATION: &db-location "johndoe/database/:/var/lib/postgresql/data"
```

我建议将数据库存放在SSD上，使用机械硬盘时访问速度会明显变慢。

一旦配置好这些设置，保存修改后的`docker-compose.yml`文件，然后继续进行安装！

## 安装

### QNAP 威联通安装
在 威联通 NAS 上安装相对简单。请注意，务必将数据库文件放在SSD上，你会感谢我的。

1. 如果还没有按照docker，请从威联通应用商店安装Container Station。
2. 在Container Station中，点击“创建”，然后点击“创建应用程序”。
3. 给你的应用程序命名，随便取个名字（例如：ResolveServer）。
4. 复制/粘贴你修改后的`docker-compose.yml`文件，点击“验证YAML”进行测试，如果通过验证，点击“创建”。
5. Container Station会下载所需的文件并启动应用程序。完成后，你应该能够使用数据库名称和凭据将Resolve连接到 威联通 的IP地址。

### Synology 群晖安装
请见 [该讨论](https://github.com/elliotmatson/Docker-Davinci-Resolve-Project-Server/discussions/15#discussioncomment-4615278)
fly6516测试中，待补充。

### Linux安装
1. 按照你Linux发行版的[Docker安装说明](https://docs.docker.com/engine/install/)进行安装。
2. 安装[Docker Compose](https://docs.docker.com/compose/install/)。
3. 将你修改后的`docker-compose.yml`文件移动到Linux机器上的一个文件夹中，然后在终端中导航到该文件夹。
4. 运行：
   ```bash
   docker-compose up -d
   ```
5. Docker Compose将下载所需的文件并启动应用程序。完成后，你应该能够使用数据库名称和凭据将Resolve连接到你的Linux服务器实例的IP地址。

## 使用不同的Postgres数据库版本
通常情况下，Resolve对不匹配的PostgreSQL版本的容忍度较低。Resolve 18使用PostgreSQL 13，而这个仓库现在默认使用这个版本。Resolve 17及以下版本使用PostgreSQL 9.5。不幸的是，主要版本9.5已达到生命周期结束（EOL），特别是9.5.4版本存在许多漏洞，使其不安全。

由于大多数人仍然使用默认的Resolve凭据来访问他们的服务器，因此安全性通常不是最主要的担忧。但如果你尝试用旧版Resolve保护项目服务器，建议升级到受支持的PostgreSQL版本。

Resolve 17及以下版本仍然使用已在PostgreSQL 12中移除的遗留功能，因此可以使用的最新主要版本是11，该版本将维持支持直到2023年11月9日。

### 设置一个PostgreSQL 9.5 或 11 版本的项目服务器
要设置PostgreSQL 9.5或11 服务器，而不是13，有两行需要在docker_compose.yml中更改：
```yaml
services:
  postgres:
    image: postgres:13
    ...
  pgbackups:
    image: prodrigestivill/postgres-backup-local:13
    ...
...
```
更改为以下内容：
```yaml
services:
  postgres:
    image: postgres:9.5
    ...
  pgbackups:
    image: prodrigestivill/postgres-backup-local:9.5
    ...
...
```
## 鸣谢
-[prodrigestivill](https://github.com/prodrigestivill/) 及其 [PostgreSQL Backup docker image](https://github.com/prodrigestivill/docker-postgres-backup-local)的贡献
