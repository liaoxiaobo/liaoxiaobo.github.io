# cce容器监控

## 集群创建流程分析

### 1、创建实例节点的安全组及规则

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NDA2NjUwMDJhM2YxNmE2ZWUxZTI5YTMwMjE5Mzg0MDRfMzQxNTdmYmVmMmQxOTVlNmZhMjkzZTEzNDkxMmIxMTJfSUQ6NzY0NjQ5MDY2OTA1NzM3OTU0Nl8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=Y2IyNmFlMWE3MjEyODg2N2VlMWRhNWRiY2EzYWEwMTFfZTJmZjE0YTM5Y2JlNmMyYzBkYjc3Y2FhNjg3OWJhYTNfSUQ6NzY0NjQ5MDY5MDQyMzE5Njg2MF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 2、cce\-api调用ros服务创建虚机

ros负责调用底层计算服务（nova、gova）创建虚机，此时会生成一个template\_id `6947565d1fdb43b196ec3ee6e345059f`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MGM0ODE4NzM5YzE1NGZmNjIyYmI3YzNjZWYxOGExOWFfZmNlNDkxNzJiOTg5NDVlOGMwNWNkYTkyNWQwZmU3NDZfSUQ6NzY0NjQ5MDY4NzI4MTYxNDAyOF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YjZmNjAyOTA4NjM2NTA4NzBmMTE0YzkxZTljODRmYmJfZmQzNjU1ZDI2ODIxODQ1MmYyYjU3Y2ZiM2Q4MmI3ZjBfSUQ6NzY0NjQ5MDYzNzczMDIzNzYyNl8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

ros服务处理创建虚机的请求，线程执行个数等于创建虚机的数量。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZDk4YzllMzUzYmZmN2UwYWViZTc2M2EwMjAyNTMwYzlfMTRmMTkyOTAyOGIwMGJlZDBmNzhiMDA3ZTIyM2JjZDBfSUQ6NzY0NjQ5MDY2MzAzMDEzMTg5MF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

待虚机全部创建成功，ros通过MQ向cce发送消息，消息内容即虚机的metadata信息

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MTVhNzFkZjI2ZjQyZDdjNTJiYWJkY2YyNTg0YWIzMGRfMTNhNjRkZGZmNTQ0ZjdjODdhMjI3NzkzZTIwZjg5YjNfSUQ6NzY0NjQ5MDcxMjIwODU0MjkyMV8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 3、cce\-api通过mfip与节点建立网络通信

cce收到消息，会先去检查mfip是否连通，接着继续k8s集群的部署，整个过程约10分钟

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=Mzg0YTljZDAzNzMyY2I4Yjc3NTE3MzU2NTczNDZkMGFfMWM1MGI2N2I3MTEzNDAxYWUzZTYwZmRlM2E5OGNlMzdfSUQ6NzY0NjQ5MDcwNjQ0NTM4OTAyMl8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YjVkNWZiNWFmODlkMWQxNjQ3YTQ2M2U4MmM0MzYwZThfNGYzOGUyODIzNjIzM2FmYjc3MGMxZDMxMDQ0MGRjMWZfSUQ6NzY0NjQ5MDY0MTQyMjkyOTExOF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OGFhZTMyNDg3NDU2MWFlNjAyZTM0YzJiMDhhMDAzMThfZDQ0MGIyODdmZTdiZTExNmVlNzYxMTM0NDViOThhOWNfSUQ6NzY0NjQ5MDY0OTg5MzUzOTAwNF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 4、节点内下载安装包，自动化部署docker、k8s等

PAAS虚机的内部会自动安装dbagent\-proxy、dbagent\-server两个服务。（各个paas api就是通过dbagent\-proxy入口 ，才能成功调用dbagent\-server去执行安装的指令）

- dbagent\-proxy 就是请求dbagent\-server的入口

