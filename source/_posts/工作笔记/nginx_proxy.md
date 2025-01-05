---
title: nginx实战笔记：配置反向代理访问监控系统
date: 2019-01-24
categories:
  - 工作笔记
tags:
  - Nginx
---
# 需求背景

1、测试环境部署了一套监控系统，测试人员经常需要访问web去分析数据或定位问题，但是访问不方便也不灵活,每次需要去找到pod运行主机IP和svc暴露的端口。
2、通过利用nginx反向代理+负载均衡配置，快速地访问系统。比如prometheus、granfana等

# 配置步骤

1、查看测试集群监控服务的pod和svc信息，prometheus和grafana均以NodePort的形式映射到主机端口。

```sh
|kubernetes-admin@q4-default# kcc get po,svc |grep grafana
pod/grafana-6c5f687b5b-79qf8                         1/1       Running     0          56m

service/grafana                         NodePort    10.104.106.121   <none>        3000:31962/TCP      12d
|kubernetes-admin@q4-default# kca get po,svc |grep prometheus
pod/prometheus-68d7459bb-cmnmn                  5/5       Running   0          2d

service/prometheus                 NodePort    10.99.169.156   <none>        81:32001/TCP,9093:32003/TCP   2d
```



这时是可以通过集群内主机内网ip+端口，成功访问到监控系统。比如访问prometheus端口32001

```sh
[root@harbor nginx-proxy]# curl 192.168.0.34:32001
<a href="/graph">Found</a>.

[root@harbor nginx-proxy]# curl 192.168.0.35:32001
<a href="/graph">Found</a>.

[root@harbor nginx-proxy]# curl 192.168.0.36:32001
<a href="/graph">Found</a>.
```



2、nginx配置文件如下

```nginx
[root@harbor nginx-proxy]# cat default.conf
    # 设定实际的后端服务器列表
    upstream prometheus_server{
	    #设定负载均衡的服务器列表，weigth参数表示权值（缺省是1），权值越高被分配到的几率越大
        server 192.168.0.34:32001 weight=1;
        server 192.168.0.35:32001 weight=2;
        server 192.168.0.36:32001 weight=3;
    }

    upstream alert_server{
        server 192.168.0.34:32003;
        server 192.168.0.35:32003;
        server 192.168.0.36:32003;
    }
    upstream grafana_server{
        server  192.168.0.34:31962;
        #server 192.168.0.35:31962;
        #server 192.168.0.36:31962;
    }


    server {
        #监听80端口，用于HTTP协议
        listen       80;
        #定义使用www.test.com访问
        server_name  www.test.com;
        proxy_headers_hash_max_size     51200;
        proxy_headers_hash_bucket_size  6400;
        proxy_set_header                Host             $host;
        proxy_set_header                X-Real-IP        $remote_addr;
        proxy_set_header                X-Forwarded-For  $proxy_add_x_forwarded_for;

        #location / {
        #    root /usr/share/nginx/html;
        #    index index.html;
        #}
        #反向代理的路径
        location / {
            #请求将转向upstream里prometheus_server定义的服务器列表
            proxy_pass  http://prometheus_server/;
            #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_connect_timeout     30s;
        }
        location /alert/ {
            proxy_pass  http://alert_server/;
            proxy_connect_timeout     30s;
        }
        location /grafana/ {
            proxy_pass  http://grafana_server/;
            proxy_connect_timeout     30s;
        }
    }
```



3、make run快速启动nginx容器
Makefile里总共三步操作：
快速启动nginx容器，映射到主机端口8081，
拷贝default.conf到nginx容器/etc/nginx/conf.d/目录
重启nginx容器

```sh
[root@harbor nginx-proxy]# ll
总用量 12
-rw-r--r-- 1 root root 1587 1月  24 21:39 default_backup.conf
-rw-r--r-- 1 root root 1327 1月  24 21:40 default.conf
-rw-r--r-- 1 root root  353 1月  24 21:44 Makefile
[root@harbor nginx-proxy]# make run
docker rm -f nginx-web
nginx-web
docker run -d -p 8081:80 --name nginx-web nginx
229484284a78d84800527337eb7a074d42ff62987cb8e479a0d6c4c65b586156
docker cp default.conf nginx-web:/etc/nginx/conf.d/
docker restart nginx-web
nginx-web
[root@harbor nginx-proxy]# docker ps |grep nginx-web
229484284a78        nginx                                                                 "nginx -g 'daemon ..."   30 seconds ago      Up 29 seconds          0.0.0.0:8081->80/tcp                                               nginx-web
```

4、在浏览器中访问 [www.test.com，这时应该就可以根据location，访问到不同的服务了。](http://www.test.com，这时应该就可以根据location，访问到不同的服务了。)
访问Grafana Dashboard：
![7.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/linux/nginx15.png?raw=true)
访问Alertmanager ui：
![7.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/linux/nginx16.png?raw=true)
访问:Prometheus ui
![7.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/linux/nginx17.png?raw=true)

# 关于负载均衡策略

Nginx提供了多种负载均衡策略，在上面其实已用到过轮询算法。在这里延伸多了解一下其它策略。

## 轮询

```nginx
upstream bck_testing_01 {
  # 默认所有服务器权重为 1
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

## 加权轮询

```nginx
upstream bck_testing_01 {
  server 192.168.250.220:8080  weight=3
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

## 最少连接

```nginx
upstream bck_testing_01 {
  least_conn;
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

## IP Hash

```nginx
upstream bck_testing_01 {
  ip_hash;
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

参考文档：
https://www.w3cschool.cn/nginx/nginx-d1aw28wa.html
https://mp.weixin.qq.com/s/qMtJtZ6g62wibRVilIHPNg