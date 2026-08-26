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
