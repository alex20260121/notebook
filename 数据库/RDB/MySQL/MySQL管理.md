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

- 查看与删除用户
```sql
-- 查看用户
SHOW GRANTS FOR 'app_user'@'192.168.1.%';

-- 删除用户
DROP USER 'app_user'@'192.168.1.%';
```
