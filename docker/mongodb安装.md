## Docker系列 - CentOS使用外部配置文件部署MongoDB，并挂载主机目录

创建文件目录用于挂载mongodb数据和配置

在centos中执行如下命令

```
cd /home/
mkdir docker
cd docker
mkdir mongo
cd mongo
mkdir data logs conf
chmod 777 data
touch logs/mongod.log
chmod 777 logs/mongod.log
vim conf/mongod.conf
```

添加如下配置

```
# 数据库文件存储位置
dbpath = /data/db
# log文件存储位置
logpath = /data/log/mongod.log
# 使用追加的方式写日志
logappend = true
# 是否以守护进程方式运行
# fork = true
# 全部ip可以访问
bind_ip = 0.0.0.0
# 端口号
port = 27017
# 是否启用认证
auth = true
# 设置oplog的大小(MB)
oplogSize=2048
```

Docker Hub上关于mongo镜像的详细说明。
https://hub.docker.com/_/mongo/


启动mongodb容器

```
docker run -itd --name mongodb --restart=always --privileged -p 27017:27017 -v /home/docker/mongo/data:/data/db -v /home/docker/mongo/conf:/data/configdb -v /home/docker/mongo/logs:/data/log/ mongo:latest -f /data/configdb/mongod.conf

# --restart=always Docker服务重启容器也启动
# --privileged 拥有真正的root权限
# -f 指定配置文件

```
查看启动的容器并进入容器

```
docker container ps -a

docker exec -it mongodb bash
mongo
use admin
```
 创建管理员账号

```
db.createUser({user:'root',pwd:'1qazxsw2',roles:['root']})

db.auth('root','1qazxsw2')

db.updateUser("apiAdmin",{roles : [{"role" : "dbAdmin","db" : "leo-api-auto-db"},{"role" : "readWrite","db" : "leo-api-auto-db"}]})
use leo-api-auto-db
```
一定要记得添加角色 readWrite, 不然会没有权限读写数据库

```
db.createUser({user:'admin',pwd:'password',roles:[{role:'dbAdmin',db:'leo-api-auto-db'},'readWrite']})

```

来源：[单击查看](https://www.cnblogs.com/vincent-li666/p/12763723.html)


安装方法：[单击查看](https://www.runoob.com/docker/docker-install-mongodb.html)








