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

## 1. 安装MySQL二进制包
在安装二进制MySQL包时，有些系统可能并没安装相关的依赖库，需要手动提前解决相关依赖关系，比如本文在RockyLinux9.8环境少了`libaio`库。
```bash
dnf -y install libaio
```
### 1.1. 创建MySQL用户和组
创建运行`MySQL`服务的用户和组，并且设置好相关的目录的所属。
- 创建用户组
```bash
groupadd mysql
```
- 创建用户
```bash
useradd -r -g mysql -s /bin/false mysql
```
### 1.2. 解压缩安装包
这一步将下载的通用二进制压缩包进行解压。
```bash
tar xvf mysql-8.4.11-linux-glibc2.28-x86_64-minimal.tar.xz -C /usr/local/
```
> [!TIP]
> 
> 如果不想在执行`mysql`命令时输入完整的绝对路径，可以将`/usr/local/mysql-8.4.11-linux-glibc2.28-x86_64-minimal/bin`目录作为系统环境变量导出。
>
> `echo "export PATH=$PATH:/usr/local/mysql-8.4.11-linux-glibc2.28-x86_64-minimal/bin" >> ~/.bashrc`

## 2. 配置
安装后设置包括创建用于导入和导出操作的安全目录、配置服务器启动选项、初始化数据目录、使用 systemd 启动 MySQL、重置 MySQL root@localhost用户帐户密码，以及运行一些测试以确保服务器正常工作。
### 2.1. 创建用于导入和导出操作的安全目录
创建 mysql-files 目录的主要作用是：作为文件导入/导出的安全沙箱目录，用于防止恶意的文件读写操作。它直接与 MySQL 的安全配置参数 secure_file_priv 绑定。
```bash
mkdir mysql-files
chown mysql:mysql mysql-files
chmod 750 mysql-files
```
> [!NOTE]
> 当`MySQL`涉及本地文件读写操作时，MySQL会强制执行路径检查，如果 secure_file_priv 被配置为指向 mysql-files 目录（例如 /var/lib/mysql-files 或 /usr/local/mysql/mysql-files），那么涉及本地文件的所有操作只能在 mysql-files 目录下进行。如果尝试读写其他目录（如 /etc/、/var/www/ 等），MySQL 会直接拒绝并报错。
### 2.2. 配置文件
配置文件可以指定`mysqld`启动时的参数选项，如果不指定配置文件或参数选项时，`mysqld`服务启动时会使用默认参数。
```bash
[mysqld]
datadir=/usr/local/mysql8/data
socket=/tmp/mysql.sock
port=3306
user=mysql
log-error=/usr/local/mysql/data/localhost.localdomain.err
secure_file_priv=/usr/local/mysql8/mysql-files
```
> [!TIP]
> 为了安全考虑，可以将该配置文件属主设为`root`用户，并且其它用户和组只有只读权限。
>
> `chown root:root mysql.cnf && chmod 644 mysql.cnf`

### 2.3. 初始化数据目录
安装 MySQL 后，必须初始化数据目录，其中包含mysql系统数据库及其表，包括授权表、服务器端帮助表和时区表。初始化过程还会创建 root@localhost超级用户帐户、 InnoDB系统表空间以及管理InnoDB表所需的其他数据结构。
```bash
mysqld --defaults-file=mysql.cnf --initialize
```
> [!NOTE]
> 初始化过程中会生成随机密码，将密码保存下来，在后续的过程中将会重置`root`密码。

### 2.4. 配置`systemd`服务
