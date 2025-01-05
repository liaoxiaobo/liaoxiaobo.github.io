---
title: NFS服务器搭建和挂载
date:  2018-05-26
categories:
  - 工作笔记
tags:
  - NFS 
---

# nfs-server-install

NFS 是Network File System的缩写，即网络文件系统。功能是通过网络让不同的机器、不同的操作系统能够彼此分享个别的数据，让应用程序在客户端通过网络访问位于服务器磁盘中的数据，是在类Unix系统间实现磁盘文件共享的一种方法。
NFS在文件传送或信息传送过程中依赖于RPC协议。所以只要用到NFS的地方都要启动RPC服务，不论是NFS SERVER或者NFS CLIENT。这样SERVER和CLIENT才能通过RPC来实现PROGRAM PORT（centos5之前，之后是rpcbind）的对应。可以这么理解RPC和NFS的关系：NFS是一个文件系统，而RPC是负责信息的传输。

Centos系统环境：
服务端IP 192.168.1.28
客户端IP 192.168.1.21、192.168.1.155

一、安装NFS-Server

```sh
rpm -qa|grep nfs #检查系统是否已安装NFS
yum install nfs-utils -y #客户端和服务端都要安装
```



![image](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/nfs/nfs01.png?raw=true)

二、服务端配置启动NFS

```
mkdir /var/nfsshare 
vim /etc/exports #配置共享目录供客户端集群以读写权限访问
systemctl start nfs
showmount -e localhost //查询NFS的共享状态

[root@test--0004 ~]# showmount -e 192.168.1.28
Export list for 192.168.1.28:
/var/nfsshare2 *
/var/nfsshare *
```



说明：NFS服务的配置文件为/etc/exports，这个文件是NFS的主要配置文件，不过系统并没有默认值，所以这个文件不一定会存在，可能要使用vim手动建立，然后在文件里面写入配置内容。内容格式如下：

```sh
<输出目录> [客户端1 选项（访问权限,用户映射,其他）] [客户端2 选项（访问权限,用户映射,其他)]
```



```sh
1、输出目录：输出目录是指NFS系统中需要共享给客户机使用的目录；
2、客户端：客户端是指网络中可以访问这个NFS输出目录的计算机
          客户端常用的指定方式：
          指定ip地址的主机：192.168.8.106
          指定子网中的所有主机：192.168.0.0/24或 192.168.0.0/255.255.255.0
          指定域名的主机：wj.bsmart.com
          指定域中的所有主机：*.bsmart.com
          所有主机：*

3、 选项：选项用来设置输出目录的访问权限、用户映射等。

   NFS主要有3类选项：

  1）访问权限选项：

          设置输出目录只读：ro
          设置输出目录读写：rw
          
2）用户映射选项

    all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；
    no_all_squash：与all_squash取反（默认设置）；
    root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）；
    no_root_squash：与rootsquash取反；
    anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；
    anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；

3）其它选项

    secure：限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）；
    insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
    sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
    async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
    wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）；
    no_wdelay：若有写操作则立即执行，应与sync配合使用；
    subtree：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)；
    no_subtree：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；
```

三、设置开机启动

```sh
systemctl enable rpcbind.service
systemctl enable nfs-service
```



四、测试挂载：

```sh
客户端执行：
mount -t nfs 192.168.1.28:/var/nfsshare /mnt #客户端挂载NFS服务器中的共享目录
touch /mnt/test.txt #客户端挂载点生成文件，此文件会被共享至NFS服务器:/var/nfsshare
echo "hello word" > /mnt/test.txt
umount /mnt #解除和NFS服务器的挂载
```