# OpenStack

# OpenStack

OpenStack的各个服务之间通过统一的REST风格的API调用，实现系统的松耦合。它内部组件的工作过程是一个有序的整体。诸如计算资源分配、控制调度、网络通信等都通过AMQP实现。Open Stack的上层用户是程序员、一般用户和Horizon界面等模块。这三者都是采用OpenStack各个组件提供的API接口进行交互，而它们之间则是通过AMQP进行互相调用，它们共同利用底层的虚拟资源为上层用户和程序提供云计算服务。 

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YzY1ZmIyYTcxMmY5NmIzODgwNDFkNzI1MzUxZjA4NzBfZWIyMmM0Nzc1MThkYmQyMmZjZDI3MTE3YWRlNmIxYjNfSUQ6NzY0NjMxMDU5MTY3Mzk5NDQ1Ml8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

## Nova

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZDYxM2VkNjMyYzYyN2U1NTU3NzM0YWEyOTcwOTc0YmNfOThmMWVmNGU2YWQwY2VlOGYxNGJjMzhlMjk2Y2Q5ZmZfSUQ6NzY0NjMxMDU5Mjc3MjU5MDc3OF8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

**nova\-api**

nova\-api 对接收到的 HTTP API 请求时，会检查客户端传入的参数是否合法有效，配额是否足够；当检查通过后，nova\-api 就会为该请求分配唯一的虚拟机 ID ，并在数据库中新建对应的项来记录虚拟机的状态；然后，nova\-api 会将请求发送给 nova\-conductor 处理。



**nova\-api\-metadata**

接受来自实例的元数据请求，instance 启动时向 Metadata Service 请求并获得自己的 metadata，instance 的 cloud\-init 根据 metadata 完成个性化配置工作。比如：安装某些包、添加 SSH 秘钥、配置 hostname 等等。



**nova\-conductor**

出于安全性和伸缩性的考虑，nova\-compute 并不会直接访问数据库，而是将这个任务委托给 nova\-conductor，充当数据库代理或处理对象转换（比如更新虚机状态）。



**nova\-scheduler**

从消息队列中获取虚拟机实例请求，并决定在哪个计算节点上创建虚机。

Filter scheduler 是 nova\-scheduler 默认的调度器，调度过程分为两步：

1. 通过过滤器（filter）选择满足条件的计算节点（运行 nova\-compute）

2. 通过权重计算（weighting）选择在最优（权重值最大）的计算节点上创建 Instance。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZmY4NGNiYmY3MWM1YWU1Y2E5MWVlMzY3Y2YyNDY3MjRfOTFjMmEwZGM5ZGY0NTNjMjI2MTczMjQzMTFmYTAxMjJfSUQ6NzY0NjMxMDU5MDA2MTI1MTgwOF8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)



**nova\-compute**

1. 定时向 OpenStack 报告计算节点的状态（通过hypervisor来获取主机资源信息）

2. 通过调用 Hypervisor API 实现虚机的生命周期管理。例如 KVM/QEMU 的 libvirt、VMware 的 VMwareAPI

> 我们可以在nova\-compute容器目录下查看到 nova源代码中已经自带了上面这几个 Hypervisor 的 Driver。
> 
> 

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MjhmMjk3Y2UxZWM1YmU4Zjk0MTU5OTE0ODAxYzUxYWZfODhlOTFlMmYzYTlhZDc3YWQ1MjI0MmNmMTgwOWQyNGJfSUQ6NzY0NjMxMDU5MTIyMTA5MTU0N18xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

nova\-compute 创建 instance 的过程可以分为 4 步：

1. 为 instance 准备资源（根据规格flavor准备内存，cpu，硬盘等）

2. 创建 instance 的镜像文件（通过glance下载p\_w\_picpath并创建instance的镜像文件）

3. 创建 instance 的 XML 定义文件

4. 创建虚拟网卡并启动虚拟机



**nova\-placement\-api**

在Newton版中，Nova正式引入了Placement API，用于管理和查询资源提供者的资源存量、使用情况、分配记录等等，以提供更好、更准确的资源跟踪、调度和分配的功能。

在整个 OpenStack 资源体系中不仅只有 compute node 提供的计算资源（CPU/RAM），还可能存在各类外部存储和外部网络资源，例如：Ceph、NFS 提供的存储服务，SDN 提供的网络服务，这些资源由外部系统提供。可见，当 resources provider 变得多样时，自然会需求一种简单且统一的管理方法，让管理员得以清晰、便捷的掌握资源系统中含有的资源类型及其使用情况，这就是 Placement API。



**nova\-console**

用户可以通过多种方式访问虚机的控制台：
 nova\-novncproxy，基于 Web 浏览器的 VNC 访问
 nova\-spicehtml5proxy，基于 HTML5 浏览器的 SPICE 访问
 nova\-xvpnvncproxy，基于 Java 客户端的 VNC 访问



**nova\-consoleauth**

负责对访问虚机控制台请求提供 Token 认证

### 常用命令

在实例的生命周期管理流程中，nova常见的一些操作如下：

- 创建：nova boot

```Python
nova boot --flavor ecs.c6.large --image af51149a-03f9-41c7-8d62-8ebc1833e260 --security-groups b9cbdd47-f75d-473f-8037-8259e430da36 --availability-zone nova --nic net-id=327c76f5-e310-49e2-afa8-ab9e74c366ee test-vm
openstack server create --flavor ecs.c6.large --image af51149a-03f9-41c7-8d62-8ebc1833e260 --security-groups b9cbdd47-f75d-473f-8037-8259e430da36 --availability-zone nova --nic net-id=327c76f5-e310-49e2-afa8-ab9e74c366ee test-vm
```

