# 云硬盘 EVS

# 云硬盘 EVS

**云硬盘**为云服务器提供的块存储设备，采用多副本的分布式机制，具有低时延、高性能、持久性、高可靠等特性，可以随时创建或释放，也可以随时扩容。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OTk5YWJkYjdlOWEwOTY3M2U2NGY0YTA5MDlkNTFlNDRfOTc1OTQ0YmU4NGJkNDUyMWQ0ODQ5NWZkOWZlYmZhYzRfSUQ6NzY0ODE5OTA3OTMyNzE5MDIxMF8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

## 核心能力

- **弹性可扩展**: 用户可以根据业务需求灵活地调整云硬盘的容量，支持扩容操作，无需暂停业务即可实现平滑扩容。

- **共享能力**:云硬盘可设置允许多挂载，允许多个弹性云服务器并发访问同一块云硬盘，适用于需要集群和高可用能力的关键企业应用场景

- **安全性**:云硬盘提供快照、克隆、镜像功能，便于数据快速恢复和历史版本的保存，为用户提供灵活的数据安全保障方案。

## 功能操作

### 挂载\&卸载云盘

云硬盘作为块存储，不能被操作系统应用直接访问；需要挂载到云服务器实例中，格式化成文件系统进行访问。可参考以下文档进行硬盘分区挂载

