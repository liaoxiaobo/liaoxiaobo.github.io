---
title: nginx实战笔记：常用功能
date: 2019-01-10
categories:
  - 工作笔记
tags:
  - Nginx
---

# Nginx常用功能

## http正向代理

正向代理是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。
当然前提是客户端要知道正向代理服务器的 IP 地址，还有代理程序的端口。

正向代理的用途：
（1）访问原来无法访问的资源，如google
（2） 可以做缓存，加速访问资源
（3）对客户端访问授权，上网进行认证
（4）代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息

![1.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/master/img/linux/nginx1.png?raw=true)

## 反向代理

反向代理（Reverse Proxy）实际运行方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。
反向代理的作用：
（1）保证内网的安全，可以使用反向代理提供WAF功能，阻止web攻击
（2）负载均衡，通过反向代理服务器来优化网站的负载
![2.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/master/img/linux/nginx2.png?raw=true)
两者区别，看图说话：
![3.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/master/img/linux/nginx3.png?raw=true)

## 负载均衡

### 基本概念：

这里提到的客户端发送的、Nginx 反向代理服务器接收到的请求数量，就是我们说的负载量。
请求数量按照一定的规则进行分发，到不同的服务器处理的规则，就是一种均衡规则。
所以将服务器接收到的请求按照规则分发的过程，称为负载均衡。

### nginx负载均衡算法

Nginx提供的负载均衡策略有2种：内置策略和扩展策略。内置策略为轮询，加权轮询，Ip hash。
扩展策略，就天马行空，只有你想不到的没有他做不到的啦，你可以参照所有的负载均衡算法，给他一一找出来做下实现。
上2个图，理解这三种负载均衡算法的实现
![4.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/master/img/linux/nginx4.png?raw=true)

![5.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/master/img/linux/nginx5.png?raw=true)
Ip hash算法，对客户端请求的ip进行hash操作，然后根据hash结果将同一个客户端ip的请求分发给同一台服务器进行处理，可以解决session不共享的问题。

# Nginx配置文件详解

Nginx配置文件主要分成四部分：main（全局设置）、server（主机设置）、upstream（上游服务器设置，主要为反向代理、负载均衡相关配置）和location（URL匹配特定位置后的设置），每部分包含若干个指令。

他们之间的关系式：server继承main，location继承server；upstream既不会继承指令也不会被继承。它有自己的特殊指令，不需要在其他地方的应用

main部分设置的指令将影响其它所有部分的设置；server部分的指令主要用于指定虚拟主机域名、IP和端口；upstream的指令用于设置一系列的后端服务器，设置反向代理及后端服务器的负载均衡；location部分用于匹配网页位置（比如，根目录”/“,”/images”,等等）。

> docker run –name nginx-test -p 8080:80 -d nginx
> 使用docker命令直接安装启动nginx

```sh
# 进入容器内查看默认的配置文件,均在/etc/nginx/路径下。
localhost:~ liaoxb$ docker exec -ti 2ccbf7fa358e sh
# find / -name nginx.conf
/etc/nginx/nginx.conf
# find / -name default.conf
/etc/nginx/conf.d/default.conf
```

默认的 nginx 配置文件 nginx.conf 内容如下：

```nginx
root@2ccbf7fa358e:/# cat /etc/nginx/nginx.conf
user  nginx;    #定义Nginx运行的用户和用户组

worker_processes  1;    #nginx进程数，建议设置为等于CPU总核心数

error_log  /var/log/nginx/error.log warn;   #全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]

pid        /var/run/nginx.pid;  #进程pid文件



events {
    worker_connections  1024;       #单个进程最大连接数（最大连接数=连接数*进程数）
}

#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
    include       /etc/nginx/mime.types;        #文件扩展名与文件类型映射表
    default_type  application/octet-stream;     #默认文件类型

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    
#开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    sendfile        on;
#此选项允许或禁止使用socke的TCP_CORK的选项，此选项仅在使用sendfile的时候使用
    #tcp_nopush     on;

    keepalive_timeout  65;      #长连接超时时间，单位是秒

    #gzip  on;      #gzip模块设置

    include /etc/nginx/conf.d/*.conf;   #增加nginx虚拟主机配置文件(conf.d)
}
```



默认的nginx虚拟主机配置文件如下（反向代理到nginx的默认html页面，位于/usr/share/nginx/html/index.html）：

```nginx
root@2ccbf7fa358e:/# cat /etc/nginx/conf.d/default.conf
#虚拟主机的配置
server {
    listen       80;    #监听端口
    server_name  localhost;     #域名可以有多个，用空格隔开

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
#对 "/" 启用反向代理
    location / {
        root   /usr/share/nginx/html;   #根目录
        index  index.html index.htm;    #设置默认页
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;     #拒绝的ip
    #}
}
```



参考文档：
https://www.runoob.com/w3cnote/nginx-setup-intro.html
https://www.runoob.com/w3cnote/nginx-setup-intro.html
https://www.w3cschool.cn/nginx/nginx-d1aw28wa.html