- 重启：nova reboot

- 启动：nova start

- 停止：nova stop

- 暂停：nova pause

- 取消暂停：nova unpause

- 挂起：nova suspend

> pause与suspend的区别在于pause将instance的运行状态保存在计算节点的内存中，而suspend保存在磁盘上。pause的优点在于resume速度比suspend快，缺点是如果计算节点重启，内存数据丢失，则无法resume。
> 
> 

- 恢复：nova resume

- 搁置：nova shelve

> shelve 操作会将 instance 作为 image 保存到 Glance 中（命名为\<instance name\>\-shelved），然后在宿主机上删除该 instance，从而释放instance占用的资源。
> 
> 

- 取消搁置：nova unshelve

> unshelve 的过程其实就是通过 image 创建一个相同的instance，instance 的其他属性（比如 flavor，IP 等）不会改变。
> 
> 

- 删除：nova delete

- 调整实例：nova resize，

> 调整 instance 的 vCPU、内存和磁盘资源，Resize 是在 Migrate 的同时应用新的 flavor
> 
> 

- 冷迁移实例：nova migrate

> 代码与nova resize相同，migrate可以看作未修改flavor的resize。
> 
> 

- 热迁移实例 nova live\-migration，

> 与 Migrate 不同，Live Migrate 能不停机在线地迁移 instance，保证了业务的连续性。Live Migrate 分两种：
> 
> 1. 源和目标节点没有共享存储，instance 在迁移的时候需要将instance 镜像文件从源节点拷贝至目标节点，最后删除源节点的instance 镜像文件
> 
> 2. 源和目标节点共享存储（比如nfs），instance 镜像文件就不需要迁移，只需要将 instance 的状态迁移到目标节点
> 
> 

- 拯救： nova rescue

> 用于修复受损的系统盘、root密码遗忘场景，通过指定 image 作为启动盘引导 instance，将 instance 本身的系统盘作为第二个磁盘挂载到操作系统上。成功修复后，通过 unrescue 正常启动 instance。
> 
> 

- 快照：nova image\-create

> 有时操作系统损坏得很严重，通过 rescue 操作无法修复，这时就得考虑通过备份恢复了。对运行的虚机（主要是备份系统盘）创建一个快照镜像，直接上传到Glance中。执行步骤如下分为以下几步：
> 
> 1. pause instance，目的是暂停所有 IO 操作，保证数据一致性
> 
> 2. 执行 qemu\-img convert 命令复制 disk 文件，生成快照文件
> 
> 3. resume instance
> 
> 4. 将快照文件上传到 Glance
> 
> 

- 重建：nova rebuild

> 应用于恢复故障虚机的场景。通过 snapshot 替换 instance 当前的镜像文件，同时保持 instance 的其他诸如网络，资源分配属性不变。
> 
> 

- 疏散：nova evacuate

> 可在计算节点发生宕机故障，OpenStack 无法与节点的 nova\-compute 通信的情况下将节点上的instance 迁移到其他计算节点上。但有个前提： Instance 的镜像文件必须放在共享存储上。
> 
> 

### 问题解答

**思考1：为什么设计了API而不是直接向nova\-compute下发创建请求？**

不光是nova，周边的组件包括存储（cinder），网络（neutron），镜像（glance）都设计了api，api作为组件唯一对外的窗口，客户能且只能向api发送REST请求。设计api的好处在于：

1）统一对外提供接口，而让用户不必了解隐藏细节。

2）通过标准调用服务，便于三方系统集成和解耦。

3）可以运行多个api来实现高可用。

**思考2：为什么使用消息队列机制？**

消息队列采用异步调用的方式，api发送请求后无需等待直接返回，同理，scheduler和compute也同样如此。在分布式系统中，通常采用异步调用的方式。其好处是

1）解耦各子服务，下游服务不需要知道上游服务在哪里运行，只需要接收到消息便可以完成调用。

2）提高性能，调用者无需等待结果便直接返回，这样可以执行更多的操作，提高系统的吞吐量。

3）提高伸缩性，子服务可以根据需求进行拓展，启动更多的实例处理更多的请求，在提高可用性的同时也提高了整个系统的伸缩性。而且这种变化不会影响到其他子服务，也就是说变化对别人是透明的。

**思考3：创建什么样的虚拟机？**

虚拟机的创建使用的是hypervisor的接口，在多种hypervisor的情况下，包括KVM，Docker，Xen等，compute怎么知道创建何种虚拟机呢？openstack便是通过Driver框架实现。Nova\-compute 为这些 Hypervisor 定义了统一的接口，hypervisor 只需要实现这些接口，就可以 driver 的形式即插即用到 OpenStack 中。

**nova\.conf文件指定了计算节点使用哪种hypervisor的driver，如图所示，在我们的环境中因为是 KVM，所以配置的是 Libvirt 的 driver。**

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MGFjZjUwNTYzOTQ0Mzg1YWE4ZGY5ZDgxNTM3OTFkMDBfMWUzYmQ5ZGRkMjAzNmE4MjViYWVhNDRiYTlmMDVlNjdfSUQ6NzY0NjMxMDU5MTMyMTYwNzM0OV8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

**思考4：为什么nova\-compute不直接和数据库交互？**

Nova\-compute需要获取和更新数据库的时候并不会直接访问数据库，而是通过nova\-conductor来访问。这样做有几个好处

a\)更高的安全性。如果nova\-compute如果直接访问数据库，需要在组件中配置数据库的访问信息，而任意一个计算节点被攻击造成信息泄露，都会导致控制节点的数据库被影响。而通过nova\-conductor访问可以避免compute直接访问数据库，降低了数据的受灾面，提高了系统的安全性。

