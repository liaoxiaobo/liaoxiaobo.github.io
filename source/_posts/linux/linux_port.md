---
title: Linux主机端口相关问题
date: 2017/11/26 16:36:25
categories:
  - Linux
tags:
  - Linux 
---

## 排查端口占用问题

1、安装net-tools命令行工具 yum install -y net-tools
2、执行命令查看被占用端口的pid netstat -anlp | grep port
如下图，发现本机8083端口被22980的程序wisecloud-met所占用

![1.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/linux/port04.png?raw=true)

3、进一步使用命令：ps -aux | grep wisecloud-met，或者直接：ps -aux | grep pid 查看，就可以明确知道8083端口是被哪个程序占用了！然后判断是否使用KILL命令干掉！

![2.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/linux/port05.png?raw=true)

知识拓展：[Linux网络管理常用命令：net-tools VS iproute2](https://www.cnblogs.com/wonux/p/6268134.html)



## 测试端口连通性

### 方法一：telnet

预置条件：安装telnet
step 1、rpm -qa telnet-server（无输出表示telnet-server未安装，则执行step2；否则执行step3）
step 2、yum -y install telnet-server（安装telnet-server）
step 3、rpm -qa telnet（无输出表示telnet未安装，则执行step4，否则执行step5）
step 4、yum -y install telnet（安装）

telnet为用户提供了在本地计算机上完成远程主机工作的能力，因此可以通过telnet来测试端口的连通性，具体用法格式：telnet ip port
如果telnet连接不存在的端口，那会显示Connecttion refused
![示例1.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/linux/port01.png?raw=true)

### 方法二：curl


预置条件：安装curl
curl 的官网下载地址：http://curl.haxx.se/download/
下载地址如下： http://curl.haxx.se/download/curl-7.38.0.tar.gz
1.wget http://curl.haxx.se/download/curl-7.38.0.tar.gz
如果使用 wget下载https开头的网址域名 时报错，你可能需要加上 –no-check-certificate 选项
2.tar -xzvf curl-7.38.0.tar.gz
3.cd curl-7.38.0
./configure
make
make install

curl是利用URL语法在命令行方式下工作的开源文件传输工具。也可以用来测试端口的连通性，具体用法:curl ip:port
说明：如果远程主机开通了相应的端口，都会输出信息，如果没有开通相应的端口，则没有任何提示，需要CTRL+C断开。
![示例2.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/linux/port02.png?raw=true)

### 方法三：wget


预置条件：安装wget

debian 或者 ubuntu : sudo apt-get install wget

centos : sudo yum -y install wget

wget是一个从网络上自动下载文件的自由工具，支持通过HTTP、HTTPS、FTP三个最常见的TCP/IP协议下载，并可以使用HTTP代理。wget名称的由来是“World Wide Web”与“get”的结合，它也可以用来测试端口的连通性具体用法:wget ip:port，如果远程主机不存在端口则会一直提示连接主机。
![示例3.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/linux/port03.png?raw=true)