- dbagent\-server 就是安装各自PAAS云服务指令（如cce）的服务端

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZTQyZmQ3YzFkMTA1NWFhNzJjNjQwOGJlN2I3YzU1OWVfNWNkY2Q3ZTMwMDYwYWQyMzhhMGM4NmNiMTYzZDQ0NDZfSUQ6NzY0NjQ5MDYzNTM4OTcxNzY4NV8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NjY1NzVjMTk0NTY5MWRmNzEzYzkxNTQ4Y2NlZWIwMWVfZjk5YmE1Nzg4YjhjMjg5YzEwMjhmYmU5OWNkM2VkMzlfSUQ6NzY0NjQ5MDcwOTI0NzI0OTYwNF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OGRhNjcyNTBkMjNiYzdjMDYxNDUwMGQwMmRiZjYzMWNfMDJmMzM3MWQxNzYwNjdiMmE2Mzc3NWNhYjJlOTE0ZTRfSUQ6NzY0NjQ5MDY5NTIwNDYzNzg4Ml8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZDk1OTZmYWVlZjA0NzZiMzMxNDFmMTVlZjhhNWRmMThfMDgxOWVmNGI5OGE0YjZhZDBjNjc5YWM0YzBiYmJhZWJfSUQ6NzY0NjQ5MDY4MjY0Mjg2MTIzN18xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

7、CCE集群部署成功

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OWUxZmY5OTA0YzhhMjFhMGUzNGY0ZmNhZWY5MzA1MDVfYWEzZTI5ZjFjYmIzOTliNjg4NzBjMTg1YmE0YWRmYWNfSUQ6NzY0NjQ5MDY2NTcxMDI5MjE4MF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

## 容器监控原理

Docker 是基于 Namespace、Cgroups 和联合文件系统实现的。其中 Cgroups 不仅可以用于容器资源的限制，还可以提供容器的资源使用率。**无论何种监控方案的实现，底层数据都来源于 Cgroups。**

我们通过 ls \-l 命令查看`/sys/fs/cgroup`文件夹，可以看到很多目录：

```Rust
$ sudo ls -l /sys/fs/cgroup/
total 0
dr-xr-xr-x 5 root root  0 Jul  9 19:32 blkio
lrwxrwxrwx 1 root root 11 Jul  9 19:32 cpu -> cpu,cpuacct
dr-xr-xr-x 5 root root  0 Jul  9 19:32 cpu,cpuacct
lrwxrwxrwx 1 root root 11 Jul  9 19:32 cpuacct -> cpu,cpuacct
dr-xr-xr-x 3 root root  0 Jul  9 19:32 cpuset
dr-xr-xr-x 5 root root  0 Jul  9 19:32 devices
dr-xr-xr-x 3 root root  0 Jul  9 19:32 freezer
dr-xr-xr-x 3 root root  0 Jul  9 19:32 hugetlb
dr-xr-xr-x 5 root root  0 Jul  9 19:32 memory
lrwxrwxrwx 1 root root 16 Jul  9 19:32 net_cls -> net_cls,net_prio
dr-xr-xr-x 3 root root  0 Jul  9 19:32 net_cls,net_prio
lrwxrwxrwx 1 root root 16 Jul  9 19:32 net_prio -> net_cls,net_prio
dr-xr-xr-x 3 root root  0 Jul  9 19:32 perf_event
dr-xr-xr-x 5 root root  0 Jul  9 19:32 pids
dr-xr-xr-x 5 root root  0 Jul  9 19:32 systemd


```

- 容器cpu监控数据来源于`/sys/fs/cgroup/cpu/docker/<容器ID>`目录

- 容器内存监控数据来源于`/sys/fs/cgroup/memory/docker/<容器ID>`目录

- 容器磁盘监控数据来源于`/sys/fs/cgroup/blkio/docker/<容器ID>`目录

- 容器网络监控数据来源于`/proc/<容器PID>/net/dev`目录

## 容器组监控指标 \{folded="true"\}

