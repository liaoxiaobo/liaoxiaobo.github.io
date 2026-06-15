# Jenkins虚机内存占用分析

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NzkzY2U3NDJlMWI5NzFmZmVhNGQ1OGEwYzA4MGNiMDNfYTM0Y2I3OTRkYTA0Nzc3NmQ0YmJlYWE0NGM2MTU5ZWFfSUQ6NzY0ODk3ODA0Mzc5MjkxOTUxNF8xNzgxMzc1MTE2OjE3ODE0NjE1MTZfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZmE4NDliOWMwMGEyZjU4MjJiMWRmMGVlNGYxY2M2MjZfMzZmYjc3NzY1ODZjMjI5Yjg5Njc2ZDZlNTQ3MjA4OTJfSUQ6NzY0ODk3ODc2ODM3MTMwNTQ0NV8xNzgxMzc1MTE2OjE3ODE0NjE1MTZfVjM)

# Jenkins 虚拟机资源占用与 UI 自动化性能瓶颈深度分析报告

本报告结合了对 Sugon Cloud 执行机（`sugon-bef86fed-...`）在自动化测试执行高峰期的系统资源（`free -h`）及容器状态（`docker stats/ps`）的连续采样观测，对系统内存、CPU 瓶颈以及 Playwright 自动化测试的资源消耗特性进行了全面汇总。

## 📊 一、 虚拟机资源占用现状分析（基于截图数据）

在测试任务并发执行的高峰期，宿主机与容器的资源吞吐呈现出明显的“一稳、三动态、内存足、CPU 满”的特征：

### 虚拟机系统层资源 \(`free -h`\)

- **物理内存总量 \(Total\):** $15\text{ GiB}$（实际物理硬件约为 $16\text{ GiB}$）。

- **实际硬占用 \(Used\):** 随着测试并发量从初期闲置的 $3.8\text{ GiB}$ 逐步爬升至 $5.0\text{ GiB}$。该部分为不可回收的硬内存，由 Jenkins 常驻进程与运行中的 Playwright 容器共同消耗。

- **文件缓存 \(Buff/Cache\):** 维持在 $8.0\text{ GiB} \sim 9.2\text{ GiB}$ 的高位。

    - *成因分析：* 观测到系统发生了高达 **$134\\text\{ GiB\}$ 的惊人磁盘读取（Block I/O）**。这是由于 Jenkins 频繁读写构建历史、日志、Workspace 源码及浏览器依赖包，引发 Linux 内核自动调用大量闲置内存作为 Page Cache 进行加速。

- **可用内存 \(Available\):** 稳定保持在 $9.4\text{ GiB} \sim 10\text{ GiB}$ 之间。**内存层面绝对安全**，无任何 OOM（内存溢出）风险。

### 容器层资源占用细查 \(`docker stats & ps`\)

当前宿主机共运行 4 个容器，Playwright 容器均采用 `"cat"` 命令常驻保持长连接，且**生命周期管理正常，跑完后会自动触发销毁**：

- `jenkins`** 容器 \(长期常驻\)：** 内存稳定在 $3.82\text{ GiB} \sim 3.88\text{ GiB}$（约占总内存 $25\%$），CPU 占用极低（$< 0.4\%$）。*注：其内存大小与 Java 默认根据宿主机总内存分分配 Max Heap Size 的机制相吻合。*

- `eloquent_mendeleev`** 容器 \(新启动 4 分钟\)：** 正在执行核心用例，**CPU 占用率直接飙升至 **$351.72\%$，内存吃掉 $1.739\text{ GiB}$。

- `tender_bouman`** 与 **`vigilant_solomon`** 容器 \(已分别运行 2 小时与 37 分钟\)：** 属于长耗时任务，目前仍在持续运行中，各占约 $38\%$ 的 CPU 和 $400\text{ MiB} \sim 600\text{ MiB}$ 的内存。

## ⚖️ 二、 核心技术结论：为什么 UI 自动化测试“重 CPU、轻内存”？

