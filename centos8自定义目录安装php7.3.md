### centos8自定义目录安装php7.3
1. 目录结构

源码目录：/home/werben/pkgsrc/php-7.3.11

安装目录：/home/werben/application/php7.3.11

2. 下载php源码
```
# 官网地址：https://www.php.net/downloads.php
wget https://www.php.net/distributions/php-7.3.11.tar.bz2
```
3. 解压源码
```
tar --bzip -xvf php-7.3.11.tar.bz2 php-7.3.11

```
4. 安装编译工具和库
```
yum install -y gcc gcc-c++
yum -y install libxml2-devel openssl-devel curl-devel libjpeg-devel libpng-devel libicu-devel freetype-devel openldap-devel openldap openldap-devel
```
5. 配置编译参数
```
#创建用户组和用户
groupadd www
useradd -g www www
#配置fpm的用户组和用户，以及安装其他扩展
./buildconf --force
./configure --prefix=/home/werben/application/php7.3.11 \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--enable-mysqlnd-compression-support \
--with-iconv-dir \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib \
--with-libxml-dir \
--enable-xml \
--disable-rpath \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--with-curl \
--enable-mbregex \
--enable-mbstring \
--enable-intl \
--enable-ftp \
--with-gd \
--enable-gd-jis-conv \
--with-openssl \
--with-mhash \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-zip \
--enable-soap \
--with-gettext \
--disable-fileinfo \
--enable-opcache \
--with-pear \
--enable-maintainer-zts \
--with-ldap=shared \
--without-gdbm


#上面的步骤可能会出现很多问题，如需要重新安装libzip，需要安装ldap，需要安装cmake

#重新安装libzip需要安装cmake,这里记录一下cmake的安装步骤,其他问题自己百度解决了，
#不记录了，中间自己去官网下了几个最新的cmake版本，编译过程中都出错了。
#发现宝塔用的是2.8.X的版本。这里我用的版本是3.5.2的版本

wget https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz
tar xvf cmake-3.5.2.tar.gz
cd cmake-3.5.2
./bootstrap --prefix=/usr/local/cmake
gmake
gmake install

cd /usr/local/cmake/bin
ln -s /usr/local/cmake/bin/cmake /usr/bin/
cmake --version

#接下来安装libzip
wget https://libzip.org/download/libzip-1.5.2.tar.gz
tar -zxf libzip-1.5.2.tar.gz
cd libzip-1.5.2
mkdir build
cd build 
cmake ..
make -j4
make install
```
6. 安装make工具
```
#如果提示make命令找不到，则才需要安装make工具
yum -y install gcc automake autoconf libtool make bison
```
7. make && make install
8. 映射全局命令
```
ln -s /home/werben/application/php7.3.11/sbin/* /usr/local/sbin/
ln -s /home/werben/application/php7.3.11/bin/* /usr/local/bin/
```
9. 配置php.ini
```
#查看php.ini的位置
php -r "phpinfo();" | grep 'php.ini'

#将源码中的php.ini*拷贝到php.ini的位置
cp /home/werben/pkgsrc/php-7.3.11/php.ini-* /home/werben/application/php7.3.11/lib/

#重命名php.ini文件
cp /home/werben/application/php7.3.11/lib/php.ini-production /home/werben/application/php7.3.11/lib/php.ini
```
10. 安装目录结构
```
#/home/werben/pkgsrc/php-7.3.11安装目录的结构
├── bin
│   ├── pear
│   ├── peardev
│   ├── pecl
│   ├── phar -> phar.phar
│   ├── phar.phar
│   ├── php
│   ├── php-cgi
│   ├── php-config
│   ├── phpdbg
│   └── phpize
├── etc
│   ├── pear.conf
│   ├── php-fpm.conf.default
│   └── php-fpm.d
├── include
│   └── php
├── lib
│   ├── php
│   ├── php.ini
│   ├── php.ini-development
│   └── php.ini-production
├── php
│   ├── man
│   └── php
├── sbin
│   └── php-fpm
└── var
    ├── log
    └── run
```

### 常见问题

1. php安装执行configure报错error: off_t undefined; check your library configuration
```
vim /etc/ld.so.conf 
#添加如下几行
/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64 
#保存退出
:wq
ldconfig -v # 使之生效
```
2. configure: error: Cannot find ldap libraries in /usr/lib 解决办法
```
cp -frp /usr/lib64/libldap* /usr/lib/
```