# 安装MySQL密码验证组件
MySQL密码验证组件`validate_password`用于测试用户密码强度以提高安全性。
- 确保`validate_password`组件库文件位于`MySQL`插件目录中`lib/plugin/component_validate_password.so`
- 确保己将`plugin_dir`插件目录设置为服务器地址`mysql -e 'SELECT @@plugin_dir' -u root -p`
```text
+-------------------------------+
| @@plugin_dir                  |
+-------------------------------+
| /usr/local/mysql8/lib/plugin/ |
+-------------------------------+
```
- 使用`INSTALL COMPONENT`安装`validate_password`密码验证组件;
```bash
INSTALL COMPONENT 'file://component_validate_password';
```
- 将以下这些选项添加到配置文件`[mysqld]`选项组内，可以根据需要调整，以下选项都是默认值。
```bash
validate_password.policy=1
validate_password.length=8
validate_password.number_count=1
validate_password.mixed_case_count=1
validate_password.special_char_count=1
validate_password.check_user_name=1
```
- 验证组件安装情况`mysql -e 'select * from mysql.component' -u root -p`
```text
+--------------+--------------------+------------------------------------+
| component_id | component_group_id | component_urn                      |
+--------------+--------------------+------------------------------------+
|            1 |                  1 | file://component_validate_password |
+--------------+--------------------+------------------------------------+
```
