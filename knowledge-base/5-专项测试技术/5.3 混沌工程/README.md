# 5.3 混沌工程

## 定位

覆盖混沌工程的理论原则、实验方法论，以及 Chaos Mesh、Chaos Blade 两款主流工具在云原生场景下的实战应用。聚焦"如何在分布式系统和 Kubernetes 环境中，通过受控故障注入验证系统韧性"。

## 设计原则

1. **理论先行**：先理解混沌工程的实验模型和原则，再动手注入故障。
2. **工具服务于场景**：Chaos Mesh 适合 Kubernetes 环境，Chaos Blade 适合更广泛的宿主机/容器/应用层故障。
3. **安全第一**：所有实验都要控制爆炸半径，优先在测试环境验证，再谨慎推向生产。
4. **可观测配套**：混沌实验必须配合监控、日志、告警，否则无法判断系统行为是否符合预期。
5. **可复用资产沉淀**：将典型实验场景 YAML、排查 Checklist、止损方案整理为模板。

## 规划内容（二级目录）

```text
5.3 混沌工程
├── 5.3.1 混沌工程理论与原则
├── 5.3.2 Chaos Mesh 实战
└── 5.3.3 Chaos Blade 实战
```

## 源文档映射

| 源文档 | 目标节点 | 处理建议 |
| --- | --- | --- |
| `source/_posts/chaos.md` | 5.3.1 / 5.3.2 / 5.3.3 | 理论与原则进 5.3.1；Chaos Mesh 架构与概述进 5.3.2；Chaos Blade 架构与概述进 5.3.3 |
| `source/_posts/chaos-tool.md` | 5.3.2 | Chaos Mesh 安装排障、PodChaos、StressChaos、IOChaos、Schedule 等实战场景 |

## 落地优先级

- **P0（优先整理）**：5.3.1 混沌工程理论与原则
  - 源文档成熟，是理解后续工具实战的基础。
- **P1（次优先）**：5.3.2 Chaos Mesh 实战
  - 源文档详细，覆盖安装、排障、多种故障注入和 Schedule 场景。
- **P2（可后延）**：5.3.3 Chaos Blade 实战
  - 源文档以概述和基础使用为主，可后续补充更多实战案例。

## 关键边界提醒

- **5.3.1 vs 5.2.1 稳定性与可靠性测试**：5.3.1 聚焦主动注入故障的实验方法论，5.2.1 聚焦稳定性测试的整体策略和指标。
- **5.3.2 Chaos Mesh vs 4.2 Kubernetes 测试**：5.3.2 讲故障注入工具用法，4.2 讲 K8s 集群本身的部署、监控与测试关注点。
- **5.3.3 Chaos Blade vs 5.3.2 Chaos Mesh**：Chaos Mesh 更偏向 K8s 原生和可视化，Chaos Blade 更偏向 CLI 和跨平台（主机、容器、JVM 等）。

## 待整理文档

- [x] `source/_posts/chaos.md` 中理论与原则部分 → 5.3.1
- [x] `source/_posts/chaos.md` 中 Chaos Mesh 概述 → 5.3.2
- [x] `source/_posts/chaos.md` 中 Chaos Blade 概述 → 5.3.3
- [x] `source/_posts/chaos-tool.md` → 5.3.2
