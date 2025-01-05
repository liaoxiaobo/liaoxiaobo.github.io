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

## find 指定目录查找文件

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

```sh
# 创建一个50M的文件，文件名为out.txt
dd if=/dev/zero of=out.txt bs=10M count=1

# 备份 /dev/hdb 全盘数据，并利用 gzip 工具进行压缩，保存到指定路径
dd if=/dev/hdb | gzip > /root/image.gz
```

## tar 文件归档

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

## vi 纯文本编辑器

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

## scp 加密方式远程复制文件

```bash
# 从远程拷贝目录到本地
scp -r root@10.10.10.10:/opt/soft/mongodb /opt/soft/ 

# 上传本地目录到远程机器指定目录
scp -r /opt/soft/mongodb root@10.10.10.10:/opt/soft/scptest
```

## ln 创建文件链接

```Bash
# 创建软链接（类似于Windows的快捷方式）
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

# 磁盘管理

## df  显示磁盘信息

df -h 打印设备上挂载了多少磁盘，以及磁盘里还有多少可用空间。

```bash
[root@harbor ~]# df -h
文件系统        容量  已用  可用 已用% 挂载点
/dev/vda1        50G   23G   24G   50% /
devtmpfs        7.8G     0  7.8G    0% /dev
tmpfs           7.8G     0  7.8G    0% /dev/shm
tmpfs           7.8G  2.9M  7.8G    1% /run
tmpfs           7.8G     0  7.8G    0% /sys/fs/cgroup
/dev/vdc1       197G  157G   31G   84% /var/lib/docker
/dev/vdd1        99G   68G   26G   73% /data
```

## du 显示每个文件和目录的磁盘使用空间

du -sh 对文件和目录磁盘使用的空间总量的查看

# 网络管理

## netstat

有时进程之间需要通信，需要开启一个socket，socket就是对外建立连接的一个窗口，然后借助TCP协议进行通信。但进行通信之前首先需要进程开启一个端口，这时就可以通过netstat命令查看开启的端口以及由哪个进程开启

```bash
netstat -a     #列出所有端口
netstat -at    #列出所有tcp端口
netstat -au    #列出所有udp端口

netstat -l        #只显示监听端口
netstat -lt       #只列出所有监听 tcp 端口
netstat -lu       #只列出所有监听 udp 端口

-t或--tcp：显示TCP传输协议的连线状况；
-u或--udp：显示UDP传输协议的连线状况；
-n或--numeric：直接使用ip地址，而不通过域名服务器；
-l或--listening：显示监控中的服务器的Socket
-p或--programs：显示正在使用Socket的程序识别码和程序名称；
```

## ss

ss 可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效

# 系统管理命令

## ps 查看进程

ps -efww  报告当前系统的进程状态。加上ww可以完整展示进程的信息
ps -auxf | sort -nr -k 4 | head -10 找出占用内存资源最多的前 10 个进程
ps -auxf | sort -nr -k 3 | head -10 找出占用cpu资源最多的前 10 个进程

## journalctl 查看服务启动日志

journalctl -u kubelet.service

journalctl -u etcd.service -f -n 100   

> 如果不知道服务名称，可以使用`systemctl list-units --type=service`命令来列出系统中的 systemd 服务。

## watch

```
watch -d -n 60 "free -h" 每分钟动态监测主机内存的使用情况
```

## rpm 安装rpm包

```bash
# 列出所有安装过的包
rpm -qa 

# 安装rpm软件包
rpm -ivh your-package.rpm

# 获取rpm包中的文件安装路径
rpm -ql nfs-utils-1.3.0-0.54.el7.x86_64 
```

## nohup 后台运行命令

```
nohup sh install.sh > output.log 2>&1 &
```

stderr和stdout重定向输出到output.log

2>&1说明： 

1 表示stdout标准输出，系统默认值是1 

2 表示stderr标准错误，系统默认值是2 

2> 表示标准错误重定向 

& 表示等同于的意思 

2>&1 表示2的输出重定向等同于1，即标准错误重定向到标准输出

## script 记录终端会话的所有操作

```bash
script -q myfile 静默模式记录，exit 退出记录
```

## crontab 管理周期性执行任务

 crontab文件的含义：用户所建立的crontab文件中，每一行都代表一项任务，每行的每个字段代表一项设置，它的格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段，格式如下：minute   hour   day   month   week   command

```Bash
[root@master01 ~]# cat /var/spool/cron/root
#Ansible: release free
0 * * * * sh /opt/free.sh
#Ansible: disk check over 85% move vip
0 * * * * sh /opt/diskcheck.sh


[root@master01 ~]# crontab -l
#Ansible: release free
0 * * * * sh /opt/free.sh
#Ansible: disk check over 85% move vip
0 * * * * sh /opt/diskcheck.sh
```