部署一个多容器的pod，然后对cpu进行加压

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MmI2NTYxODhlOGI2N2VjNGRmMGNiMTQzMTVkYmJlMzNfOTZmYjgxNjY5OTg3YjJlN2I0NmMzYTUxM2Q4OWY0ZmZfSUQ6NzY0NjQ5MDY5ODI3OTA0NjMzMl8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 内存使用量

存在如下两种计算指标

`sum(container_memory_working_set_bytes{pod="test-centos-68ff9cc8cf-j9qkp",container!="POD",container!=""})/1024/1024`

`sum(container_memory_usage_bytes{pod="test-centos-68ff9cc8cf-j9qkp",container!="POD",container!=""})/1024/1024`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NTE5NjI4ZWFjNDg4ODIwMjgyNGE0MTRhYTVhMDVkM2VfYjRkOTkwNzU4MDAyY2VlMjAxMTdiYTFkNDkyMTQ0MzJfSUQ6NzY0NjQ5MDY3MjQ1MDU3MTQ3OF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

`container_memory_working_set_bytes` 指标更准确合理，与kubectl top pod查询值一致

### CPU使用量

`sum(rate(container_cpu_usage_seconds_total{pod="test-centos-68ff9cc8cf-j9qkp", container!="", container!="POD"}[1m]))*1000`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NDM0N2I3Y2JkNDc3MTY1MzQ4NmMyZjc1Yjk3NjY5ODhfOTVkYzg2OTkzNWE2NzViMjI4ZTBmNzQyZjUzNDlkNmFfSUQ6NzY0NjQ5MDY3NzczNTM2MTc0Nl8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 文件系统使用量

`sum(container_fs_usage_bytes{pod="test-centos-68ff9cc8cf-j9qkp",container!="POD",container!=""})/1024/1024`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=N2U4N2UwYzFlZWQ1NjM5MTUwMGU3NjBiOTA5OWY0OWNfZTgwY2ViMGY5YTMzNjM1OWQxZTdjNjg4NDdjMWU0Y2JfSUQ6NzY0NjQ5MDY5OTgwMTcwOTc4Nl8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 网络传输/接收速率

`rate(container_network_transmit_bytes_total{pod="test-centos-68ff9cc8cf-j9qkp",interface="eth0"}[1m])`

`rate(container_network_receive_bytes_total{pod="test-centos-68ff9cc8cf-j9qkp",interface="eth0"}[1m])`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ODlmMTgyMzcxYzZmNzQ0MjczYmZiNjRiYjYxNWY3MDZfZTY0ZTUyNTEwYmM3OGU3ZDA0MjkwZDY4OGQwZWVkNDZfSUQ6NzY0NjQ5MDY1NDc0MjIwMzYxNF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

## 节点监控指标

### CPU使用率

使用命令打满一个cpu

`1-avg(irate(node_cpu_seconds_total{mode="idle",kubernetes_node="demo-master-1"}[5m]))`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OTYxN2ZiY2QwZjBhYWUyYmI2NGRlNDg4M2JmNWFlN2FfMTFmZjBmZjFlM2Q0NGNlNzRkZWEwNmFmN2EwYmU3YWZfSUQ6NzY0NjQ5MDY1ODM5OTYwMzkwOF8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 1分钟负载

`node_load1{kubernetes_node="demo-master-1"}`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OGRmMGUwOTc0YWJkZWQ5ODFiMmJkZWYzMjdkN2E2ZDhfMmFjNzc4Y2QxZjQxYzczYjJmYWM3ZjNmMTczNjE5OTJfSUQ6NzY0NjQ5MDcyNTEyNjgxODk5N18xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 可用内存

`node_memory_MemAvailable_bytes{kubernetes_node="demo-master-1"}/1024/1024`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MDcxYzk5NjY4Y2EzZDE4ZTk4YTEwNTYwZTcwYjI2ODNfZGIzYjQzMTExZWZlZDI0YTdkNjEwMzE4ZDdlMGViZmRfSUQ6NzY0NjQ5MDcyNjQyMjk0MDg3OV8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ODVmZmE5ZWYyZGFkMDk4YmUxZGY2ZmRmYWU4MWRhN2VfNzEzZmMxYTQ0OGYzZGQ1MWU2NWIyOTY2MWQ1ZDc5OTJfSUQ6NzY0NjQ5MDczODUyMzQ1ODc1N18xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 内存使用率

