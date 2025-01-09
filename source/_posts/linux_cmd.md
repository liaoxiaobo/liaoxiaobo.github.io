---
title: Linux命令总结
date: 2017/7/5 20:36:25
categories:
  - Linux
tags:
  - Linux
---
# 文件和目录管理

## grep

```bash
根据正则表达式查找相关内容并打印对应的数据

-A 除了显示符合范本样式的那一行之外，并显示该行之后的内容。
-B 在显示符合范本样式的那一行之外，并显示该行之前的内容。
-C 除了显示符合范本样式的那一列之外，并显示该列之前后的内容。
-c 计算符合范本样式的列数。
-i 忽略字符大小写的差别。
-v 反转查找

# 查看日志里面有 error 关键字的日志记录，与这个记录前后三行的日志信息
cat log-fail.txt |grep error -C 3
```

## awk

```bash
localhost% cat test.txt
NAME    STATUS   ROLES                    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION          CONTAINER-RUNTIME
k8s01   Ready    wise-controller          38d   v1.16.4   192.168.0.3    <none>        CentOS                  3.10.0-862.el7.x86_64   docker://18.9.8
k8s02   Ready    wise-controller          38d   v1.16.4   192.168.0.7    <none>        CentOS                  3.10.0-862.el7.x86_64   docker://18.9.8
k8s03   Ready    wise-controller          38d   v1.16.4   192.168.0.10   <none>        CentOS                  3.10.0-862.el7.x86_64   docker://18.9.8
k8s04   NotReady node                     38d   v1.16.4   192.168.0.15   <none>        CentOS                  3.10.0-862.el7.x86_64   docker://18.9.8

# 打印文件的第一列和第五列
localhost% awk '{print $1,$5}' test.txt
NAME VERSION
k8s01 v1.16.4
k8s02 v1.16.4
k8s03 v1.16.4
k8s04 v1.16.4

# 打印最后一列字段
localhost% awk '{print $NF}' test.txt
CONTAINER-RUNTIME
docker://18.9.8
docker://18.9.8
docker://18.9.8
docker://18.9.8

# 打印倒数第二列字段：
localhost% awk '{print $(NF-1)}' test.txt
KERNEL-VERSION
3.10.0-862.el7.x86_64
3.10.0-862.el7.x86_64
3.10.0-862.el7.x86_64
3.10.0-862.el7.x86_64

# 打印文本文件的总行数
localhost% awk 'END{print NR}' test.txt
5

# 打印文本第1行数据
localhost% awk 'NR==1{print}' test.txt
NAME    STATUS   ROLES                    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION          CONTAINER-RUNTIME

# 打印文本大于第1行的数据
localhost% awk 'NR>1{print}' test.txt
k8s01   Ready    wise-controller          38d   v1.16.4   192.168.0.3    <none>        CentOS                  3.10.0-862.el7.x86_64   docker://18.9.8
k8s02   Ready    wise-controller          38d   v1.16.4   192.168.0.7    <none>        CentOS                  3.10.0-862.el7.x86_64   docker://18.9.8
k8s03   Ready    wise-controller          38d   v1.16.4   192.168.0.10   <none>        CentOS                  3.10.0-862.el7.x86_64   docker://18.9.8
k8s04   NotReady node                     38d   v1.16.4   192.168.0.15   <none>        CentOS                  3.10.0-862.el7.x86_64   docker://18.9.8
```

## sed

定位到数据行并对数据进行增删改查操作

```bash
# 替换文本中的字符串：
sed 's/book/books/' file

# 使用后缀/g标记会替换每一行中的所有匹配：
[root@harbor image_pull_push]# cat img-list.txt |sed 's/registry.cn-hangzhou.aliyuncs.com/192.168.0.11/g'

# 当需要从第N处匹配配开始替换时，可以使用 /Ng：
echo sksksksksksk | sed 's/sk/SK/3g'
skskSKSKSKSK

# 替换某目录下所有文件的某字符串
oldip=172.22.12.241
newip=172.22.12.242
# 查看之前包含oldip的文件
find . -type f | xargs grep $oldip
# 替换IP地址
find . -type f | xargs sed -i "s/$oldip/$newip/"
# 检查更新后的
find . -type f | xargs grep $newip
```

## find

```bash
# 当前目录搜索所有文件，文件内容 包含 “140.206.111.111” 的内容
find . -type f -name "*" | xargs grep "140.206.111.111"

# 查找当前目录下是否存在README.md 
find . -name README.md

# 查找根目录/是否存在nginx.conf
find / -name nginx.conf 

# 指定目录查找大于1G的文件
find /data -size +1024M 
```

## echo

