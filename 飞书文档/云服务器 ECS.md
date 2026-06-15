# 云服务器 ECS

# 云服务器 ECS

## 核心能力

**一云多芯场景支持**

- 在云平台同一区域内提供鲲鹏、飞腾、海光、intel、AMD等不同芯片架构的物理节点，为用户提供一致的使用体验。

**广泛的业务场景支持**

- 提供通用计算型、GPU计算加速型等云主机规格，支持多种加速直通设备的使用，满足不用行业不同场景需求。

**高可用能力**

- 故障自动快速恢复，存储多副本；云主机备份以及容灾保护、快照保护；强大的云主机高可用能力

**灵活的弹性伸缩能力**

- 根据业务和场景的需要，可以灵活的调整云主机CPU、内存大小，动态的增减云硬盘数量以及扩容、实时更新云主机带宽等，精准的匹配业务要求

## 功能操作

### 变更规格

- 支持在线升配和关机降配两种操作

- 该操作会变更虚机xml里的memory、vcpu字段值

### 连接实例

- SSH密钥对\&密码登录（仅Linux实例）

- VNC 连接

- 远程用户名密码连接（需要绑定公网IP）

### 重置登录密码

**前提条件**

- 实例已安装密码重置插件fsagent

- 确保插件未被安全软件阻止运行，否则重置密码功能无法使用

**实现方案**

1. 调用metadata接口获取实例的uuid，调用common接口获取password\_url，如图

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NDhmNTRlNTYwN2VjZTdlZTgyNDM5MDFjODgxYjlkYmVfZWMxODQ2ODNhM2QyODFjZTMwNTk3ODljMTA0Mjc1MzFfSUQ6NzY0ODE5OTA0MjExNTcwMTk5Nl8xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

```Bash
curl 169.254.169.254/openstack/latest/meta_data.json
curl 169.254.169.254/common

```

2. 根据uuid、password\_url、os\_type，拼接成curl请求，每隔30s执行一次去获取最新shell脚本。若有新密码信息则执行该脚本，实现在线重置密码

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YTM1ZmI0YWZkNmI4ZjMxMjhhMTRiZmEyNmM0ZDA0ZjdfNDhjMDM0YTNlZGQ5OGZjNDUwZjBjZmY0NjE1M2E4MWFfSUQ6NzY0ODE5OTA0MjMwMDAyMTk3N18xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

```Bash
curl http://100.126.255.250:30546/password/<uuid>

```

```Bash
echo "scloudadmin:123456" | sudo chpasswd
# window
net user "Administrator" "<yourPassword>"

```

### 时间同步

实现方案同重置登陆密码

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NGM2YjNjODEzZWM4NmQ5ZGFmNDVhNWY3MDMyOGVkOWJfYzVlMDczZDkxOGM0Yjg2NTFkMTExZjkwMjAwZjZiZWZfSUQ6NzY0ODE5OTA0MjU4MDk3NDc4OV8xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

### 重建操作系统

- 重装操作系统后弹性云服务器IP地址和MAC地址不发生改变。

- 重装操作系统会清除系统盘数据，包括系统盘上的系统分区和所有其它分区，请做好数据备份。

- 重装操作系统不影响数据盘数据。

- 该操作会修改虚机xml里系统盘盘符（通常是vda）的image id字段值

### 热迁移

## 关联资源

### 设置安全组

- 将ECS实例加入安全组时，实际上是将ECS实例的网卡加入了安全组。

- 用户可以根据源IP地址、应用层协议、端口等配置精准的安全组规则，以实现对每块弹性网卡的流量进行安全访问控制。

### 修改内网IP

- 变更内网IP会重启云服务器

### 绑定\&解绑公网IP

- 实例与EIP是一对多的关系，即一个实例（多网卡）可以绑定多个公网IP

- 默认情况下，除了少数安全端口之外，云服务器的大部分端口都是关闭的，您需要在安全组中打开相应的下行规则以允许外网访问。

### 挂载\&卸载网卡

- 实例与网卡（PORT）是一对多的关系，即一个实例可以挂载多张网卡

新增\&移除`<interface type='bridge'>`的网络配置字段

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZGJmNjU3ZTU3NGNmMjI1OTY3YTIxYTA3MjViMjRmOTJfMTc5ZGZhZTEyNmI3ZGYyZjRjMjc2ZmI2OWNkYTM4NjlfSUQ6NzY0ODE5OTA0NDIxNjU1Njc3Nl8xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

### 挂载\&卸载云盘