b\)更好的伸缩性。在同一时间，数据库能够访问的次数是有限制的，在大规模的openstack使用场景下，如果所有的计算节点都直接访问数据库，会很快到达数据库能够访问的临界点，造成崩溃。而通过nova\-conductor访问，管理员可以通过增加nova\-conductor来应对日益增长的数据库的访问量，从而提高系统的伸缩性。

## Neutron

Neutron 的设计目标是实现“网络即服务（Networking as a Service）”。为了达到这一目标，在设计上遵循了基于 SDN 实现网络虚拟化的原则，在实现上充分利用了 Linux 系统上的各种网络相关的技术。比如Linux Bridge、Open vSwitch、iptables。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YWI2YmMwYWI4ZWY0MTI4YzRmODBhZmViMzhjMTQ4YjlfODkxNDVlYzQxODMwNWY2ZGIyNTU0OTI4NjI2M2ZiMzlfSUQ6NzY0NjMxMDU5MDA0ODc4MzU3Ml8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)



**Neutron Server **

Neutron\-sever负责接收外部请求（比如nova\-api创建网络），将不同的api发送到不同Neutron plugin。



**Neutron plugin**

neutron 插件，由core plugins（核心插件）和service plugins（服务插件）组成。担任类似接收请求派发任务的角色，将具体要执行的业务操作和参数通知给对应的agent来执行，并且同时向database完成一些注册信息

> **注意：两种插件core plugin、service plugin已经合入到neutron\-server中。**
> 
> 



**Neutron Agent**

Agent负责具体的网络创建，接收相应的plugin通知的业务操作和参数，并转换为具体的命令行操作。



**ML2 Core Plugin**

Moduler Layer 2（ML2）是 Neutron 在 Havana 版本实现的一个新的 core plugin，用于替代原有的 linux bridge plugin 和 open vswitch plugin。

Neutron 可以通过开发不同的 plugin 和 agent 支持不同的网络技术。这是一种相当开放的架构。不过随着支持的 network provider 数量的增加，开发人员发现了两个突出的问题：

1. **只能在 OpenStack 中使用一种 core plugin，多种 network provider 无法共存。  **

2. **不同 plugin 之间存在大量重复代码，开发新的 plugin 工作量大。**

ML2 作为新一代的 core plugin，提供了一个框架，允许在 OpenStack 网络中同时使用多种 Layer 2 网络技术，不同的节点可以使用不同的网络实现机制。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NzY0NGUwMDcwYWVjNTlkNTYyOWM3MWQwYTVhZWVmMGZfNGRhZGM3NWQyOWNmNDY2OTRhMzJkZmMzMGNlZTMxZDZfSUQ6NzY0NjMxMDU4OTY3Mzg2ODQ2Nl8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

如上图所示，采用 ML2 plugin 后，可以在不同节点上分别部署 linux bridge agent, open vswitch agent, hyper\-v agent 或其他第三方 agent。

有了 ML2，要支持新的 network provider 就变得简单多了：无需从头开发 core plugin，只需要开发相应的 mechanism driver，大大减少了要编写和维护的代码。

## Cinder

Cinder是OpenStack的块存储服务。它把块存储设备进行池化，并对终端用户提供自服务的HTTP API去请求和使用存储资源。这使得终端用户不需要知道存储部署在哪里，也不需要知道是哪种存储设备。   

总结来说，就是为虚拟机提供块存储，提供统一的API接口，屏蔽异构设备和产品。

Cinder的Volume Type对不同品质的资源进行分级（性能、可靠性、可用性、硬盘类型），每种级别的资源就有不同的服务水平\(Service Level\)。云平台管理员定义不同Volume Type的名字和属性，并跟后端存储\(Cinder\-volume与Backend\)进行一一对应。