`1-(node_memory_MemAvailable_bytes{kubernetes_node="demo-master-1"}/node_memory_MemTotal_bytes{kubernetes_node="demo-master-1"})`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OTg2Mjk4MGU3NmFjOGJmNDMxZjdkMTdjNWFhY2M2YzhfNzhiZmIyMTliYmJiMWMwZTg1Njk1MzRlZTc0OTA0ZWFfSUQ6NzY0NjQ5MDcyMDY4NTE0OTM2NV8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 文件系统容量

`node_filesystem_size_bytes/1024/1024/1024`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MTc1MDcwZjI3NDFiN2U4ODhmYzY0NzQ1NWNlMDlhMzFfZjRjN2Y0NzY5MDVlODZkNjBlMTUxNjcxOGIyYzQ0MTJfSUQ6NzY0NjQ5MDc0MTQ1NTMyNjQxMl8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MDNmZWU0NmNkMjQ3ZmMwYzExOWI1NTYwZDQ0MTc3MmJfZTI0ZTY2MzE4MTFmMTY2ZTgyZjNiOTM3MDYzN2ZiNDhfSUQ6NzY0NjQ5MDczNTI2ODcyODAxM18xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

cce问题记录

1、prometheus只能查询到节点系统盘的容量，不能查询到数据盘

2、prometheus查询到节点系统盘的容量50G，与后台实际100G不符

### 磁盘IOPS

- 磁盘I/O：I/O，即input/output，磁盘的输入输出，输入指的是对磁盘写入数据，输出指的是从磁盘读出数据，磁盘I/O可以理解为读写。应用发起的一次或多次数据请求，I/O请求的数据量又称I/O大小，单位为KiB，例如4KiB、256KiB、1024KiB等；

- 磁盘IOPS：磁盘IOPS是指一秒内磁盘进行多少次I/O读写；

- 磁盘吞吐量：每秒磁盘I/O的流量，即磁盘写入加上读出的数据的大小

综上，磁盘I/O、IOPS和吞吐量的关系公式为：

> 吞吐量 = IOPS \* I/O大小
> 
> 

`irate(node_disk_reads_completed_total{kubernetes_node="demo-master-1"}[5m])`

`irate(node_disk_writes_completed_total{kubernetes_node="demo-master-1"}[5m])`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NDE4Y2IyNWViNWVkYjRjYWIwZmNkNDA1ZmNkNWYyMGVfOGY2ZDkzZTExNGYzM2ExYjkxNDY5NzUwYTVhZTAxNDNfSUQ6NzY0NjQ5MDczMDc5NzUzNDQwMl8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 磁盘读写速率

`irate(node_disk_written_bytes_total{kubernetes_node="demo-master-1"}[5m])`

`irate(node_disk_read_bytes_total{kubernetes_node="demo-master-1"}[5m])`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=Y2JkMmJjNTIzZTVkNjM4ZDhkODg3ODlmNzA5NGFiOWNfODJkYjY1ZTNmNDg3YmI2YTQ4OWU3MzE2NjlkNTBmZGVfSUQ6NzY0NjQ5MDc0NDkzMjQwNDQwNV8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

### 网络传输/接收速率

`rate(node_network_receive_bytes_total{kubernetes_node="demo-master-1"}[2m])`

`rate(node_network_transmit_bytes_total{kubernetes_node="demo-master-1"}[2m])`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MWJjYWYwNGE0OWFhNmQ1OTE5MjI1NTc5ZTY0MDM5MzhfZTE3NzE5NGI0YjFiNzc5YWViYzVkNmI2NWQ5YmMwNmVfSUQ6NzY0NjQ5MDc0OTE4OTU5MDIxMl8xNzgxMzc0ODYzOjE3ODE0NjEyNjNfVjM)

