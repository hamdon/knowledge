
### 个性化域名绑定

常见SaaS平台应用都会给用户分配一个「二级域名」，也就是常说的「个性化域名」。个性化域名的绑定实现，利用域名泛解析和服务器nginx URL重写rewrite ，即可将URL重定向到实际的URL。

如：

访问的URL ：http://ming.uedst.com/

实际的URL ：http://www.uedst.com/space/ming


nginx.conf 配置如下：
```
server {
    listen    80;
    server_name *.uedst.com;
    if ( $host ~* (\b(?!www\b)\w+)\.\w+\.\w+ ) {
      set $userdomain $1;
    }
    location / {
      rewrite ^/$ /space/$userdomain last;
      proxy_pass http://www.uedst.com/;
    }
  }
  ```
  
### 私有域名绑定
对于实现用户私有域名绑定的情况，如果系统的用户数量不多的话，也完全可以通过在nginx中手动配置来一一绑定实现；但对于一个多用户的云平台来说，通过直接修改配置文件来绑定用户私有域名，这维护难度也太大了！

有没有一种简单又灵活的方式，来实现私有域名绑定呢？答案肯定是有的。

通过OpenResty来实现私有域名绑定
![实现私有域名绑定](https://raw.githubusercontent.com/hamdon/knowledge/master/image/2.jpg)



#### 只需三步
1、系统平台域名做泛解析「 *.http://uedst.com」, 给用户分配「个性化域名http://u2.web.uedst.com」

2、系统平台将用户「私有域名http://www.duiler.com」与用户「个性化域名http://u2.web.uedst.com」对应关系保存到redis缓存中
```
$ set www.duiler.com  u2.web.uedst.com
```
3、用户将「私有域名http://www.duiler.com」做 cname别名解析指向到系统平台分配自己的「个性化域名http://u2.web.uedst.com」上

nginx.conf 配置
```
server {
        # 域名指向都不匹配时，会选择此配置，对于泛解析的域名，可以在这里做一些处理
        listen 80 default_server;
        server_name _;
        resolver 8.8.8.8;
        location / {            
            lua_code_cache off;
            #设置$userdomain变量及默认值
            set $userdomain default;
            #引入lua文件
            access_by_lua_file /lua/cname.lua;
            #反向代理
            proxy_pass http://$userdomain;
            proxy_set_header Host $userdomain;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
    server {
        #匹配系统分配的二级域名
        listen 80;
        server_name  *.web.uedst.com;
        location / {
            default_type text/html;
            content_by_lua_block {
                #可看到实际访问的域名
                ngx.say("hello:", ngx.var.host);
                #ngx.say(ngx.var.document_uri);
            }
        }
    }
```
cname.lua文件
参考官方demo https://github.com/openresty/lua-resty-redis
```
local redis = require "resty.redis"
local red = redis:new()

red:set_timeout(1000)

local ok, err = red:connect("127.0.0.1", 6379)
if not ok then
ngx.say("failed to connect: ", err)
return
end

local host = ngx.var.host —获取访问的域名
local res, err = red:get(host) —查找对应的cname记录
if res then
ngx.var.userdomain = res —修改nginx变量
return
end

local ok, err = red:set_keepalive(10000, 100)
if not ok then
ngx.say("failed to set keepalive: ", err)
return
end
```

[OpenResty官网 ] http://openresty.org/cn/



[OpenResty安装 参考 ]http://openresty.org/cn/installation.html


[OpenResty教程 参考 ]https://moonbingbing.gitbooks.io/openresty-best-practices/content/


[本文引用出处] https://www.runoob.com/markdown/md-link.html