[初始化数据盘\_云服务器 ECS\(ECS\)\-阿里云帮助中心](https://help.aliyun.com/zh/ecs/user-guide/initialize-a-data-disk/?spm=a2c4g.11186623.help-menu-25365.d_4_2_1_3.1e741cc5KdtMlJ&scm=20140722.H_108498._.OR_help-T_cn#DAS#zh-V_1)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZmU1M2MwNjgzMzFiMDU0YzdhMDAyN2NmYmQ3ZDc3ZWFfMmIxMmI4ZThiYzk4M2ExMGY3MjhhM2ZhM2I3YmUwY2NfSUQ6NzY0ODE5OTA3OTM4MTgxNDQ2Ml8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

### 云硬盘扩容

1. 在ECS控制台上扩容云盘容量后，对应分区和文件系统并未扩容，如图所示；

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ODkwMjkyNWYxMmVkOGIyMGZlYTIwODZkMmJkM2Y2OWJfODBkZDdjNTg2YzJmOWMyZjkzMmViMTkwYmRhZmYxOTRfSUQ6NzY0ODE5OTA3NzU4MjQ1ODA3MV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

2. 需要进入ECS实例内部继续[扩容云盘的分区和文件系统](https://help.aliyun.com/zh/ecs/user-guide/step-2-resize-partitions-and-file-systems/?spm=a2c4g.11186623.help-menu-25365.d_4_2_3_2.25d33a8aLBodmH&scm=20140722.H_470068._.OR_help-T_cn#DAS#zh-V_1)，扩容有以下两种场景

    1. 云盘已分区，将扩容部分的容量划分至已有分区，需要扩容分区和文件系统

    2. 云盘已分区，将扩容部分的容量划分至新增分区，需要扩容分区和文件系统

    3. 云盘未分区（裸设备），直接扩容文件系统

### 使用系统盘快照创建镜像

通过快照创建自定义镜像，您可以将一台ECS实例的操作系统、数据制作成环境副本，再通过自定义镜像创建多台ECS实例，快速复制应用环境。

使用限制：

- 仅系统盘快照支持创建自定义镜像。

- 使用创建的自定义镜像创建ECS实例后，[部分VPC网络可能出现网络无法连通](https://help.aliyun.com/zh/ecs/how-to-solve-unreachable-network-errors-when-a-vpc-type-instance-is-created-from-a-custom-image?spm=a2c4g.11186623.0.0.298afe7cVEZgDI)的异常情况，主要与`/etc/sysconfig/network`的配置相关

### 使用数据盘快照创建云盘

为云盘创建快照，再使用快照创建云盘获取基础数据，实现同城容灾和异地容灾

## 与其它服务的关系

# 硬盘性能基准测试

> [如何在Linux实例中使用FIO工具测试块存储性能\_云服务器 ECS\(ECS\)\-阿里云帮助中心](https://help.aliyun.com/zh/ecs/user-guide/test-the-performance-of-block-storage-devices?spm=a2c4g.11186623.help-menu-25365.d_4_2_5_2.475e29edYhr2aR)
> 
> 

## 性能指标

3. **每秒读写次数 IOPS**

IOPS是指单位时间内系统能处理的I/O请求数量，一般情况下指的是4kb随机I/O的读写性能。

单队列深度串行操作时，`IOPS= 1s/ 平均访问时延`公式可近似估算IOPS，不适用于多队列或并发操作。

4. **吞吐量 Throughput**

吞吐量是指单位时间内能够读写的数据数量，单位为MB/s。`吞吐量（bw） ≈ IOPS × 块大小`

5. **访问时延 Latency**

访问时延通常指的是单个I/O操作从发起到完成所需的时间，包括寻道时间、旋转延迟和传输时间等

## fio命令参数

## 硬盘性能预估

HDD 和 SSD 的性能会因**品牌、型号、接口类型（SATA、NVMe）、负载模式** 而有所不同，但通常有一个大致的性能区间。

### 硬盘信息

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZmMyOGI3Y2Q4NjVjNDI4MTg3ODEzZTE0NzlhOTBmZTVfN2NhN2FjMDUzODU4NDI0NGYyMTM0MWVkNTdmODFjMDZfSUQ6NzY0ODE5OTA3NzMwMTcwMTgzOV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

> **型号**：HGST HUS726T6TALE6L4
> 
> **容量**：6TB
> 
> **旋转速率**：7200 RPM（比 5400 RPM HDD 速度更快）
> 
> **接口**：SATA 3\.2（最大 6\.0 Gb/s，即 600 MB/s，实际远达不到）
> 
> **扇区大小**：4K 物理扇区
> 
> **用途**：适合大文件存储、备份、日志存储等应用场景
> 
> 

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NjgxMGMzMjRkMWZjZGRmNzhlNzQzNjc1ZmQ0YWFlOTdfMjMyNjE0MDg3M2Q0ZDMyMjQ0NTI4ZTM2ZWEwMmIyNGNfSUQ6NzY0ODE5OTA4MDk1NDM4MzU3OV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

> **型号**：SAMSUNG MZ7LH1T9HMLT\-00005
> 
> **容量**：1\.92 TB
> 
> **接口**：SATA 3\.2（最大 6\.0 Gb/s，即 600 MB/s）
> 
> **扇区大小**：4K 物理扇区
> 
> **用途**：适合 OLTP、数据库、虚拟化存储等场景
> 
> 

### 机械硬盘（HDD）性能区间

6. 顺序读写速度

    - 5,400 转 HDD​：​50–100 MB/s​（如西部数据 Green 系列）

    - 7,200 转 HDD​：**​100–200 MB/s**​（如希捷 Barracuda）

    - 企业级 HDD​（10,000/15,000 转）：​200–250 MB/s​（受接口限制）

7. 随机读写性能（4K IOPS）​

    - 5,400/7,200 转 HDD​：​**100–200 IOPS**​（受机械寻道限制）

8. 访问延迟

    - 典型值 5–15 ms​（毫秒级），受磁头寻道和盘片旋转影响

9. 适用场景

    - 大容量冷数据存储（如电影、备份）、预算有限场景

### 固态硬盘（SSD）性能区间

10. 顺序读写速度

    - SATA 接口 SSD​：

典型范围 **500–600 MB/s**​（如西部数据 WD Blue SN550、三星 860 EVO）

- NVMe 协议（PCIe 接口）​​：\- PCIe 3\.0 x4：​1,500–3,500 MB/s​（如三星 970 EVO Plus）\- PCIe 4\.0/5\.0 x4：高端型号可达 5,000–12,000 MB/s​（如 PCIe 5\.0 SSD）

11. 随机读写性能（4K IOPS）​

    - 普通消费级 SSD​：

        - 随机读取：​**50,000–100,000 IOPS**

        - 随机写入：​30,000–80,000 IOPS

    - 高端 NVMe SSD​：

        - 随机读取：​500,000–1,000,000\+ IOPS​（如企业级 SSD）

12. 访问延迟

    - 通常为 0\.01–0\.1 ms​（微秒级），远低于 HDD

13. 适用场景

    - 操作系统启动、游戏加载、数据库等高 IOPS 需求场景

### 影响性能的关键因素

14. **HDD**：

    - **转速**：5400 RPM（低速归档）\< 7200 RPM（通用）\< 10k/15k RPM（企业级）。

    - **数据密度**：高密度盘片（如 SMR）顺序写快，随机写差。

    - **碎片化**：碎片化严重时随机性能下降。

15. **SSD**：

    - **接口协议**：NVMe \> SATA。

    - **NAND 类型**：SLC \> MLC \> TLC \> QLC（性能/寿命递减）。

    - **写入放大**（WA）：随机写可能触发垃圾回收，降低实际性能。

## 测试建议

16. 直接对裸设备进行测试，避免文件系统缓存影响

17. 清除磁盘旧数据` dd if=/dev/zero of=/dev/sdX bs=1M status=progress`

18. 磁盘预热 `dd if=/dev/zero of=/dev/sdX bs=1M count=10000 oflag=direct`

19. 检查磁盘温度，理想温度：30\-45°C，超过 50°C 可能导致掉速 `smartctl -A /dev/sdX | grep Temperature`

20. 测试参数​：

    - 顺序读写（测BW）：块大小 128K–1M，队列深度 32–64。

    - 随机读写（测IOPS）：块大小 4K，队列深度 1（单线程）或 64（多线程）

    - 加上参数`--trim`，在每次fio测试前清理磁盘，确保磁盘为初始状态

## 实测结果

### 1M顺序写 （HDD）

`fio -direct=1 -iodepth=1 -rw=write -ioengine=libaio -bs=1M -size=10G -numjobs=1 -group_reporting -filename=/dev/sdl -name=mytest`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZjBjOWM2MTQwZjAxNjllZTI4YmQ3MGYwM2I4MzFkMmZfZTkwMDc1NGY1ZTQxNTNlYzBmZGJhOTMwMWQ0Njg2OThfSUQ6NzY0ODE5OTA4MTA2NzYxMzM4OV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

**关键数据解读：**

- **带宽 228MiB/s**（239MB/s），这是典型的 SATA HDD 写入速度，属于正常范围。

- **IOPS 228**，符合 1M 块大小的顺序写性能。

- **延迟平均 4\.3ms，P99 约 12\.5ms**，符合 HDD 的特性。

- **磁盘利用率**：达到 **99\.8%**，说明磁盘在全速写入，未出现瓶颈

### 4k随机写（HDD）

`fio -direct=1 -iodepth=1 -rw=randwrite -ioengine=libaio -bs=4k -size=10G -numjobs=1 -group_reporting -filename=/dev/sdl -name=mytest`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZWM0NDk5ZWU0NjFmYTE3MWUwYjY0MGIwNmIzYmRhMzNfZWNhMzY5NDliNGJiN2MyMjExYTNhMzljMTBmMDNlMzdfSUQ6NzY0ODE5OTA3ODk0MTQ3ODA5OV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

**关键数据解读：**

- **IOPS 536**，HDD 在随机写入时会受到磁头寻道时间影响，这个 IOPS 在机械盘上是合理的。

- **带宽 2\.1MB/s**，随机写入的 HDD 性能通常在 1\-4MB/s 之间，因此也符合预期。

- **P99 延迟 11\.3ms**，机械盘的寻道时间通常在 5\-15ms，所以这个延迟也正常。

### 1M顺序读写（SATA SSD）

`fio -direct=1 -iodepth=1 -rw=write -ioengine=libaio -bs=1M -size=10G -numjobs=1 -group_reporting -filename=/dev/sdc -name=mytest`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MGNmMmRkNzhkZWJjNmYzMjE4MjAzYzYwYTM5MWM5NTRfMzE5ODgxMGM3N2RiZWY1OTMzYjkwZjYzYzViMzdjMDVfSUQ6NzY0ODE5OTA4MTM3Mzc0ODQwMl8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

**关键数据解读：**

- 读取带宽实测 546MB/s，与预期 550MB/s 非常接近，性能正常

- 读取延迟（clat 平均值）= 1\.88ms，99% 的请求延迟低于 **2\.1ms**，仅 **0\.05% 超过 2\.6ms**，稳定性很好

### 4k随机读写（SATA SSD）

`fio -direct=1 -iodepth=32 -rw=randwrite -ioengine=libaio -bs=4k -size=10G -numjobs=4 -group_reporting -filename=/dev/sdc -name=mytest`

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MTM3M2EwNzlhODAwMjM3YTdiN2VlOThmNzUzYjFmMDRfMmRmMDRjMDg2ZjdmY2UxNGNmMGMyOGZkNzVhMmIxNTVfSUQ6NzY0ODE5OTA3ODU3NDAxNzc1NV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

# 工作笔记

## 本地存储池 localfs

http://172\.22\.5\.6:8090/pages/viewpage\.action?pageId=73412629

### 设计背景

随着云计算技术的快速崛起，越来越多的企业和个人开始将数据和应用程序迁移到云端，以实现更高效的数据管理和资源共享。然而，在某些情况下，完全依赖云端存储可能并不总是最佳选择。例如，对于需要快速访问的数据或对于对延迟敏感的应用程序，本地存储提供了更低的延迟和更高的访问速度。

其次，本地存储为数据备份和恢复提供了额外的保障。尽管云存储具有高可靠性和冗余性，但在某些极端情况下，如网络中断或云服务提供商的问题，依赖云端存储可能会导致数据访问受限。此时，本地存储可以作为备份选项，确保数据的可用性和业务的连续性。

此外，本地存储还有助于减轻云服务的负载。通过将部分数据存储在本地，可以减少对云服务的依赖和网络带宽的消耗，从而降低运营成本并提高性能。

综上所述，云计算中本地存储的背景主要源于对**快速数据访问**、**数据备份**和恢复以及减轻云服务负载的需求。本地存储将在云计算环境中发挥重要作用，为用户提供更加灵活和可靠的数据管理方案。

### 整体架构

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YjgxODMxMDgyNjEyZmE0YTNjMTgzM2VhNWE2ZTNlN2VfMWY3NjQwZTAxZjAzYzc5MGMwOWFiYmYwNzNiYTg3MDJfSUQ6NzY0ODE5OTA4MDQ0Njg3Mjc1Nl8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

- localfs跟xstor，zbs、xsky等存储形态近似，与云平台解耦，单独提供存储服务，并统一由localfs\-api对外提供接口服务，并将系统盘和数据盘操作调度到具体计算节点。

- localfs\-node内置qemu img等基础组件，每个组件如Gova，Cinder，Golance等以plugin形式集成相关操作，负责接受来自LocalFs\-API的调度请求，并将请求转换成qemu、virsh等相关存储指令执行；

约束：

1\.创建数据卷必须指定计算节点；

2\.虚拟机挂载的数据卷必须在同一计算节点；

3\.虚拟机不支持冷迁移；虚机挂载其它存储池里的云盘时，不支持热迁移；

4\.暂只支持qcow2；

### 注意事项

在只有本地存储场景下，由于云主机的数据（系统盘和数据盘）存储在每台服务器自身存储上，需要重点关注：

21. 镜像建议使用qcow2格式（支持ISO创建虚拟机），相比raw镜像，对本地存储占用空间会相对较少

22. 本地存储可以由一块硬盘或多块硬盘构成，结合单块硬盘存在单点异常丢数据风险，建议对每节点上的系统盘采用Raid1方案，所有数据盘采用Raid5生成一块本地存储盘的方案；

23. 本地存储目录需手动挂载到宿主机上并持久化，请将挂载命令写入到/etc/rc\.local中，其中磁盘一定要写UUID，/localfs/sugon\-test就是本地存储目录

    1. 首先确认rc\.local是否有执行权限，没有可以执行：chmod \-R 777 /etc/rc\.local

    2. 具体挂载

## Lun克隆与lun拷贝

目前环境参数设置如下，配置在终端执行以下命令查看 kubectl describe cm/golance\-config \-n openstack

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YzkwYzE5MjA5ZWM1MGY4Mjk4MjIyYmQ4NTY5Y2Y2MzBfM2RhZTY0OWZkZmM3Mjg1OTYzZTA5YjQ5MzU0OTI1OTdfSUQ6NzY0ODE5OTA4MTA2NzYyOTc3M18xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

1、镜像lun的克隆次数小于image\_cache\_clone\_limit ，创建虚机会直接走源lun克隆的流程

2、镜像lun的克隆次数大于image\_cache\_clone\_limit，自动创建新的image\-cache\-lun，即lun拷贝。再次创建虚机会基于新的cache lun，走lun克隆的流程，从而达到分流的效果

有xstor web界面的，可以在lun管理界面，查看lun克隆层次，可以找到lun克隆了哪些系统盘

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MWY4YjA0OGIyOGM4MzA5ZmM3MDQ3YTE3ZDBmMmMxZTVfN2I0OGQxYzJkYjZmYzJjNDE2ZGQ4YzRlNTU3Y2FmZTRfSUQ6NzY0ODE5OTA4MDE0MDg2ODc5NF8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

## 系统盘\&数据盘配置定义

### xstor虚机

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YTFkNjkwMDdlZDZkMDAyOTFjYTQzYTlmZmU3ODQzM2NfM2M0MzBmY2RiZjM5NDUwNGFiZDdiYjAwNWE1ZWIwZDZfSUQ6NzY0ODE5OTA3OTM2OTI4MDcwOF8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

查看系统盘、数据盘在xstor里的存储位置

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MTk0YjhkMjcxNDUyMDFlY2Y1MzM2ZTBmMWY4NGM2M2FfOGJmZTg2MGQ3NDEyZTY3MzdiNGYwMWFiMmFiM2NmNWVfSUQ6NzY0ODE5OTA3OTI0NzU4MDMzOV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

### zbs虚机

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MWY1Y2JjNzk3NDY5M2EyYWVhNmE2ODIyYWNjNjI2YzdfYWE4MjUxNTgxOTA3MDE0NDkxNzllZjhlMzhmZjMwM2RfSUQ6NzY0ODE5OTA3ODYxMDEyODExNV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

### xsky虚机

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NDlhMThkZWM3OTE1ZDU2YzVjNGMzNGE2ZjViMGUwYTlfYmUyZWQ5Njc0N2IyZjJkYjJlY2UwNWZlMGUwYzkzNzJfSUQ6NzY0ODE5OTA3Nzg4NDQ0Nzk0Ml8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

### usan虚机

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MDQ1ZDY5NjBkOGM2MWI0YzAzZjIyZWI0ZjY2OGEyMGNfZWRmZmNhZGY3M2FkNGI2Y2ZiMDMwZWZlZTViN2E4MTlfSUQ6NzY0ODE5OTA3OTgwOTUwMjQwNV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

### localfs虚机

- 镜像缓存目录：`/localfs/localstorage/imagecache/` 该目录存放镜像（qcow2格式），是所有localfs虚机系统盘的后端基础映像

- 卷目录：`/localfs/localstorage/volume/` 该目录存放虚机的系统盘\+数据盘\+快照文件

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MTk0ZWE0OTYwYWJhNTliOTAwYWE0OWNjNzMxNTVhYzVfMmRkODA1MTA1MWE1MmRkMmJhZTZjNzhlYzMwY2VkODBfSUQ6NzY0ODE5OTA3ODg5MTExMzY3OV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

### xbd虚机

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NTg3YmJlMDZiMzg0NzNhZDliZmI2MDg5Mzc4ZTJjZGFfN2I0YjQ0MjM2YTJhZmJhNTMwNjNiZDY0MzI0MmZjZDZfSUQ6NzY0ODE5OTA3OTQ0MDM4NzI1OF8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

### 总结

- localfs、usan支持raw、qcow2格式镜像创建虚机，其余共享存储只支持raw格式镜像

- localfs、usan虚机系统盘是基于镜像文件的增量快照（file、block），镜像文件路径通过参数定义

- usan、zbs、xstor虚机挂载数据盘，均会在其宿主机创建一个block设备（如dev/sdl）

# 知识拓展

## 存储逻辑概念及架构

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NDA4NmEzZTUzMGUyNGFmZThkYjBkOTVmNmQ0ZDIzYmJfN2MzY2JiOWFiZWYyMmYxODNjNjczYzIwMDM1MWYyYWVfSUQ6NzY0ODE5OTA3ODU2ODE2ODY1Ml8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

## 存储接口协议 \{folded="true"\}

存储接口协议对存储介质的性能有直接影响，因为接口决定了数据传输的速度、延迟和效率。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NjE3NTM2YWMzOWEyOWE1YTRiMTc1MjYwODQzNDUxMzJfOTg4NzA2NzRkYjQ1ZDhiZDU1YTIxNzNlM2E2YTEzZTlfSUQ6NzY0ODE5OTA3NzcxNzAxOTgzOV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

### USB \(Universal Serial Bus\)

- **特点：**

    - 外置 HDD 常用接口协议。

    - 传输速率取决于 USB 版本：

        - USB 2\.0：480 Mbps（60 MB/s）

        - USB 3\.0：5 Gbps（500 MB/s）

        - USB 3\.1 Gen 2：10 Gbps（1 GB/s）

        - USB 4\.0：40 Gbps（5 GB/s）

- **优势：** 插拔方便，适合便携式存储。

- **限制：** 数据传输速度通常低于 SATA 和 SAS。

### iSCSI \(Internet Small Computer System Interface\)

- \*\*定义：\*\*iSCSI 是一种基于 TCP/IP 协议的存储网络协议，用于通过标准以太网将存储设备（如硬盘阵列、SSD、存储服务器）呈现为远程服务器的本地磁盘。它通过 IP 网络传输 SCSI（Small Computer System Interface）命令，从而实现存储设备的远程访问。

- **特点**：

    - \*\*基于 IP 网络：\*\*iSCSI 不需要专用存储网络设备（如光纤通道），可以运行在普通以太网上，适合各种规模的存储系统。

    - \*\*远程存储呈现：\*\*使用 iSCSI，远程存储设备可以映射为服务器的本地磁盘，支持常规文件系统操作。

    - \*\*兼容性强：\*\*由于基于标准 SCSI 和 TCP/IP，iSCSI 可与多种存储设备和网络设备兼容。

- **限制：**

    - iSCSI 的性能受限于网络带宽、延迟和抖动，与专用存储网络（如光纤通道）相比较慢。

    - iSCSI 需要占用主机 CPU 来处理 TCP/IP 协议栈，增加了服务器的计算负载。

    - 对于低延迟、高带宽的应用场景（如数据库或虚拟化），iSCSI 性能可能无法满足需求。

- **工作原理**

    1. **SCSI 转换为 TCP/IP**

iSCSI 协议将传统 SCSI 指令封装为 TCP/IP 数据包，通过 IP 网络传输至目标存储设备。

24. **目标设备解析指令**

存储设备（iSCSI Target）接收到 TCP/IP 数据包后，解析出其中的 SCSI 指令，并在本地执行。

25. **反馈数据**

数据返回时，存储设备将响应封装为 TCP/IP 数据包，通过网络传回客户端（iSCSI Initiator）。

\<image token="Oxoub8lZIoV3YfxzCZKcXJpKnMc" width="692" height="458" align="center"/\>


### SATA \(Serial ATA\)

- **特点：** 设计用于 HDD，后来被用于 SSD。

- **速度：**

    - SATA I：1\.5 Gbps（150 MB/s）

    - SATA II: 3\.0 Gbps（300 MB/s）

    - SATA III: 6\.0 Gbps（600 MB/s）

- **优势：**

    - 兼容性强，适用于桌面电脑、笔记本、NAS 和服务器。

    - 成本低，技术成熟。

- **限制：** 带宽相对较低，适合传统 HDD 性能需求。

### SAS \(Serial Attached SCSI\)

SAS\(Serial Attached SCSI\)即串行连接SCSI，是新一代的SCSI技术，此接口的设计是为了改善存储系统的效能、可用性和扩充性，并且提供与SATA硬盘的兼容性。

- **特点：** 面向企业级存储设计，兼容传统 SCSI 协议，性能和可靠性更高。

- **速度：**

    - SAS 1\.0：3\.0 Gbps

    - SAS 2\.0：6\.0 Gbps

    - SAS 3\.0：12\.0 Gbps

    - SAS 4\.0：22\.5 Gbps（最新标准）

- **优势：**

    - 支持多设备连接（高达 128 个设备）。

    - 提供双端口冗余，适合高可靠性需求。

    - 具备更好的数据完整性和错误检测功能。

- **场景：** 数据中心、企业存储阵列。

### Fibre Channel \(FC\)

光纤通道\(Fibre Channel\)是一种高速网络技术，可以有序提供无损的原始数据块数据。该技术定义了多个通信层，用于使用光纤通道协议\(FCP\)传输SCSI命令和信息单元。FC和SCSI接口一样光纤通道最初也不是为硬盘设计开发的接口技术，是专门为网络系统设计的，但随着存储系统对速度的需求，才逐渐应用到硬盘系统中。

- **特点：**

    - 企业存储系统中的高性能接口协议，通常用于 SAN（存储区域网络）。

    - 提供低延迟和高带宽。

- **速度：**

    - 常见速率：16 Gbps、32 Gbps。

- **场景：** 企业级存储和数据中心。

### NVMe（Non\-Volatile Memory Express）

- **定义：** 专为闪存（SSD）设计的新一代存储协议，利用 **PCIe 总线**进行通信。

- **主要特点：**

    - 直接与闪存通信，充分发挥 SSD 性能。

    - **带宽高：** 基于 PCIe，速度可以达到 **32 Gbps**（约 4 GB/s）或更高，具体取决于 PCIe 通道数（如 PCIe 3\.0 x4、PCIe 4\.0 x4 等）。

    - **低延迟：** 通过并行指令队列大幅减少 I/O 等待时间。

    - **接口类型：** 使用 **M\.2** 或 **U\.2** 接口连接。

众所周知，固态硬盘（SSD）拥有比机械硬盘（HDD）更快的读写速度。目前大多数机器使用的是SATA总线标准，实际最高传输约为600MB/s。而支持PCIe总线\+NVMe协议的SSD，实际传输速度将超过1000MB/s。

PCIe 存储的出现比 NVMe 早几年，但以往的解决方案受到 SATA 和 AHCI 等较旧数据传输协议的瓶颈限制，导致 PCIe 存储无法发挥全部潜力，NVMe 正是这种瓶颈的解决方案，提供低延迟命令和 64000 个队列，消除了各种限制因素。多队列设计可以提高数据传输速度，因为数据是利用芯片和块以分散形式写入 SSD 的，而不是像机械硬盘一样在旋转的磁盘上写入数据。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZDY1NGIzY2UzOWYyZTFlYTdjZDdjNDIyYzNhOTYzNzRfYmM2ZDliZTJmNDFkNmVlMWEwOTdlNGRmZWJiYzFmYWFfSUQ6NzY0ODE5OTA4MTA5NzQzMjI4NV8xNzgxMzY5MTgyOjE3ODE0NTU1ODJfVjM)

