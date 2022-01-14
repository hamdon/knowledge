
个性化域名绑定

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
  
  私有域名绑定
对于实现用户私有域名绑定的情况，如果系统的用户数量不多的话，也完全可以通过在nginx中手动配置来一一绑定实现；但对于一个多用户的云平台来说，通过直接修改配置文件来绑定用户私有域名，这维护难度也太大了！

有没有一种简单又灵活的方式，来实现私有域名绑定呢？答案肯定是有的。

通过OpenResty来实现私有域名绑定