```bash
# 创建带有内容的文件
[root@harbor ~]# echo hello > new.yaml
[root@harbor ~]# cat new.yaml
hello
# 追加写入文件
[root@harbor ~]# echo hello-test >> new.yaml
[root@harbor ~]# cat new.yaml
hello
hello-test
```

## wc

```bash
wc -l 统计文本行数
wc -w 统计文本字数
wc -c 统计文本字节数
```

## dd

**dd命令** 用于复制文件并对原文件的内容进行转换和格式化处理。dd命令功能很强大的，对于一些比较底层的问题，使用dd命令往往可以得到出人意料的效果。

```sh
[root@localhost text]# dd if=/dev/zero of=sun.txt bs=1M count=1
1+0 records in
1+0 records out
1048576 bytes (1.0 MB) copied, 0.006107 seconds, 172 MB/s

[root@localhost text]# du -sh sun.txt 
1.1M    sun.txt
```

该命令创建了一个1M大小的文件sun.txt，其中参数解释：

- **if** 代表输入文件。如果不指定if，默认就会从stdin中读取输入。
- **of** 代表输出文件。如果不指定of，默认就会将stdout作为默认输出。
- **bs** 代表字节为单位的块大小。
- **count** 代表被复制的块数。
- **/dev/zero** 是一个字符设备，会不断返回0值字节（\0）。

**测试磁盘写入速度**

```shell
[root@localhost ~]# dd if=/dev/zero of=/tmp/testfile bs=1G count=1 oflag=direct
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB) copied, 7.10845 s, 151 MB/s
```

**测试磁盘读取速度**

```shell
[root@localhost ~]# dd if=/tmp/testfile of=/dev/null bs=1G count=1 iflag=direct
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB) copied, 6.53009 s, 164 MB/s
```

## tar

tar参数介绍

```Bash
-c：创建一个包
-v：显示所有过程
-f：指定包的文件名
-z：有gzip属性的
-j：有bz2属性的
-x：从包中解压
```

```bash
# 仅打包，不压缩！将 /home/vivek/bin/ 目录打包
tar -cvf /tmp/bin-backup.tar /home/vivek/bin/

# 打包 并使用 bzip2 算法压缩压缩
tar -jcvf /tmp/bin-backup.tar.bz2 /home/vivek/bin/

# 打包 并使用 gzip 算法压缩压缩
tar -zcvf /tmp/bin-backup.tar.gz /home/vivek/bin/

# 解压缩
tar -zxvf bin-backup.tar.gz

# 详细列举包里的所有文件
tar -tvf /opt/extra.tgz
```

## vi

vi编辑器提供了丰富的内置命令，有些内置命令使用键盘组合键即可完成，有些内置命令则需要以冒号“：”开头输入。常用内置命令如下：

```bash
上下翻页：Ctrl+f、Ctrl+b
跳到当前行最后字符处：Fn+右键
跳到当前行最前面字符处：Fn+左键、0

查找字符串：/字符串  n继续查找
字符全局替换：:%s/foo/bar/g  :{作用范围}s/{目标}/{替换}/{替换标志}

x或X：删除一个字符，x删除光标后的，而X删除光标前的；
D：删除从当前光标到光标所在行尾的全部字符；
删除行：dd ndd（n代表删除或复制的行数）
复制行：yy  nyy
粘贴行：p、P

复原前一个操作：u
重做上一个操作：ctrl+r
显示行号：:set nu


Ctrl+u：向文件首翻半屏；
Ctrl+d：向文件尾翻半屏；
Ctrl+f：向文件尾翻一屏；
Ctrl+b：向文件首翻一屏；
Esc：从编辑模式切换到命令模式；
ZZ：命令模式下保存当前文件所做的修改后退出vi；
:行号：光标跳转到指定行的行首；
:$：光标跳转到最后一行的行首；

```

## scp

**scp命令** 用于在Linux下进行远程拷贝文件的命令，和它类似的命令有cp，不过cp只是在本机进行拷贝不能跨服务器，而且scp传输是加密的。

```bash
# 从远程拷贝目录到本地
scp -r root@10.10.10.10:/opt/soft/mongodb /opt/soft/ 

# 上传本地目录到远程机器指定目录
scp -r /opt/soft/mongodb root@10.10.10.10:/opt/soft/scptest
```

## ln

**ln命令** 用来为文件创建链接，链接类型分为硬链接和符号链接两种，默认的链接类型是硬链接。如果要创建符号链接必须使用"-s"选项。