![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MjRlNjk5ZmVjNGEwNTRkYzBhZjA2N2ZkZjBkZTY4Y2FfMWI1MzU3ZWE2MDc4YTIzNTdiMjM4NDM3ZjQ1YzY3ZTlfSUQ6NzY0NjMxMDU5MDk2OTYxMzUxN18xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

Cinder 包含如下几个组件：

**cinder\-api**
接收 API 请求，调用 cinder\-volume 。

**cinder\-scheduler**
scheduler 通过调度算法选择最合适的存储节点创建 volume。

**cinder\-volume**
管理 volume 的服务，与 volume provider 协调工作，管理 volume 的生命周期。运行 cinder\-volume 服务的节点被称作为存储节点。

cinder\-volume 可以同时支持多种 volume provider，每种 volume provider 通过自己的 driver 与cinder\-volume 协调工作。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OWE3MmVhYmI3NTgyYjE4MTA1OWIwMzg4YzZlN2M3MThfNTE5NTczNjE4YTQ3NGExNmUwMjJmYmFlODUwYzA3ZTZfSUQ6NzY0NjMxMDU5MDg1MjE0MDI0NF8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

**volume provider**
数据的存储设备，为 volume 提供物理存储空间。cinder\-volume 为这些 volume provider 定义了统一的 driver 接口，volume provider 只需要实现这些接口，就可以 driver 的形式即插即用到 OpenStack 中。下面是 cinder driver 的架构示意图：

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YmVhZTI3NGRhNTNkMjQyMDJkMTBiMzMyMjgxOTgyNjlfMmFiZGIxZjc4Mjg0NmE0NWNlN2IxNGRiZjA5NDAyMTlfSUQ6NzY0NjMxMDU5MDM0OTEzNTAzN18xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

## Glance

glance命令创建windows用户标准镜像：

```Python
glance image-create --visibility public --disk-format raw --container-format bare --min-disk 20 --property hypervisor_type=kvm --property purpose=kvm --property hw_qemu_guest_agent=yes --property os_type=windows --property architecture=x86_64 --property os_version=Windows10 --property os_bits=64 --file ./windows10-fs.raw --name windows10-fs --progress
```

**上传cirros镜像**

```Bash
# img镜像转raw镜像、qcow2镜像
qemu-img convert -O raw cirros-0.5.2-x86_64-disk.img cirros-0.5.2-x86_64-disk.raw
qemu-img convert -O qcow2 cirros-0.5.2-x86_64-disk.img cirros-0.5.2-x86_64-disk.qcow2

# qcow2格式镜像转换为raw格式镜像
qemu-img convert -f qcow2 -O raw cirros-0.5.2-x86_64-disk.qcow2 cirros-0.5.2-x86_64-disk.raw

# openstack命令上传raw镜像
openstack image create cirros-0.5.2 --public --min-disk 1 --container-format bare --disk-format raw --property hypervisor_type=kvm --property purpose=kvm  --property os_version=centos7.6 --property os_bits=64 --property architecture=x86_64  --file cirros-0.5.2-x86_64-disk.raw

# glance命令上传raw镜像
glance image-create --name  cirros-0.5.2 --visibility  public --min-disk 1 --container-format bare --disk-format raw --property hypervisor_type=kvm --property purpose=kvm  --property os_version=centos7.6 --property os_bits=64 --property architecture=x86_64 --file cirros-0.5.2-x86_64-disk.raw --progress
```

如下，img格式镜像本质上也是qcow2格式的。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YjUyZTAxNDUxYWQ4ZTlkMmYyODMxZjgyMDU1MzFkOGNfMGZkYTlkMTc1NWYxMjBiZWQ0OGFjOGUyOTUyNWVjZTRfSUQ6NzY0NjMxMDU5MjU0NjIxMzA1OF8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

## Magnum

Magnum是OpenStack中的容器编排服务引擎，向上提供统一API，向下异构兼容K8S，Mesos，Swarm等容器管理平台，是OpenStack与容器结合的官方正式项目。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MTYyN2YwNjg0ZGNmZDQxNzI0MGJiYjIwYTIyNmFhMGJfNTFmNTY2YjUzMzM5YmI0MDUwNjY5NGQwOGU4MjAyY2JfSUQ6NzY0NjMxMDU5MTI5NjczNjQ0NV8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

### 核心概念

- Baymodel：Baymodel是Flavor的一个扩展，Flavor主要是定义虚拟机或者物理机的规格，Baymodel主要是定义一个Docker集群的一些规格，例如这个集群的管理节点的Flavor，计算节点的Flavor，集群使用的Image等等，都可以通过Baymodel去定义。

- Bay：Bay在magnum主要表示一个集群，现在通过Magnum可以创建K8s和Swarm的Bay，也就是K8s和Swarm的集群。

- Node：主要是指Bay中的某个节点。

- Pod：是Kubernetes最基本的部署调度单元，可以包含多个Container，逻辑上表示某种应用的一个实例。

- Service：可以理解为是Pod的一个路由，因为Pod在运行中可能被删除或者IP发生变化，Service可以保证Pod的动态变化对访问端是透明的。

- Replication Controller：是Pod的复制抽象，用來做Pod的複製、Scale、HA、load balance

### 工作流程

- 第一步需要创建BayModel，就是为需要提供容器服务的bay创建一些集群的定义规格。

- 第二步就可以在第一步创建的BayModel基础上创建Bay了，用户可以选择使用Kubernetes或者Swarm。

- 第三步，当bay创建完成后，用户就可以通过调用Magnum API和后台的k8s或者swarm交互来创建container了。

BayModel实例：

```Python
[root@busybox-openstack-c49dc584d-bwmsg /]$ openstack coe cluster template show a86f5ec0-702d-4ed4-a45c-73f9837071b9
+-----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field                 | Value                                                                                                                                                                            |
+-----------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| insecure_registry     | -                                                                                                                                                                                |
| labels                | {u'availability_zone': u'default-az', u'kube_tag': u'v1.16.6-es2', u'log_volume_size': u'200', u'etcd_discovery_flavor_id': u'213', u'ingress_controller': u'nginx',             |
|                       | u'flannel_backend': u'host-gw', u'docker_volume_type': u'hdd', u'etcd_volume_size': u'5', u'high_performance_volume_type': u'', u'cloud_provider_type': u'internal',             |
|                       | u'k8s_keystone_auth_tag': u'1.18.0', u'monitor_volume_size': u'100', u'floating_ip_bandwidth': u'102400', u'boot_volume_size': u'40', u'log_retention_time': u'180d',            |
|                       | u'container_runtime': u'containerd', u'tiller_enabled': u'true', u'volume_az': u'default-az', u'etcd_volume_type': u'hdd', u'flannel_network_cidr': u'10.253.0.0/16',            |
|                       | u'monitoring_enabled': u'true', u'container_infra_prefix': u'hub.ecns.io/openstackmagnum/', u'master_lb_floating_ip_enabled': u'true', u'boot_volume_type': u'hdd',              |
|                       | u'tiller_namespace': u'tiller'}                                                                                                                                                  |
| updated_at            | -                                                                                                                                                                                |
| floating_ip_enabled   | True                                                                                                                                                                             |
| fixed_subnet          | -                                                                                                                                                                                |
| master_flavor_id      | 221                                                                                                                                                                              |
| uuid                  | a86f5ec0-702d-4ed4-a45c-73f9837071b9                                                                                                                                             |
| no_proxy              | -                                                                                                                                                                                |
| https_proxy           | -                                                                                                                                                                                |
| tls_disabled          | False                                                                                                                                                                            |
| keypair_id            | liaoxb11                                                                                                                                                                         |
| public                | False                                                                                                                                                                            |
| http_proxy            | -                                                                                                                                                                                |
| docker_volume_size    | 100                                                                                                                                                                              |
| server_type           | vm                                                                                                                                                                               |
| external_network_id   | b6ada451-0042-4f12-bd94-3f3523ff39d8                                                                                                                                             |
| cluster_distro        | fedora-atomic                                                                                                                                                                    |
| image_id              | 1468ba5c-5020-4df5-a9dd-1111529ebb63                                                                                                                                             |
| volume_driver         | cinder                                                                                                                                                                           |
| registry_enabled      | False                                                                                                                                                                            |
| docker_storage_driver | overlay2                                                                                                                                                                         |
| apiserver_port        | -                                                                                                                                                                                |
| name                  | cluster-template-20210819105716                                                                                                                                                  |
| created_at            | 2021-08-19T15:57:17+00:00                                                                                                                                                        |
| network_driver        | flannel                                                                                                                                                                          |
| fixed_network         | -                                                                                                                                                                                |
| coe                   | kubernetes                                                                                                                                                                       |
| flavor_id             | 221                                                                                                                                                                              |
| master_lb_enabled     | True                                                                                                                                                                             |
| dns_nameserver        | 8.8.8.8                                                                                                                                                                          |
| hidden                | False
```

### 总结

**Magnum自身作为一套 API 框架，本身调用其它的容器管理平台的 API 来实现功能。目前支持的后端包括 Kubernetes、Swarm和Mesos。**

如果说 Nova 是一套支持不同 Hypervisor 虚拟机平台的 API 框架，那么 Magnum 则是支持不同容器机制的 API 框架。**Magnum Conductor是整个项目的核心，首先通过Heat部署虚拟机实例或者裸机实例上。然后通过Cloud init在虚拟机实例或者裸机实例上，调用Kubernetes、Swarm或者Mesos部署容器集群。**

# 计算虚拟化技术

计算虚拟化技术的通用实现方案是在操作系统与硬件之间加入一个虚拟化软件层,将服务器物理资源抽象成逻辑资源,使得上层操作系统可以直接运行在虚拟环境上,并允许具有不同操作系统的多个虚拟机\(VirtualMachine,VM\)相互隔离,并行运行在同一台物理机上,从而提供更高的IT 资源利用率和灵活性。

每台虚拟机都是一个完整的系统,它具有处理器、内存、网络设备、存储设备和 BIOS。在虚拟机中运行的操作系统软件称为虚拟机操作系统 GuestOS。计算虚拟化的这个软件层,也就是虚拟机监控器 \(Virtual Machine Monitor,VMM\),通常被称为Hypervisor。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MWM5NWNlMWJmZmIzOWE5MmVjOTgyYmE5MDA1ODc4ODFfZTgyYzA0MjFiMTY2ZDQwMzYyZmMyOWRmMmRmZDFmMzFfSUQ6NzY0NjMxMDU5MTU3MDY3NjkzOV8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

## **Xen 虚拟化**

Xen是运行在裸机上的虚拟化管理程序\(Hypervisor\)。它支持全虚拟化和准虚拟化，Xen 的架构更为复杂，因为它引入了一个独立的 Hypervisor 层以及专门的 Dom0 虚拟机来处理 I/O 和虚拟机管理。

Xen最重要的优势在于准虚拟化，此外未经修改的操作系统也可以直接在Xen上运行\(如Windows\)，能让虚拟机有效运行而不需要仿真，因此虚拟机能感知到Hypervisor，而不需要模拟虚拟硬件，从而能实现高性能。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YzJmYzU1NzdiYWU4NmNjMTE2MWEzZTg2MzRkMjU3ZjVfOGE5NWQ4NGE3MzlmZWM0NzVkMjc0MWYwMDU0NmUzOGNfSUQ6NzY0NjMxMDU5MDk5MDU2ODY1MV8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)



