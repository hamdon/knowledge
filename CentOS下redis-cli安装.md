### Step1：安装gcc wget等系统lib

```
yum install -y gcc wget
```

### Step2：获取redis稳定版并解压
```
cd /tmp
wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable
```
### Step3：编译
```
make
```

### 其他错误：

如果报错：jemalloc/jemalloc.h：没有那个文件或目录
可以尝试如下编译
```
make MALLOC=libc

```
```
报错原因：
关于分配器allocator， 如果有MALLOC这个环境变量，会有用这个环境变量去建立Redis。
libc并不是默认的分配器， 默认的是 jemalloc, 因为 jemalloc 被证明比libc有更少的 fragmentation problems。
但是如果你又没有jemalloc 而只有 libc 当然 make 出错。 所以加这么一个参数。
```
### Step4：全局使用redis-cli
```
cp src/redis-cli /usr/bin/
```