```Bash
# 创建符号链接（类似于Windows的快捷方式）
ln -sf /etc/hostname /usr/share/nginx/html/index.html

# 创建硬链接，两个文件的inode号相同,inode信息中的“链接数”会+1。当链接数减到0，即表明文件已删除
[root@master01 wh_log]# ln wh_test.sh link001.sh
[root@master01 wh_log]# ll -li
总用量 1800
17293406 -rw-r--r-- 1 root root 187963 12月 18 10:13 es-1.log
17293408 -rw-r--r-- 1 root root 223798 12月 18 10:13 es-2.log
17293409 -rw-r--r-- 1 root root 214475 12月 18 10:13 es-3.log
17293557 -rw-r--r-- 2 root root    201 12月 19 08:46 link001.sh
17293554 -rw-r--r-- 1 root root 186335 12月 18 15:13 pgsql-1.log
17293555 -rw-r--r-- 1 root root 435966 12月 18 15:13 pgsql-2.log
17293556 -rw-r--r-- 1 root root 546363 12月 18 15:13 pgsql-3.log
17293407 -rw-r--r-- 1 root root    225 12月 18 10:12 test.sh
17293557 -rw-r--r-- 2 root root    201 12月 19 08:46 wh_test.sh
```



# 网络管理

## ip

**ip命令** 用来显示或操纵Linux主机的路由、网络设备、策略路由和隧道，是Linux下较新的功能强大的网络配置工具。

网卡配置

```Bash
ip link                          # 显示网络接口信息
ip link set eth0 up              # 开启网卡
ip link set eth0 down            # 关闭网卡
ip link set eth0 promisc on      # 开启网卡的混合模式
ip link set eth0 promisc offi    # 关闭网卡的混合模式
ip link set eth0 txqueuelen 1200 # 设置网卡队列长度
ip link set eth0 mtu 1400        # 设置网卡最大传输单元

ip a                             # 显示网卡IP信息
ip a a 192.168.0.1/24 dev eth0 # 为eth0网卡添加IP地址192.168.0.1
ip a d 192.168.0.1/24 dev eth0 # 为eth0网卡删除一个IP地址192.168.0.1
```

> ip命令配置网卡信息，在网卡或机器重启后配置会丢失。要想配置永久生效，那就要修改网卡配置文件。如下：
>
> ```Bash
> # 文件名对应网口名，CentOS 7\8 默认配置文件
> [Linux]# vi /etc/sysconfig/network-scripts/ifcfg-eth0
> DEVICE="eth0"
> BOOTPROTO="static"
> BROADCAST="192.168.0.255"
> HWADDR="00:16:36:1B:BB:74"
> IPADDR="192.168.0.100"
> NETMASK="255.255.255.0"
> ONBOOT="yes"
> ```

路由配置

```sh
ip r                         # 显示系统路由
ip r add default via 192.168.1.254   # 设置系统默认路由
ip r d default          # 删除默认路由

ip r a 192.168.4.0/24 via 192.168.0.254 dev eth0 # 设置192.168.4.0网段的网关为192.168.0.254,数据走eth0接口
ip r a default via 192.168.0.254 dev eth0        # 设置默认网关为192.168.0.254
ip r d 192.168.4.0/24   # 删除192.168.4.0网段的网关
ip r d 192.168.1.0/24 dev eth0 # 删除路由
```

## netstat

有时进程之间需要通信，需要开启一个socket，socket就是对外建立连接的一个窗口，然后借助TCP协议进行通信。但进行通信之前首先需要进程开启一个端口，这时就可以通过netstat命令查看开启的端口以及由哪个进程开启

```bash
#列出所有端口
netstat -a
#列出所有tcp端口
netstat -at
#列出所有udp端口
netstat -au
#只列出所有监听 udp 端口
netstat -lu
# 通过端口找进程ID
netstat -anp|grep 8081 | grep LISTEN|awk '{printf $7}'|cut -d/ -f1

-n或--numeric：直接使用ip地址，而不通过域名服务器；
-l或--listening：显示监控中的服务器的Socket
-p或--programs：添加“PID/进程名称”到netstat输出中；
```

## ss

ss 可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。
```sh
ss -t -a    # 显示TCP连接
ss -u -a    # 显示所有UDP Sockets
ss -s       # 显示 Sockets 摘要
ss -l       # 列出所有打开的网络连接端口
ss -pl      # 查看进程使用的socket
ss -lp | grep 3306  # 找出打开套接字/端口应用程序
```

# 系统管理

## firewalld-cmd

firewalld是centos7的一大特性，最大的好处有两个：支持动态更新，不用重启服务；第二个就是加入了防火墙的“zone”概念。

firewalld自身并不具备防火墙的功能，而是和iptables一样需要通过内核的netfilter来实现，也就是说firewalld和 iptables一样，他们的作用都是用于维护规则，而真正使用规则干活的是内核的netfilter，只不过firewalld和iptables的结 构以及使用方法不一样罢了。