- 挂载后，此时磁盘列表中“状态”变更为“已挂载”。在使用之前，您需要对该 [云盘格式化](https://cloud.baidu.com/doc/CDS/s/Dk0dargsj)。格式化完成后，磁盘方可使用。

- 点击"卸载"后，若在 Windows 系统中操作，请先对要卸载的磁盘进行脱机；若在 Linux 系统中操作，请先对磁盘执行 `umount` 操作

新增\&移除`<target dev='vdb' bus='virtio'/>`的disk配置，如图

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MGZmNTAwYzdiN2Y2ODk1N2Y3NDRmZjg5ZmM4MThmZGZfNzdkZTlkMDllYTgzZWY5MDliMmViZGMwYzA5YzBjY2ZfSUQ6NzY0ODE5OTA0NTE3MjgwODkwMl8xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

### 挂载\&卸载**主机设备**

新增\&移除 `<disk type='block' device='disk'>`、`<controller type='scsi'>`配置字段

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NjViYTA5NDdmZWEzNDgxMDllNTcwNmJkYmI3ZjQxODVfMWJiYzA2ODVhZjVhZGU1YjFjY2FhZjYyNjEwYThiZDBfSUQ6NzY0ODE5OTA0Mjk0NTgzMDEyNF8xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

### 挂载\&卸载CDROM

3. 创建一个空启动的虚机（此时vnc界面会显示No bootable device\.）

4. 上传iso镜像文件，并挂载cdrom（作为一个block设备，挂载到主机目录/dev/sdai）

5. 设置第一启动顺序为cdrom（变更`device='cdrom'`，配置`<boot order='1'/>`参数）

6. 硬重启虚机，根据引导提示开始安装操作系统

7. 重装成功后，卸载cdrom

变更`<disk type='block' device='cdrom'>`的配置，如图

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NTM5NzZiZTcwOTFlN2RmOWZiODBkZGExNjU4NjNiMWJfOTlhMDFjZjFkNjI0NzlmOWYwMDUzYWViYTg3NjhkNWFfSUQ6NzY0ODE5OTA0MjY2OTEyMDc0NF8xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

### 挂载\&卸载GPU

页面显示虚机已经挂载GPU成功

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YzhjMzgwZjc5MWQ1ODhmOTEwZTMyY2M5ZmJhMGYwYTlfZDMxYmFkZTQ5YmUxYTNiMjUwNzAwMGJjYmY3OTY2Y2FfSUQ6NzY0ODE5OTA0MTc5MjQ5NDgzM18xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

虚机的XML配置验证

```XML
<hostdev mode='subsystem' type='pci' managed='yes'>
      <driver name='vfio'/>
      <source>
        <address domain='0x0000' bus='0x5e' slot='0x00' function='0x0'/>
      </source>
      <alias name='hostdev0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </hostdev>

```

执行 `lspci |grep NVIDIA`，虚拟机能成功识别gpu设备

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NzQxZDI2MzgyMzRiYmU5ODk0NWIxODZmMTM0ZGFjZGRfNzU4MTY5Yjc4MzlmOGEzNmY0YjUzODk1OWZiMzE2MjRfSUQ6NzY0ODE5OTA0MzI4OTgyODU4OF8xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

虚机内安装GPU相关驱动完成后，执行`nvidia-smi`检查驱动信息

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NDVjYzMzOGFiNTBlMjFhNTM3MmNkY2UyZWEyNTY0NjlfZWI1Njg4YTVlZTBmNGY0YThlMmViNGRmNzcyMTlhNjhfSUQ6NzY0ODE5OTA0MjA4NjA3OTY2N18xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

### 挂载\&卸载DCU

# 工作笔记

## 镜像格式

> [镜像文件从raw格式转换为qcow2](https://tinychen.com/20240624-qemu-img-to-qcow2/)
> 
> [KVM 磁盘格式 raw、qcow2 \- 命令记不住](https://www.opsxdev.com/articles/2021/10/08/1633706883061)
> 
> 

- raw 格式文件是**原始磁盘镜像**，它直接复制了物理硬盘的扇区数据，没有经过任何压缩或转换。因此，raw 文件大小与它所代表的虚拟磁盘大小（如根磁盘vda）完全相同，读写速度最快。

- qcow2 格式文件是 **QEMU 镜像文件格式**，它是一种**精简配置的磁盘镜像**，支持多种压缩算法，可以有效减小文件大小。因此qcow2 文件远小于虚拟磁盘大小，这样相当于把宿主机的存储容量超分。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZTA1ZThkYjg5ZmJiN2Q4Mjk3ZjlkNTUxNzc3NDVmYmFfZTY4NzMzOGIyMTVkZThjNDMwOThjOWI1MWE5NGExNGNfSUQ6NzY0ODE5OTA0NDU4NTgwMjk0OV8xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

在原来的盘上追加空间：

```Bash
# 先创建4G的空间
dd if=/dev/zero of=zeros.raw bs=1024k count=4096

# 追加到原有的镜像之后
cat foresight.img zeros.raw > new-foresight.img

```

## 实例创建流程

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YTg3ZmY0MzU2ZWFiYzcwYTM2ZTY5MGQ0MzQ0NDE2NjVfZmJiODQ5M2I1NWIwYzI4NDQwNTQzNjJkNDA1NTVhNWNfSUQ6NzY0ODE5OTA0NDI1MDE0Mzk0N18xNzgxMzY5MjA1OjE3ODE0NTU2MDVfVjM)

## 实例启动过程

8. 加载内核态，完成内核相关初始化工作

9. 内核启动后，`systemd` 作为第一个用户态进程启动，负责初始化操作系统相关服务和资源，包括文件系统挂载、NetworkManager服务启动、SSH 服务启动等

10. `NetworkManager`成功从DHCP服务器获取IPv4地址 \(`192.168.100.5`\)，并配置了子网掩码、网关和DNS。

11. `fs-agent` 在启动和运行过程中完成实例自定义配置（类似[cloud\-init工具](https://help.aliyun.com/zh/ecs/user-guide/manage-the-instance-initialization-configuration?spm=a2c4g.11186623.0.0.7c0b71ffIt4hcM)），包括修改hostname、修改密码、扩展卷、ntp同步、配置内网IP等