## **KVM 虚拟化**

### 架构介绍

KVM\(Kernel\-basedVirtualMachine\)是基于 Linux 内核的开源的全虚拟化解决方案,支持硬件辅助虚拟化技术\(IntelVT 或 AMD\-V\)。KVM已集成到Linux的内核模块，该内核模块使得 Linux 系统变成了一个 Hypervisor。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZDk4N2IwMGUxYmQyNmVhMGFjM2IxOGRkY2M5MmI5Y2JfYzg4NDgxMmIwNGMyNTU3MDRhMmFhOTUyZjhkMTVjZTlfSUQ6NzY0NjMxMDU5MjE2MDY0ODM4MF8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

- 每个虚拟机都是一个常规的 Linux 进程,运行在 QEMU\-KVM 进程的地址空间,通过 Linux调度程序进行调度

- 虚机的创建和运行是 QEMU 和 KVM 相互配合的过程。两者的通信接口主要是一系列针对特殊设备文件 /dev/kvm 的 IOCTL 调用。其中最重要的是创建虚机，它可以理解成KVM 为了某个特定的虚机创建对应的内核数据结构，同时，KVM 返回一个文件句柄来代表所创建的虚机。

- **KVM：**本身只能够提供 CPU、内存虚拟化等部分功能,而I/O 等其他设备的虚拟化则需要依靠 QEMU 来完成

- **QEMU**：运行在用户空间，提供硬件 I/O 虚拟化，通过 /dev/kvm 接口和kvm实现交互。