```Bash
[root@master01 ~]# firewall-cmd --zone=public --list-ports
2121/tcp 3306/tcp 8081/tcp 30000/tcp 38060/tcp 40000/tcp 40001/tcp

[root@master01 ~]# firewall-cmd --permanent --add-port=3307/tcp --zone=public
success

[root@master01 ~]# firewall-cmd --reload
success

[root@master01 ~]# firewall-cmd --zone=public --list-ports
2121/tcp 3306/tcp 3307/tcp 8081/tcp 30000/tcp 38060/tcp 40000/tcp 40001/tcp
```

firewalld中常见的zone（默认为public）以及相应的策略规则如下所示：

| 区域     | 默认规则策略                                                 |
| -------- | :----------------------------------------------------------- |
| trusted  | 允许所有的数据包                                             |
| home     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、mdns、ipp-client、amba-client与dhcpv6-client服务相关，则允许流量 |
| internal | 等同于home区域                                               |
| work     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、ipp-client与dhcpv6-client服务相关，则允许流量 |
| public   | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、dhcpv6-client服务相关，则允许流量 |
| external | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量 |
| dmz      | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量 |
| block    | 拒绝流入的流量，除非与流出的流量相关                         |
| drop     | 拒绝流入的流量，除非与流出的流量相关                         |

## ps

**ps命令** 用于报告当前系统的进程状态。

ps命令是最基本同时也是非常强大的进程查看命令，使用该命令可以确定有哪些进程正在运行和运行的状态、进程是否结束、进程有没有僵死、哪些进程占用了过多的资源等等，总之大部分信息都是可以通过执行该命令得到的。

```sh
# 加上ww可以完整展示进程的信息
ps -efww
# 找出占用内存资源最多的前 10 个进程
ps -auxf | sort -nr -k 4 | head -10
# 找出占用cpu资源最多的前 10 个进程
ps -auxf | sort -nr -k 3 | head -10 
```
## journalctl

检索 systemd 日志，是 CentOS 7 才有的工具

```sh
# 查看服务启动日志
journalctl -u kubelet.service
journalctl -u etcd.service -f -n 100 
```
> 如果不知道服务名称，可以使用`systemctl list-units --type=service`命令来列出系统中的 systemd 服务。

## watch

**watch命令** 以周期性的方式执行给定的指令，指令输出以全屏方式显示

```sh
# 每隔60s查看主机内存的使用情况
watch -d -n 60 "free -h"
```

## rpm

**rpm命令** 是RPM软件包的管理工具

```bash
# 列出所有安装过的包
rpm -qa 

# 安装rpm软件包
rpm -ivh your-package.rpm

# 获取rpm包中的文件安装路径
rpm -ql nfs-utils-1.3.0-0.54.el7.x86_64 
```

## nohup

**nohup命令** 可以将程序以忽略挂起信号的方式运行起来，被运行的程序的输出信息将不会显示到终端。

```sh
# 后台运行脚本，stderr和stdout重定向输出到output.log
nohup sh install.sh > output.log 2>&1 &
```

> 2>&1说明： 
>
> 1 表示stdout标准输出，系统默认值是1 
>
> 2 表示stderr标准错误，系统默认值是2 
>
> 2> 表示标准错误重定向 
>
> & 表示等同于的意思 
>
> 2>&1 表示2的输出重定向等同于1，即标准错误重定向到标准输出

## script

**script** 用于在终端会话中，记录用户的所有操作和命令的输出信息。

使用命令`exit`或者快捷键`Ctrl + D`停止记录。

```bash
# 静默模式记录
script -q myfile
```

## crontab

**crontab命令** 被用来提交和管理用户的需要周期性执行的任务

用户可以使用 crontab 工具来定制自己的计划任务。所有用户定义的crontab文件都被保存在`/var/spool/cron`目录中。其文件名与用户名一致。

crontab文件的含义：用户所建立的crontab文件中，每一行都代表一项任务，每行的每个字段代表一项设置，它的格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段，格式如下：

```shell
minute   hour   day   month   week   command     顺序：分 时 日 月 周
```

```Bash
[root@master01 ~]# cat /var/spool/cron/root
#Ansible: release free
0 * * * * sh /opt/free.sh
#Ansible: disk check over 85% move vip
0 * * * * sh /opt/diskcheck.sh

# 列出该用户的定时任务
[root@master01 ~]# crontab -l
#Ansible: release free
0 * * * * sh /opt/free.sh
#Ansible: disk check over 85% move vip
0 * * * * sh /opt/diskcheck.sh
```
