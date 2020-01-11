### CentOS 7 安装 Mysql 8.0 教程

1. #### 安装Mysql 8.0


1）  配置Mysql 8.0安装源：
#sudo rpm -Uvh https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
2）安装Mysql 8.0
#sudo yum --enablerepo=mysql80-community install mysql-community-server
安装步骤：
        解析依赖 --> 输入第一个【y】同意下载 --> 输入第二个【y】同意安装 --> 至此Mysql 8.0安装完成。

3） 启动Mysql服务
#sudo service mysqld start
 该服务需要root权限启动

4） 查看Mysql服务状态
#service mysqld status

5） 查看root用户临时密码
#grep "A temporary password" /var/log/mysqld.log

6） 配置Mysql安全策略
#mysql_secure_installation

第一步：设置新的（Mysql中的）root用户密码（需由大写、小写、数字、符号四种混合组成）

第二步：配置是否启用密码安全性检查插件，保证密码强度，按需启用。建议【y】

第三步：选择一种密码强度，0【LOW】是长度八位以上；1【MEDIUM】是长度八位以上，而且由数字、大小写、符号组成；2【STRONG】是长度八位以上，而且由数字、大小写、符号组成，并通过字典文件检测，按需选择。建议【2】

第四步：系统自动检测root用户的密码强度，如分数过低可以输入【y】进行更改密码，否则输入【n】跳过

第五步：选择是否删除匿名用户。建议【y】第五步：选择是否删除匿名用户。建议【y】

第六步：选择是否运行root用户远程连接。建议【n】可根据下文添加另一远程用户

第七步：选择是否删除测试数据库。建议【y】

第八步：选择是否刷新privilege表，即是否执行flush privileges命令。建议【y】
 到此安全策略配置完成。
y

2. #### 配置远程访问

1）登录mysql控制台
#mysql -uroot -p
3）授权给远程用户
#GRANT ALL ON *.* TO '[用户名]'@'%';（ALL表示授予所有权限、*.*表示所有数据库中的所有表、%表示任意IP可以远程连接）
    其他权限：ALTER、ALTER ROUTINE、CREATE、CREATE ROUTINE、CREATE TABLESPACE、CREATE TEMPORARY TABLES、CREATE USER、CREATE VIEW、DELETE、DROP、EVENT、EXECUTE、FILE、GRANT OPTION、INDEX、INSERT、LOCK TABLES、PROCESS、PROXY、REFERENCES、RELOAD、REPLICATION CLIENT、REPLICATION SLAVE、SELECT、SHOW DATABASES、SHOW VIEW、SHUTDOWN、SUPER、TRIGGER、UPDATE、USAGE。
    例如GRANT INSERT,SELECT,UPDATE ON *.* TO '[用户名]'@'%';

4）使用navicat连接

    连接时将会出现如下错误："2059 - authentication plugin 'caching_sha2_password' cannot be loaded: 乱码"
错误原因，Mysql 8.0的新特性，旧版本Navicat不支持。

    解决方案两种：

        ①以旧版的方式重新设置远程用户的密码。

#ALTER USER '[用户名]'@'%' IDENTIFIED WITH mysql_native_password BY '[密码]';
 ②给Navicat更新驱动，暂不推荐此方案。