- **QEMU\-KVM**: 相比于 QEMU 单独运行虚机时的性能，KVM 让 QEMU 创建的虚拟机几乎可以以接近本地的速度运行。为了简化代码，KVM 在 QEMU 的基础上做了修改，这就是qemu\-kvm。

### 功能特性

KVM 所支持的功能包括：

- 支持 CPU 和 memory 超分（Overcommit）

- 支持半虚拟化 I/O （virtio）

- 支持热插拔 （cpu，块设备、网络设备等）

- 支持对称多处理（Symmetric Multi\-Processing，缩写为 SMP ）

- 支持实时迁移（Live Migration）

- 支持 PCI 设备直接分配和 单根 I/O 虚拟化 （SR\-IOV）

- 支持 内核同页合并 （KSM ）

- 支持 NUMA （Non\-Uniform Memory Access，非一致存储访问结构 ）

## Libvirt

### **为什么需要libvirt**

1. Hypervisor 比如 qemu\-kvm 的命令行虚拟机管理工具参数众多，难于使用。

2. Hypervisor 种类众多，没有统一的编程接口来管理它们，这对云环境来说非常重要。

3. 没有统一的方式来方便地定义虚拟机相关的各种可管理对象。

libvirt 已经成为使用最为广泛的对各种虚拟机进行管理的工具和应用程序接口（API），而且一些常用的虚拟机管理工具（如virsh、virt\-install、virt\-manager等）和云计算框架平台（如OpenStack、OpenNebula、Eucalyptus等）都在底层使用libvirt的应用程序接口。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZWZiNWIwNzA1NGZlYzY0MDcwZWJlNjA4MTI4ODkyZDBfMGY5YzcwMmUxZmViNDFmNDI0YWUyMTc2YzgyNjI4OTFfSUQ6NzY0NjMxMDU5MTYxMTI2MDA4OV8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

**virsh： **基于 libvirt 的 命令行工具 （CLI），可以通过daemon进程（libvirtd）调用qemu\-kvm操作管理虚拟机

```Bash
# virsh常用命令
查看虚拟机的XML配置 virsh dumpxml mydomain
简单显示虚拟机信息：virsh dominfo mydomain
查看所有的虚拟机：virsh list --all

创建虚拟机：virsh create /path/to/domain.xml
强制关闭虚拟机：virsh destroy mydomain
正常关闭虚拟机：virsh shutdown mydomain
删除虚机：virsh undefine mydomain
设置虚机自启动: virsh autostart mydomain

编辑虚机配置文件：virsh edit mydomain
设置虚机CPU：virsh setvcpus mydomain4
设置虚机内存:virsh setmem mydomain 4G --persistent
```

- **virt\-Manager**：基于 libvirt 的 GUI 工具

- **virt\-install **：创建KVM虚机的命令行工具

### libvirt **管理KVM虚机的实例**

这里只描述基本的过程。具体的过程，下一篇文章会具体分析 Nova 中 libvirt 的使用。

1. 定义虚机的基本配置，包括 vCPU、内存、磁盘或者cdrom以及启动顺序，生成 xml 配置，调用 virDomainCreateXML API 启动一个虚机

2. 使用 Domain 相关的 API 来管理虚机的生命周期。

3. 添加磁盘：定义一个 disk 的 xml 配置，使用 virDomainAttachDevice API 将它挂载到虚机上。如果不是本地的源磁盘，需要提前准备好。

4. 添加interface：使用 Network API 定义一个虚拟网络（需要提前准备好物理网络），然后定义一个 interface 的 XML 配置，使用 virDomainAttachDevice API 将它加到虚机。

5. 按照需要，重复2、3、4步骤。

### **libvirt XML 配置定义**

Libvirt 使用 XML 来定义各种对象，其中，与 OpenStack Nova 关系比较密切的有：

**disk\(磁盘\)**

任何磁盘设备，包括软盘（floppy）、硬盘（hard disk）、光驱（cdrom）或者半虚拟化驱动都使用 \<disk\> 元素来定义。

- ”type“ 用来指定device source 的类型："file", "block", "dir", "network", 或者 "volume"。具体的 source  由 \<source\> 标签定义。

- ”device“ 用来指定 device target 的类型："floppy", "disk", "cdrom", and "lun", 默认为 "disk" 。具体的 target 由 \<target\> 标签定义。

```XML
<disk type='network' device='disk'>
      <driver name='qemu' type='raw' cache='none'/>
      <source protocol='iscsi' name='iqn.2001-01.com.suma:storage.target.10.0.39.206/1'>
        <host name='10.0.39.206' port='3260'/>
        <initiator>
          <iqn name='iqn.2023-01.com.xstor:44aa7bcb-9082-4164-b2b1-06f73adf343e'/>
        </initiator>
      </source>
      <target dev='vda' bus='virtio'/>
      <serial>44aa7bcb-9082-4164-b2b1-06f73adf343e</serial>
      <boot order='1'/>
      <alias name='virtio-disk0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </disk>
    <disk type='block' device='disk'>
      <driver name='qemu' type='raw' cache='none' discard='unmap'/>
      <source dev='/dev/sdl'/>
      <backingStore/>
      <target dev='vdb' bus='virtio'/>
      <serial>60f91434-96ec-4cc7-b472-012a446bffe4</serial>
      <alias name='virtio-disk1'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </disk>
    <disk type='file' device='cdrom'>
      <driver name='qemu'/>
      <target dev='hdc' bus='ide'/>
      <readonly/>
      <alias name='ide0-1-0'/>
      <address type='drive' controller='0' bus='1' target='0' unit='0'/>
    </disk>
```

**hostdev（主机设备分配）**