通过对上述截图数据的分析，直击了 UI 自动化测试最核心的资源痛点：**跑 Playwright 自动化测试时，CPU 几乎总是比内存更容易成为真正的瓶颈。**

### 为什么 CPU 会瞬间爆表？

Playwright 表面上运行的是 Python/Node\.js 脚本，但其底层是在**直接驱动真实的浏览器内核（Chromium / WebKit / Firefox）**，浏览器本身就是典型的“CPU 密集型”软件：

- **自动等待机制（Auto\-wait）的高频轮询：** 当脚本在查找某个元素时，底层会以毫秒级的高频率不断轮询 DOM 树。**一旦页面加载慢或脚本处于超时（Timeout）等待状态，这种高频轮询会瞬间把 CPU 单核烧满。**

- **DOM 树解析与高频重绘：** 脚本在页面上进行的每一次点击、滚动、输入，都会触发浏览器动态计算 CSS、布局重排（Reflow）和重绘（Repaint）。

- **多媒体与多线程编码：** 如果流水线开启了用例录屏（Video）、自动截图或跟踪分析（Trace），浏览器需要实时对屏幕进行图像编码与压缩，这些都是极度消耗 CPU 算力的操作。

### 为什么内存反而不容易成为瓶颈？

- **动态销毁机制：** UI 自动化容器由于具备“跑完即销毁”的特性，内存属于周期性释放的“定量”资源。单个 Playwright 容器在生命周期内通常只需占用 $1\text{ GiB} \sim 2\text{ GiB}$ 内存，3 并发也仅消耗约 $6\text{ GiB}$，对于 $16\text{ GiB}$ 的虚机而言绰绰有余。

- **隐蔽的“排队”机制：** 内存不足时系统会触发 OOM 直接报错，容易引起警觉；而 CPU 不足时系统不会死机，而是让进程排队。**这种 CPU 争抢会导致浏览器响应变慢，从而引发 Playwright 产生大量偶发性、难以复现的元素查找超时（Timeout）误报。**

## 🛠️ 三、 针对性优化与避坑指南

明确了 CPU 是核心瓶颈且当前存在 3 任务并发、部分任务超 2 小时的情况，提出以下优化建议：

### 执行机选型“重核轻存”

在采购或分配自动化执行机时，**优先关注 CPU 核心数，而非内存大小**。

- **经验公式：** **4核 8G** 的服务器运行 UI 自动化的实际表现与稳定性，往往远好于 **2核 16G**。

### 严格控制 Jenkins 流水线的最大并发数

鉴于单个 Playwright 容器就能吃掉 $350\%$ 以上的 CPU，不能无节制地并行启动测试容器。

- **业内经验：** **每 1\.5 \~ 2 个 CPU 核心，允许分配给 1 个并发的 Playwright 容器**。

- **落地建议：** 建议将当前这台虚机标签（Label）下的**最大并发任务数限制为 1 或 2**，通过排队机制（Queue）使测试任务有序执行，避免多任务争抢 CPU 导致用例整体执行时间被无限拉长。

### 始终保持 Headless（无头）模式

在 CI/CD 环境中，确保脚本中声明了 `headless=True`。无头模式下浏览器不会渲染真实的物理窗口，能为宿主机省下大量的 CPU 算力。

### 降级非必要的录屏与截图策略

Trace Viewer 和视频录制是非常恐怖的 CPU 杀手。

- **优化策略：** 将配置调整为 `video="on-first-retry"`（仅在失败重试时才开启录屏和追踪），对于正常通过的流水线，一律不录制视频、不保存高频 Trace，只保留最终的 HTML 报告。

### 排查并拆分“长耗时”测试用例

针对运行已超 2 小时的容器，建议进入对应 Jenkins 任务日志中排查原因。

- 如果是全量回归用例过大，建议将大任务拆分为多个 Job **异步串行**或**分发到不同执行机并行**。

- 检查脚本中是否存在未设置超时时间的死循环等待，防止执行机通道被长期无效占用。

