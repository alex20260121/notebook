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
- 重启`MySQL`服务让配置生效
```bash
systemctl restart mysqld.service
```
# 环境变量
`validate_password`组件公开了一组系统变量，可以自定义MySQL的密策略，当密码以明文的方式出现在`sql`语句中时，该组件会检查密码是否满足当前密码策略，如果强度不足则会拒绝该密码。
- `validate_password.policy=1`
设置密码强度策略，默认为1，有效值范围：0、1、2对应LOW、MEDIUM、STRONG。
- `validate_password.length=8`
设置密码字符最小长度，取值类型为整数。
- `validate_password.number_count=1`
设置的密码中，必须至少包含 1 个数字字符（0-9）。**只有当`validate_password.policy≥2`时才会生效。**
- `validate_password.mixed_case_count=1`
规定密码中必须包含的大小写字母的最小数量,这里`=1`必须要1个大写字母+1个小写字母，**只有当`validate_password.policy≥2`时才会生效。**