```XML
<hostdev mode='subsystem' type='usb'> # USB 设备直接分配
      <source startupPolicy='optional'>
        <vendor id='0x1234'/>
        <product id='0xbeef'/>
      </source>
      <boot order='2'/>
    </hostdev>
    <hostdev managed="yes" mode="subsystem" type="pci">    #主机网卡设备直接分配
      <source>
        <address bus="0x61" domain="0x0000" function="0x1" slot="0x00" />
      </source>
    </hostdev>
```

**network interface \(网卡\)**

有几种 interface 类型：

（1）type = ‘network’ 定义一个连接 Virtual network 的 interface

（2）type=‘birdge’ 定义一个 Bridge to LAN（桥接到物理网络）的interface：前提是主机上存在一个 bridge，该 bridge 已经连到物理LAN。

（3）type=‘hostdev’ 定义一个由主机PCI 网卡直接分配（PCI Passthrough）的 interface： 分配主机上的网卡给虚机

```XML
<interface type='bridge'>
      <mac address='fa:16:3e:63:13:03'/>
      <source bridge='br-int'/>
      <virtualport type='openvswitch'>
        <parameters interfaceid='9e8397ee-e7c2-40df-97f6-ebb893b94190'/>
      </virtualport>
      <target dev='tap9e8397ee-e7'/>
      <model type='virtio'/>
      <mtu size='1500'/>
      <alias name='net0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </interface>
```

# 网络虚拟化技术

## 虚拟交换机 OpenvSwitch

> 在基于Linux内核的系统上，应用最广泛的还是系统自带的虚拟交换机`Linux Bridge`，它是一个单纯的基于MAC地址学习的二层交换机，简单高效，但同时缺乏一些高级特性，比如OpenFlow,VLAN tag,QOS,ACL,Flow等，而且在隧道协议支持上，Linux Bridge只支持vxlan，OVS支持gre/vxlan/IPsec等，这也决定了OVS更适用于实现SDN技术。
> 
> 

虚拟交换机（vswitch）主要有两个作用：传递虚拟机VM之间的流量，以及实现VM和外界网络的通信。

### OVS 架构

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZWE5MjI0NTAxZjQyZDY3ZTgwOWVhYTQ3MWYyNWZiNjhfZWIzODI0MmYzZDFkYTVkNjcwOWQyOTQ0Y2I3NWU1NjlfSUQ6NzY0NjMxMDU5MjczOTExODI5MV8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

在 OVS 中，datapath 负责执行数据交换，也就是把从接收端口收到的数据包在流表中进行匹配，并执行匹配到的动作。

为了说明datapath，来看一张更详细的架构图，图中的大部分组件上面都有提到

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MWEyNzhkN2EwNDI2Yzc5MTUwZjc0NjQ5MmQzNTU4NzhfMGIxZmUzOTljMzM2ZDFhZjMwOTc4NTRhMGY1ZjhhNzNfSUQ6NzY0NjMxMDU5MDgzOTY1NTYwNF8xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

首先，`datapath`内核模块收到进入数据包\(物理网卡或虚拟网卡\)，然后查找其缓存\(datapath flows\)，当有一个匹配的flow时它执行对应的操作，否则`datapath`会把该数据包送入用户空间由`ovs-vswitchd`负责在其OpenFlow flows中查询\(图1中的First Packet\)，`ovs-vswitchd`查询后把匹配的actions返回给`datapath`并设置一条datapath flows到`datapath`中，这样后续进入的同类型的数据包\(图1中的Subsequent Packets\)因为缓存匹配会被`datapath`直接处理，避免后续同样的流继续upcall到用户空间进行流表匹配。

**用户空间****`ovs-vswitchd`****和内核模块****`datapath`****决定了数据包的转发。**

### 命令行工具

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YjBkYjMwNmJkM2Y1ZTVkZmYwNzc3MDAyN2RmMjdiMzZfZmI5MTU2NTNhYzFlNTkzNjg1OTM2YWEyOTVkN2RkNzFfSUQ6NzY0NjMxMDU5MTU3MDY5MzMyM18xNzgxMzY5MDk5OjE3ODE0NTU0OTlfVjM)

**ovs\-ofctl**

`ovs-ofctl`是专门管理配置OpenFlow交换机的命令行工具，我们可以用它手动配置OVS中的OpenFlow flows，注意其不能操作datapath flows和”hidden” flows，datapath flow和”hidden” flows由OVS自身管理。

```YAML
#查看br-int网桥中的OpenFlow flows
ovs-ofctl dump-flows br-int

#查看br-int端口信息   
ovs-ofctl show br-int

#添加新的flow：对于从端口p0进入交换机的数据包，如果它不包含任何VLAN tag，则自动为它添加VLAN tag 101
ovs-ofctl add-flow br0 "priority=3,in_port=100,dl_vlan=0xffff,actions=mod_vlan_vid:101,normal"

#对于从端口3进入的数据包，若其vlan tag为100，去掉其vlan tag，并从端口1发出 
ovs-ofctl add-flow br0 in_port=3,dl_vlan=101,actions=strip_vlan,output:1

#添加新的flow: 修改从端口p1收到的数据包的源地址为9.181.137.1,show 查看p1端口ID为100   
ovs-ofctl add-flow br0 "priority=1 idle_timeout=0,in_port=100,actions=mod_nw_src:9.181.137.1,normal"

#添加新的flow: 重定向所有的ICMP数据包到端口 p2
ovs-ofctl add-flow br0 idle_timeout=0,dl_type=0x0800,nw_proto=1,actions=output:102

#删除编号为 100 的端口上的所有流表项   
ovs-ofctl del-flows br0 "in_port=100"
```

**ovs\-vsctl**

