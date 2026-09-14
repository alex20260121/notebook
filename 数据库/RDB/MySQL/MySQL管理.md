# MySQL管理
MySQL管理涵盖了从安装到配置，日常运维到性能调优与高可用架构的管理与维护。

## 1. 用户与权限管理
安全第一道防线，合理的权限控制遵循“最小权限原则”。

### 1.1. 用户与权限管理
合理的权限控制遵循“最小权限原则”。

- 创建用户与设置密码(创建允许特定 IP（或网段）访问的用户)：
```sql
CREATE USER 'app_user'@'192.168.1.%' IDENTIFIED BY 'StrongPassword123!';
```

- 授权与撤销权限：
```sql
-- 授予特定数据库的增删改查权限
GRANT SELECT, INSERT, UPDATE, DELETE ON app_db.* TO 'app_user'@'192.168.1.%';

-- 撤销删除权限
REVOKE DELETE ON app_db.* FROM 'app_user'@'192.168.1.%';

-- 刷新权限使其生效
FLUSH PRIVILEGES;
```
- 授于最高权限(相当于`root`)
```sql
-- 查看所有用户及其允许的 host
SELECT user, host FROM mysql.user WHERE user = '你的用户名';

-- 1. 赋予所有库表的所有权限，并允许其转授权限
GRANT ALL PRIVILEGES ON *.* TO 'devuser'@'%' WITH GRANT OPTION;

-- 2. 刷新权限使其立即生效
FLUSH PRIVILEGES;
```
> [!TIP]
> 核心参数说明：
>   - ON *.*：表示对所有数据库、所有数据表生效。
>   - WITH GRANT OPTION：允许该用户像 root 一样，给其他用户创建账号并授权（非常关键，不加则无法管理其他用户）。

- 查看与删除用户
```sql
-- 查看用户
SHOW GRANTS FOR 'app_user'@'192.168.1.%';

-- 删除用户
DROP USER 'app_user'@'192.168.1.%';
```
