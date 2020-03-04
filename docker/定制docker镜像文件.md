## 现在以定制php-fpm镜像为例子,加入supervisor和系统任务调度cron

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
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY ./crontab /var/spool/cron/crontabs/root
RUN chmod 0644 /var/spool/cron/crontabs/root
RUN crontab /var/spool/cron/crontabs/root

RUN chmod 777 /var/run
RUN chmod 777 /etc/supervisor

COPY ./entrypoint.sh /usr/local/bin/
RUN chmod 777 /usr/local/bin/entrypoint.sh
CMD ["entrypoint.sh"]
```
 ### 5. 创建系统任务调试crontab文件,并增加内容
```
#touch crontab
```
内容
```
#!/usr/bin/env bash
* * * * * /usr/local/bin/php /www/artisan schedule:run >> /dev/null 2>&1
```
 ### 6. 创建入口执行命令脚本 entrypoint.sh，并增加内容
```
#touch entrypoint.sh
```
内容
```
#!/bin/bash
set -e
cron
supervisord
supervisorctl update
supervisorctl reload
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
#docker run --name php7-fpm --restart=always --privileged=true -e TZ="Asia/Shanghai" -v /docker/supervisor/config:/etc/supervisor/conf.d -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/bin/docker -v /docker/www/broker:/www -v /docker/php/php.ini:/usr/local/etc/php/php.ini:ro -v /docker/php/www.conf:/usr/local/etc/php-fpm.d/www.conf:ro -v /docker/php/logs/php.log:/var/log/php.log:rw -v /docker/php/php-fpm.conf:/usr/local/etc/php-fpm.conf:ro -d php-fpm-mmy php-fpm
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
#docker exec -it 0f97fabb71b1 crontab -l
```
回显内容
```
#!/usr/bin/env bash
* * * * * /usr/local/bin/php /www/artisan schedule:run >> /dev/null 2>&1
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