`ovs-vsctl`是一个管理或配置`ovs-vswitchd`的高级命令行工具，高级是说其操作对用户友好，封装了对数据库的操作细节。它是管理OVS最常用的命令，除了配置flows之外，其它大部分操作比如Bridge/Port/Interface/Controller/Database/Vlan等都可以完成

```Python
#添加网桥
ovs-vsctl add-br br-int

#列出网桥
ovs-vsctl list-br

#查看全部信息
ovs-vsctl show

#给网桥添加端口
ovs-vsctl add-port br-int tap-xxx

#删除一个Port到网桥
ovs-vsctl del-port br-int tap-xxxx
#查看网桥上所有Port   
ovs-vsctl list-ports br-int

#设置端口p1的vlan tag为100
ovs-vsctl set Port p1 tag=100
#设置Port p0类型为internal
ovs-vsctl set Interface p0 type=internal


#获取br0网桥的OpenFlow控制器地址，没有控制器则返回空 
ovs-vsctl get-controller br0
#设置OpenFlow控制器,控制器地址为192.168.1.10，端口为6633
ovs-vsctl set-controller br0 tcp:192.168.1.10:6633
#移除controller
ovs-vsctl del-controller br0
```

## 曙光云网络

```Bash
# 显示当前虚拟交换机的网桥、端口、接口信息
[root@master01 ~]# ovs-vsctl show
4c9df523-d1e1-4025-9ae8-eda92ec0aeda
    Manager "ptcp:6640:0.0.0.0"
        is_connected: true
    Bridge br-ex
        datapath_type: system
        Port br-ex
            Interface br-ex
                type: internal
        Port patch-snat-br-int
            Interface patch-snat-br-int
                type: patch
                options: {peer=patch-snat-br-ex}
        Port pod-mgmt
            tag: 1000
            Interface pod-mgmt
                type: internal
        Port business
            Interface business
        Port br-int-patch
            Interface br-int-patch
                type: patch
                options: {peer=br-ex-patch}
    Bridge br-int
        Controller "tcp:100.125.254.1:6633"
            is_connected: true
        fail_mode: secure
        datapath_type: system
        Port geneve-vtp
            Interface geneve-vtp
                type: geneve
                options: {key=flow, local_ip="100.125.254.1", remote_ip=flow}
        Port tap-dns
            Interface tap-dns
                type: internal
        Port tap-metadata
            Interface tap-metadata
                type: internal
        Port patch-snat-br-ex
            Interface patch-snat-br-ex
                type: patch
                options: {peer=patch-snat-br-int}
        Port br-ex-patch
            Interface br-ex-patch
                type: patch
                options: {peer=br-int-patch}
        Port br-int
            Interface br-int
                type: internal
```

### 网桥 `br-ex`

- 功能​：`br-ex` 通常用于连接外部网络（如物理网络或外部网关）。

- 关键组件​：

    1. 内部端口 

        1. `br-ex`​：创建会有个对应的虚拟网卡 `br-ex`，用于处理外部网络流量。

        2. `pod-mgmt`​：标记为 VLAN 1000，用于管理 Pod 的流量隔离。

    2. Patch 端口 

        1. `patch-snat-br-int`​：与 `br-int` 的 `patch-snat-br-ex` 连接，用于跨网桥通信（如 SNAT 流量转发）。

        2. `br-int-patch`​：与 `br-int` 的 `br-ex-patch` 连接，用于跨网桥互通。

    3. `Port business`​：连接到物理网卡（业务网络）。

### 网桥 `br-int`

- 功能​：`br-int` 是集成网桥，通常用于内部虚拟网络（如虚拟机或容器的流量）。

- 关键组件​：

    1. SDN 控制器​：连接到 `tcp:100.125.254.1:6633`，负责下发流表规则。

    2. Geneve 隧道 `geneve-vtp`​：用于构建覆盖网络（如跨主机的虚拟网络），`local_ip` 指定本机 IP，`remote_ip=flow` 表示由流表决定目标。

    3. 内部端口​：

        - `tap-dns`：可能用于 DNS 服务。

        - `tap-metadata`：用于元数据服务，IP地址为 169\.254\.169\.254（如云平台的实例元数据）。

        - `br-int`​：创建虚拟网卡 `br-int`，用于内部流量处理。

    4. Patch 端口​：

        - `patch-snat-br-ex`：与 `br-ex` 的 `patch-snat-br-int` 配对，用于跨网桥 SNAT 流量。

        - `br-ex-patch`：与 `br-ex` 的 `br-int-patch` 配对，用于通用跨网桥通信。

### 关键功能总结

1. 跨网桥通信​：通过 `patch` 端口连接 `br-ex` 和 `br-int`，实现外部网络与内部虚拟网络的互通。

2. 覆盖网络​（overlay ）：Geneve 隧道支持跨主机的虚拟网络通信（如 Kubernetes 或 OpenStack 多节点场景）。

3. VLAN 隔离​：`pod-mgmt` 使用 VLAN 1000，隔离管理流量。

4. SDN 控制​：通过控制器 `100.125.254.1:6633` 动态管理流表，实现灵活的网络策略。

### Underlay 与 Overlay 的关系

- Underlay 网络​：物理网络基础设施（`100.125.0.0/16`），负责传输：

    - 节点间 Geneve 隧道流量（如 `100.125.254.1` ↔ `100.125.254.3`）。

    - SDN 控制器通信（`100.125.254.1:6633`）。

- Overlay 网络​：基于 Geneve 隧道构建的虚拟网络，运行在 Underlay 之上，用于：

    - 虚拟机/容器跨节点通信。

    - 租户隔离、虚拟子网等。

