### 在CentOS 8上安装MySQL 8.0
通过以root用户或具有sudo特权的用户身份使用CentOS软件包管理器来安装MySQL 8.0服务器：
```
$sudo dnf install @mysql
```
@mysql模块将安装MySQL及其所有依赖项。

安装完成后，通过运行以下命令来启动MySQL服务并使它在启动时自动启动：
```
$sudo systemctl enable --now mysqld
```
要检查MySQL服务器是否正在运行，请输入：
```
$sudo systemctl status mysqld
```
回信息如下：
```
mysqld.service - MySQL 8.0 database server

Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)

Active: active (running) since Thu 2019-10-17 22:09:39 UTC; 15s ago
```
注：说明安装MySQL 8.0成功了。

#### 保护MySQL的操作
运行mysql_secure_installation脚本，该脚本执行一些与安全性相关的操作并设置MySQL根密码：
```
$sudo mysql_secure_installation
```
系统将要求你配置VALIDATE PASSWORD PLUGIN（验证密码插件），该插件用于测试MySQL用户密码的强度并提高安全性，密码验证策略分为三个级别：低、中和强，如果你不想设置验证密码插件，请按Enter。

在下一个提示符下，将要求你设置MySQL root用户的密码，完成此操作后，脚本还将要求你删除匿名用户，限制root用户对本地计算机的访问，并删除测试数据库，你应该对所有问题回答“是”。

要从命令行与MySQL服务器进行交互，请使用MySQL客户端实用程序，它作为依赖项安装，通过键入以下内容测试根访问权限：
```
$ mysql -u root -p
```
在出现提示时输入root密码，然后将显示MySQL shell，如下所示：
```
Welcome to the MySQL monitor.  Commands end with ; or \g.

Your MySQL connection id is 12

Server version: 8.0.17 Source distribution
```
至此，已经在CentOS 8服务器上安装并保护了MySQL 8.0，并准备使用它。

#### 身份验证的操作
由于CentOS 8中的某些客户端工具和库与caching_sha2_password方法不兼容，因此CentOS 8存储库中包含的MySQL 8.0服务器设置为使用旧的mysql_native_password身份验证插件，该方法在上游MySQL 8.0发行版中设置为默认。

对于大多数设置，mysql_native_password方法应该没问题，但是，如果你想将默认身份验证插件更改为caching_sha2_password，这样可以更快并提供更好的安全性，请打开以下配置文件：
```
$sudo vim /etc/my.cnf.d/mysql-default-authentication-plugin.cnf
```
将default_authentication_plugin的值更改为caching_sha2_password：
```
[mysqld]

default_authentication_plugin=caching_sha2_password
```
关闭并保存文件，然后重新启动MySQL服务器以使更改生效：
```
$ sudo systemctl restart mysqld
```

### 结论
CentOS 8随MySQL 8.0一起发行，安装就像键入dnf install @mysql一样简单。

现在你的MySQL 8服务器已启动并正在运行，可以连接到MySQL Shell，并开始创建新的数据库和用户。