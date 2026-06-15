# 云监控CMS

# 智能云监控架构

全栈云监控主要分为：平台资源监控、平台组件监控、平台云产品监控。

- **平台资源监控**：如物理主机内存、cpu、磁盘总量使用量的统计。

- **平台组件监控**：如Nova、Cinder、RABBITMQ服务是否健康。

- **平台云产品监控**：如MYSQL、REDIS、EMR产品的系统资源监控和性能监控。

## 数据采集

### ding\-agent

用于采集云服务器的Metrics（cpu, memory, disk, network, uptime）

### Metrics Server

Metrics Server从kubelet公开的Summary API中采集度量数据，能够收集包括了Pod、Node、容器、Service等主要Kubernetes核心资源的Metrics（度量数据），且对外提供一套标准的API。

### telegraf

telegraf是InfluxData公司开源的一款采集器，内置非常多的采集插件，可以把采集的数据推给OpenTSDB

### categraf

categraf是夜莺主推的一款 all\-in\-one 采集器，不但支持指标采集，未来也计划支持日志和调用链路的数据采集

## 数据处理

### Nightingale

夜莺监控可以接收各种采集器上报的监控数据，转存到时序库（可以支持Prometheus、M3DB、VictoriaMetrics、Thanos等），并提供告警规则、屏蔽规则、订阅规则的配置能力，提供监控数据的查看能力，提供告警自愈机制（告警触发之后自动回调某个webhook地址或者执行某个脚本），提供历史告警事件的存储管理、分组查看的能力。

- **n9e\-api**：承接前端请求，将用户配置写入数据库

- **n9e\-server**：告警引擎和数据转发模块

- **n9e\-redis**：每套server依赖一个redis

## 数据存储

### VictoriaMetrics

vmstorage、vminsert、vmselect三者组合构成VictoriaMetrics的集群功能，三者都可以通过启动多个实例来分担承载流量，通常要在vminsert和vmselect前面架设负载均衡。

- **vminsert**：接收来自客户端的数据写入请求，并转发到选定的vmstorage

- **vmselect**：接收来自客户端的数据查询请求，并负责转发到所有的vmstorage查询结果

- **n9e\-vmstorage**：负责数据存储，删除storage node会丢失约1/N的历史数据

