# 基于LLM的脚本自愈修复

基于大语言模型（LLM）的智能定位修复是当前测试工程化领域最前沿的方向之一。它正在将测试人员从繁琐的“查日志、断点调试、改脚本”中解放出来。

我们可以将其分为**入门、进阶、高阶**三个阶段，由浅入深地推进。

---

## 第一阶段：增强型异常诊断（入门级）

在这个阶段，我们不改变现有的测试流程，而是利用 LLM 充当“专家大脑”，辅助分析已有的报错。

**落地场景**

- **日志翻译与解释：** 将晦涩难懂的堆栈信息（Stack Trace）交给 LLM，让它用人话解释报错原因。

- **代码辅助审查：** 发现报错后，将报错信息和相关代码片段发给 LLM，询问“哪里可能写错了”。

**实施方案**

- **工具：** 直接使用 ChatGPT、Claude 或公司内部的大模型接口。

- **做法：** 编写一套 **Standard Prompt（标准提示词）**。

> - **示例 Prompt：** “你是一个资深 Java 开发工程师。以下是自动化测试抛出的异常堆栈和相关的业务代码。请分析报错原因，并指出代码中第几行可能存在逻辑缺陷。”
> 
> 

**核心价值**

- **提效：** 减少新人由于看不懂日志而浪费的时间。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NmYwZTI1YTc4MzEzODZiZjIzYzRjZDE5NjEwODY2YjJfMmYwNzQzYzIxMTFhODNkZWRhYWM1NzliYzgwMDQwNmJfSUQ6NzY0ODM2NTM2ODMzOTEwNzA0NF8xNzgxMzc1MDg5OjE3ODE0NjE0ODlfVjM)

---

## 第二阶段：RAG 驱动的知识库定位（进阶级）

当错误反复出现时，纯靠 LLM 泛化能力不够，需要结合公司的**历史业务知识**。

**落地场景**

- **历史缺陷关联：** 当新 Bug 出现时，自动检索过去两年内类似的 Bug 单，告诉测试人员“这个问题可能和去年的 XXX 模块漏测有关”。

- **自动化脚本自愈（Self\-healing）：** 当 UI 自动化脚本因为页面元素变更而失败时，LLM 自动分析 HTML 结构，寻找新的定位符。

**实施方案**

- **技术架构：RAG \(Retrieval\-Augmented Generation\)**。

    - **Step 1：** 将历史 Bug 单、技术文档、代码库向量化存储到向量数据库（如 Milvus 或 Pinecone）。

    - **Step 2：** 当测试失败时，提取失败特征，在数据库中检索相似片段。

    - **Step 3：** 将检索到的背景知识 \+ 当前错误信息一起塞给 LLM，生成定位报告。

**核心价值**

- **精准：** 结合了业务上下文，定位准确率显著提升，不再是“空谈理论”。

---

## 第三阶段：Multi\-Agent 驱动的自动修复（高阶级）

这是目前的行业天花板：不仅仅是定位，而是直接给出修复建议（Patch），甚至自动提交代码。

**落地场景**

- **自动程序修复（APR）：** 发现 Bug \-\> 定位代码 \-\> 生成修复补丁 \-\> 运行单元测试验证 \-\> 提交 Merge Request。

- **智能冒烟测试：** 开发提交代码后，LLM 自动分析代码改动（Diff），识别受影响的范围，并自动生成、执行测试用例进行修复验证。

**实施方案**

- **技术架构：AI Agents \(多智能体协作\)**。

    - **分析 Agent：** 负责读取日志和代码，定位 Root Cause。

    - **修复 Agent：** 负责编写修改代码。

    - **验证 Agent：** 负责编写并执行 TestCase，确保修复没有引入新 Bug。

- **循环机制：** 如果验证失败，反馈给修复 Agent 重新修改，直到通过测试。

## 评价指标

怎么衡量智能定位修复做的好不好？

- **MTTD（平均检测时间）** 是否下降。

- **MTTR（平均修复时间）** 是否下降。

- **定位准确率：** AI 给出的前 3 个建议中包含真实原因的比例。

---

## Jenkins pipeline 结果智能定位的落地实战

### 实践分享

必备使用工具：Codex App \+ GPT 5\.4

失败用例输入来源：http://172\.22\.5\.60:8080/job/SugonCloud\_UI\_Regression\_Test/275/allure/\#

线程分享链接：codex://threads/019d470f\-e8c3\-77c3\-87c5\-9c2e8ea4b700

相关截图：

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YjE3ODBiM2VhODcyMTdkNGYzOTcxYzhlMTdkNDZhM2RfYzdjZGMwYWI5YzkwM2NiM2M5MzJjZjFkNWEwMzFmYTJfSUQ6NzY0ODM2NTM0Nzg2MjUxNDkwNl8xNzgxMzc1MDg5OjE3ODE0NjE0ODlfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZjA2YjEzZDNlYzI4NTljYzg1NDA2NGI0NTE5ZTgxZjBfZjY2MGMyODQwZDcwMGZkOTk4MjczY2QzZjRlMmY4MmRfSUQ6NzY0ODM2NTM1NjQxODkxMTQ2OV8xNzgxMzc1MDg5OjE3ODE0NjE0ODlfVjM)

### 发现的不足

* [ ] 某些测试场景下，提供的失败截图存在延迟，这会对LLM的问题定位产生误导。示例如下：

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=M2U4NGMwZTUyYjEwNDY2Y2I3YTljMjljMWRmMTlhMDJfN2RjMDJjNWY0NGJiOWE5MzQ0M2I5Y2EzZWVjMzg1MDNfSUQ6NzY0ODM2NTM2MDkzMTk4MjU2NF8xNzgxMzc1MDg5OjE3ODE0NjE0ODlfVjM)

* [ ] 某些失败用例没有产生截图\-》失败时截图的逻辑还有点问题，待进一步排查原因

* [ ] 如果失败点是后台执行gova、cinder等命令出错，提供的失败截图对于LLM并没有帮助

### 未来能力规划

* [ ] Jenkins piepline接入LLM，基于 Allure 测试报告，一键生成根因定位报告，实现测试定位提效

* [ ] Jenkins piepline结果的常见错误日志归类与优化

