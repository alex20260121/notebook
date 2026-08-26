# MySQL数据库
MySQL 是全球最流行的 **开源关系型数据库管理系统（RDBMS）** 之一。它基于 SQL（结构化查询语言） 构建，目前由
Oracle（甲骨文）公司维护与支持。在互联网应用、企业级软件及 Web 开发中占据着极其重要的地位。

以下从核心概念、架构特性、核心优势及适用场景对 MySQL 进行全面概述：

## 核心架构与特性

1.  可插拔的存储引擎（Pluggable Storage Engines）

      - MySQL 的架构在查询解析/优化与底层数据存储之间解耦，支持不同的存储引擎：
          - InnoDB（默认且最常用）：完全支持 ACID 事务、行级锁（Row-level Locking）、外键约束及崩溃恢复能力。
          - MyISAM：早期的默认引擎，不支持事务和行级锁（仅表级锁），但读取速度快、占用空间小（现主要用于特定只读或非事务场景）。
          - Memory：数据全存放在内存中，适合临时缓存或高速检索。

2.  事务与数据一致性（ACID）

      - 依托 InnoDB 引擎，MySQL 支持多版本并发控制（MVCC），支持读未提交、读已提交、可重复读（默认）、串行化四种事务隔离级别。
      - 通过 Redo Log（重做日志） 保证持久性，通过 Undo Log（回滚日志） 保证原子性和实现 MVCC。

3.  高效的索引机制

      - 默认采用 B+ Tree 索引结构，支持主键索引（聚簇索引）、二级索引（非聚簇索引）、联合索引、覆盖索引以及全文索引（Full-Text）。

4.  复制与高可用（Replication & HA）

      - Binlog 机制：基于二进制日志（Binary Log）实现数据主从复制（异步复制、半同步复制）。
      - 高可用方案：支持主从架构（Primary-Replica）、组复制（MGR / Group Replication）以及官方的 MySQL
        InnoDB Cluster 架构。

## 主要优势

  - 成熟稳定与高性能：经过几十年的大规模工业级验证，在典型 OLTP（在线事务处理）场景下吞吐量极高。
  - 开源与生态丰富：拥有海量的开源社区资源、周边工具（如 Navicat、DBeaver、Percona
    Toolkit）及完备的驱动支持（Java/Python/Go/PHP/Node.js 等）。
  - 轻量且易于部署：资源占用相对较小，安装配置直观，各大云厂商（AWS RDS、阿里云 RDS、腾讯云等）均提供成熟的托管服务。
  - 扩展性强：单机性能受限时，可通过读写分离、分库分表（ShardingSphere 等方案）或分布式方案拓展。

## 典型应用场景

1.  Web 与移动端应用：经典 LAMP / LNMP（Linux + Apache/Nginx + MySQL +
    PHP/Python）栈的核心数据底座。
2.  电子商务与交易系统：处理订单、用户、库存等对事务和数据一致性要求较高的场景。
3.  内容管理与社交系统：如 WordPress、各类论坛、CMS 等平台的标准存储。
4.  微服务业务数据库：各大微服务模块中独立业务数据的持久化存储。

## 总结

MySQL 是一款以
OLTP（联机事务处理）为主、轻量稳定、生态成熟、兼顾性能与易用性的关系型数据库。虽然在极其复杂的分析型查询（OLAP）或海量非结构化数据存储上，通常会配合
ClickHouse、Elasticsearch 或 Redis 等系统使用，但在关系型数据处理领域，它依然是绝大多数项目的首选数据库。

## 1. 安装与布署
MySQL支持多个平台的署安装，如Debian APT源，RedHat的YUM源，还可以使用通用的二进制安装[MySQL多平台布署安装](https://dev.mysql.com/downloads/),这里使用更灵活的Linux通用二进制安装，版本选择MySQL社区版8.4.11 LTS。
### 1.1. 下载二进制压缩归档
官方提供的二进制包主机包含两种安装类型；一种是包含二进制调试文件，另一种是不带二进制调试文件，后者文件大小比前者小很多，安装包带有**minimal**关键字。本文选择最小化安装版本[minimal](https://dev.mysql.com/downloads/mysql/)。
### 1.2. 解压安装
MySQL依赖libaio库，如果本地没有安装该库，在后续初始化数据库目录和服务启动时会报错，本文所有关于MySQL的安装和布署步骤都在RockyLinux9.8上且用户为root下执行。
- 安装`libaio`库
```bash
dnf -y install libaio
```
- 创建`mysql`用户组
```bash
groupadd mysql
```
- 创建`mysql`用户
```bash
useradd -r -g mysql -s /bin/false mysql
```
- 解压归档包安装
```bash
tar xvf mysql-8.4.11-linux-glibc2.28-x86_64-minimal.tar.xz
```
## 2. 配置文件
MySQL服务启动时将读取配置文件内容选项，如果不指定配置文件，`mysqld`服务将会使用默认的选项。
```bash
[mysqld]
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
port=3306
log-error=/usr/local/mysql/data/localhost.localdomain.err
user=mysql
```
> [!NOTE]
> `datadir`: MySQL数据目录。
> `socket`: MySQL网络socks文件。
> `port`: 指定MySQL监听端口。
> `log-error`: 错误日志文件。
> `user`: 运行MySQL服务的用户。