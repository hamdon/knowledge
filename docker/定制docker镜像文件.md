## 现在以定制php-fpm镜像为例子,加入supervisor和系统任务调度cron

### 首先要更新docker的镜像地址到阿里云，或者国内某个有名的地址。
[单击查看](https://cr.console.aliyun.com/cn-zhangjiakou/instances/mirrors)

 Ubuntu 和CentOs的配置
1. 安装／升级Docker客户端
推荐安装1.10.0以上版本的Docker客户端，参考文档 docker-ce

2. 配置镜像加速器
针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://dttvopmr.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 1. 下载要引用的镜像

```
 #docker pull crunchgeek/php-fpm:7.2
```
 ### 2. 查看已经安装镜像
```
#docker images
crunchgeek/php-fpm   7.2                 929d1ee5af21        12 months ago       1.02GB
```
### 3. 新建一个目录,并进到这个目录
```
#mkdir php
#cd php
```
### 4. 创建Dockerfile文件,并增加内容
```
#touch Dockerfile
```
内容
```
FROM crunchgeek/php-fpm:7.2
MAINTAINER Hamdon "cao4141@qq.com"
RUN apt-get install cron -y \
 && apt-get install supervisor \
 && apt-get autoremove \
 && apt-get autoclean \
 && apt-get clean \
 && pecl install imagick \
 && docker-php-ext-enable imagick \
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY ./crontab /etc/crontab 
RUN chmod 600 /etc/crontab

RUN chmod 777 /var/run
RUN chmod 777 /etc/supervisor

COPY ./entrypoint.sh /usr/local/bin/
RUN chmod 777 /usr/local/bin/entrypoint.sh
CMD ["entrypoint.sh"]
```
来一份带mongodb 和 ffmpeg的
```
FROM crunchgeek/php-fpm:7.2
MAINTAINER Hamdon "cao4141@qq.com"
RUN echo "deb [check-valid-until=no] http://archive.debian.org/debian jessie-backports main" > /etc/apt/sources.list.d/jessie-backports.list
RUN sed -i '/deb http:\/\/deb.debian.org\/debian jessie-updates main/d' /etc/apt/sources.list
RUN apt-get -o Acquire::Check-Valid-Until=false update
RUN apt-get -y --force-yes install yasm ffmpeg

RUN apt-get install cron -y \
 && apt-get install supervisor \
 && apt-get install -y libmagickwand-dev --no-install-recommends \
 && apt-get autoremove \
 && apt-get autoclean \
 && apt-get clean \
 && pecl install imagick \
 && pecl install mongodb \
 && docker-php-ext-enable imagick \
 && docker-php-ext-enable mongodb \
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY ./crontab /etc/crontab 
RUN chmod 600 /etc/crontab

RUN chmod 777 /var/run
RUN chmod 777 /etc/supervisor

COPY ./entrypoint.sh /usr/local/bin/
RUN chmod 777 /usr/local/bin/entrypoint.sh
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

```
~~里面的第一个链接地址可以修改成这个试试~~(废弃)
```
http://archive.debian.org/debian
改成：
http://mirrors.163.com/debian 
```

 ### 5. 创建系统任务调试crontab文件,并增加内容
```
#touch crontab
#chmod 0600 crontab
```
内容
```
SHELL=/bin/bash
PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
* * * * * root /usr/local/bin/php /www/xxxx/artisan schedule:run >> /dev/null 2>&1
```
 ### 6. 创建入口执行命令脚本 entrypoint.sh，并增加内容
```
#touch entrypoint.sh
```
内容
```
#!/bin/bash
set -e
/etc/init.d/cron start
/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
exec "$@"
```
### 7. 目录结构
```
[root@localhost php]# tree
.
├── crontab
├── Dockerfile
└── entrypoint.sh

0 directories, 3 files
```

### 8. 构建镜像
```
#docker build -t php-fpm-mmy .
```
### 9. 查看已经安装镜像
```
#docker images
crunchgeek/php-fpm   7.2                 929d1ee5af21        12 months ago       1.02GB
php-fpm-mmy          latest              ca5e9f3d0b8a        18 hours ago        1.02GB
```
### 10. 执行启用容器命令
```
#docker run --name php7-fpm --restart=always --privileged=true -e TZ="Asia/Shanghai" -v /docker/supervisor/config:/etc/supervisor/conf.d -v /docker/crontab/crontab:/etc/crontab:rw -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/bin/docker -v /docker/www/broker:/www -v /docker/php/php.ini:/usr/local/etc/php/php.ini:ro -v /docker/php/www.conf:/usr/local/etc/php-fpm.d/www.conf:ro -v /docker/php/logs/php.log:/var/log/php.log:rw -v /docker/php/php-fpm.conf:/usr/local/etc/php-fpm.conf:ro -d php-fpm-mmy php-fpm
```
/docker/php 目录结构
```
.
├── logs
│   └── php.log
├── php-fpm.conf
├── php.ini
└── www.conf

2 directories, 3 files

```
/docker/supervisor 目录结构
```
.
└── config
    └── broker-worker.conf

1 directory, 1 file
```
/docker/www/broker 目录为网站实际内容

### 11. 查看运行容器
```
#docker ps -a
0f97fabb71b1        php-fpm-mmy         "docker-php-entrypoi…"   18 hours ago        Up About an hour (healthy)   9000/tcp             php7-fpm
```
### 12. 运行各个测试命令
```
#docker exec -it 0f97fabb71b1 /etc/init.d/cron status
```
回显内容
```
[ ok ] cron is running.
```
```
#docker exec -it 0f97fabb71b1 php-fpm -v
```
显示内容
```
PHP 7.2.15 (fpm-fcgi) (built: Feb  9 2019 02:52:09)
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.15, Copyright (c) 1999-2018, by Zend Technologies
    with Xdebug v2.6.0, Copyright (c) 2002-2018, by Derick Rethans
```
```
docker exec -it 0f97fabb71b1 supervisorctl status
```
显示内容
```
broker-worker:broker-worker_0    RUNNING   pid 9797, uptime 0:00:03
```
如果出现
```
broker-worker:broker-worker_0    FATAL     Exited too quickly (process log may have details)
```
可以重启一下supervisor，执行以下命令
```
docker exec -it 0f97fabb71b1 supervisorctl reload
```
### 13. 导出镜像
```
docker save -o php_fpm_mmy.tar.gz php-fpm-mmy

```
导入
```
docker load -i php_fpm_mmy.tar.gz
```
[单击查看](https://blog.csdn.net/ncdx111/article/details/79878098)
### Docker 中的 PHP 如何安装扩展

[单击查看](https://my.oschina.net/antsky/blog/1591418)

### Docker 挂载文件，宿主机修改后容器里文件没有修改
[单击查看](https://cloud.tencent.com/developer/article/1708294)
[单击查看](https://blog.csdn.net/yanjiee/article/details/119773641)

### linux计划任务踩坑
https://blog.csdn.net/qq_41874930/article/details/112781957

https://blog.csdn.net/u012206617/article/details/106146006/


docker 挂载的文件，要设置权限为：0666时修改宿主机文件，docker容器才会实时同步，但是/etc/crontab如果设置为0666的话，则容器里面的cron不会执行此文件，要在宿主机修改文件的权限为：0600时，docker里面才会执行这个/etc/crontab文件，所以如果docker挂载的文件不是定时执行任务，则设置文件权限为0666(chmod 0666 xxxx),如果是定时任务文件，则设置文件权限为0600(chmod 0600 xxxx),为了让定时任务的文件可以一致，可以修改完宿主机的文件，再进到容器里面修改/etc/crontab文件，然后重启（/etc/init.d/cron restart）,或者修改完宿主机的定时任务文件后，重启docker(docker restat xxxx),或者修改完宿主机的文件后，使用docker cp命令复制文件到docker容器里面（docker cp 本地文件路径 容器ID/容器NAME:容器内路径）

/etc/crontab 文件对应宿主机的文件要给chmod 0600 权限，docker容器里面的定时任务才会执行
```
SHELL=/bin/bash
PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
* * * * * root /usr/local/bin/php /www/xxxx/artisan schedule:run >> /dev/null 2>&1
*/1 * * * * root echo "11" >> /www/a.log
```