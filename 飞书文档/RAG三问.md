# RAG三问

# 什么是RAG

## 定义

凡是和如何有效进行信息检索的相关技术，都可以称为RAG。

检索增强生成（RAG）是指对大语言模型输出进行优化，使其能够在生成响应之前引用训练数据来源之外的权威知识库

## 核心组件

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZWM4NjJhMzAwMGQ1MzMwYTM1N2I2MTcyZDNmYmZiOGRfYmMwNDNjN2IzNzQ0YTQ5NzNkMzVlOWQzMjcwZWZkNmFfSUQ6NzUxMTM1NjAzODEwNTUzMDM2OV8xNzgxMzc0NzE1OjE3ODE0NjExMTVfVjM)

1. 知识嵌入（Embedding）：负责将文本知识转化为向量表示，捕捉语义信息。

2. 向量数据库（Vector DB）：存储由知识嵌入模块生成的向量表示。

3. 检索器（Retriever）：接收用户查询并将其转化为向量，然后从向量数据库中检索相似的文档。

4. 生成器（Generator）：基于检索到的相关上下文信息生成流畅且可信的回答。

与人类的大脑做类比

- 知识嵌入 \-》大脑的长期记忆形成

- 向量数据库 \-〉大脑存储知识

- 检索器 \-》触发大脑的回忆机制

- 生成器 \-〉大脑的推理能力

## 微调和RAG的区别

微调（Fine\-tuning）和检索增强生成（RAG）确实都致力于提升LLM的输出准确性，但实现路径和适用场景存在本质差异：

总结：

- 两者区别类比，开卷考试和闭卷考试

- 若幻觉源于知识缺失 → RAG更有效

- 若幻觉源于表达逻辑错误 → 微调更有效

# 如何快速上手 RAG

## **使用框架：LlamaIndex的5步示例**

## **如何理解嵌入模型？**

1. **语义表示**：

    - 嵌入模型将文本映射到一个连续的向量空间中，语义相似的文本在向量空间中的距离会更近。例如，"猫"和"狗"的向量会比"猫"和"汽车"的向量更接近。

    - 这种表示方式让计算机能够通过数学运算（如余弦相似度）来比较文本的相似性。

2. **降维与稠密表示**：

    - 传统的文本表示方法（如词袋模型或TF\-IDF）通常是稀疏且高维的，而嵌入模型生成的向量是稠密的（维度较低但信息密集）。

    - 这种稠密表示更高效，适合机器学习任务。

3. **预训练与微调**：

    - 嵌入模型通常是预训练的（如 `BAAI/bge-small-zh`），在大规模语料库上学习通用的语义表示。

    - 也可以针对特定任务进行微调，以优化性能。

## LangChain vs LlamaIndex

他俩有重叠之处。也都和大语言模型相关。LangChain应用面广，LlamaIndex更专。

Langchain 是一个更通用的框架，可用于构建各种应用程序。它提供了用于加载、处理和索引数据以及与LLM交互的工具。Langchain 也比 LlamaIndex 更灵活，允许用户自定义其应用程序的行为。里面有链和代理的思想以及实践。

LlamaIndex 专为构建搜索和检索这个功能而设计。LlamaIndex 也比 Langchain 更高效，使其成为需要处理大量数据的应用程序的更好选择。其实就是专注于文档检索——Indexing。这个LangChain里面有一个模块，基本上也覆盖了。

如果你的目标是构建需要灵活和可扩展的通用LLM应用程序，那么 Langchain 。如果你的目标是构建需要高效且简单的搜索和检索应用程序，那么 LlamaIndex 

## LangGraph vs 传统 LangChain

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NTY5Y2MyNjZmNTQ2NDRiOTViYzAyZThkMWZjOWVmZWVfYmMzOTAwNjYwMWU3NWRkY2ZkNzgyNmY4OWVmMGJhZTRfSUQ6NzUxMTY3Nzg3NjI4NzU2OTk0OF8xNzgxMzc0NzE1OjE3ODE0NjExMTVfVjM)

# 如何优化RAG系统

[2025年企业知识库推荐：产品经理必看！RAG、安全合规与国产化全栈适配的选型指南 – 人人都是产品经理](https://www.woshipm.com/ai/6308360.html)

# 技术文章摘要

## 智能体系统的几种常见范式

### 基础结构：LLM增强调用

智能体系统的基本构建模块都是类似这样的大模型调用，包括检索、工具和记忆等能力。当前的主流大模型能够主动利用这些能力——它们可以自主生成搜索查询、选择合适的工具，并决定需要保留哪些信息。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZDgwYzNiN2ZlODEyOGVlOTZlNzcwYmY2Nzc4NDBhYWRfZWY3OGMzZThhZGQxOThiYjUzOTQzZTdmYjJhMzU4ZmJfSUQ6NzYwMjEzNTg5NDk5NjgxNDc5NV8xNzgxMzc0NzE1OjE3ODE0NjExMTVfVjM)

### 工作流：LLM连续调用



**何时使用此工作流：** 此工作流非常适合任务可以轻松、清晰地分解为固定子任务的情况。主要目标是通过使每个LLM调用成为更容易的任务来用延迟换取更高的准确性。

**使用场景：**

- 生成营销文案，然后将其翻译成不同的语言。

- 编写文档大纲，检查大纲是否符合某些标准，然后基于大纲编写文档。

### 工作流：大模型路由



**何时使用此工作流：** 当复杂任务包含适合分别处理的不同类别，且这些类别能够通过LLM或传统的分类模型/算法准确区分时，大模型路由工作流尤为适用。

**使用场景**

- 将不同类型的客户服务查询（一般问题、退款请求、技术支持）引导到不同的下游流程、提示和工具中。

- 将简单/常见问题路由到较小的模型，将困难/不寻常的问题路由到更强大的模型，以优化成本和速度。

### 工作流：并行化



**何时使用此工作流：** 当子任务可以拆分并行处理以提升速度，或需要从多个角度、多次尝试以获得更高置信度结果时，并行化是一种有效方式。对于包含多个考量因素的复杂任务，通常将每个因素交由单独的LLM调用分别处理，能够让模型更专注于各自的特定方面，从而取得更好的效果。

**使用场景：**

- **分段**：

    - 可以通过将用户查询和不当内容筛查分别交由不同的模型实例处理来实现保护措施。通常，这种方式比让同一个LLM同时负责保护和核心响应更有效。

    - 自动化评估LLM性能时，可让每个LLM调用分别评估模型在特定提示下的不同表现方面。

- **投票**：

    - 审查代码漏洞时，可采用多个不同的提示分别对代码进行审查，并在发现问题时进行标记。

    - 评估内容是否不当时，可以通过多个提示从不同角度进行评估，并根据需要设定不同的投票阈值，以平衡假阳性和假阴性的比例。

### 工作流：反思 \- 优化



**何时使用此工作流：** 当我们拥有明确的评估标准，并且通过迭代改进能够带来可衡量的价值时，这种工作流尤其有效。

有两个明显的判断适用的标志：

1. 当人类能够阐述反馈时，LLM的响应会有显著提升；

2. 当LLM本身能够给出类似的人类反馈。这一过程类似于人类作家在创作精美文档时所经历的反复打磨。

**使用场景：**

- 文学翻译：翻译者LLM初稿可能难以捕捉细微差别，但评估者LLM能够给出有价值的批评意见。

- 复杂搜索任务：需要多轮搜索和分析以收集全面信息，由评估者判断是否需要进一步搜索。

### Agentic

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MmU1ZjRjN2E4MjdjMDUzOGNiMmFiMDVjOWRjMjcyN2ZfMTcxZGVkYTkzNWE5ZmM0OTE4NDhhMjg3MDEzZTQwMTVfSUQ6NzYwMjE0MjAwODIwMjkyMzIzNl8xNzgxMzc0NzE1OjE3ODE0NjExMTVfVjM)

**何时使用Agent：** 当任务属于开放性问题，难以或无法预先确定所需步骤数，或者无法通过硬编码实现固定流程时，Agent就非常适用。在这种情况下，LLM可能需要多轮运行，因此你必须对其决策过程有一定的信任。智能体的自主性使其非常适合在受信任的环境中扩展任务。

但正因为Agent具备自主性，也意味着更高的成本和潜在的错误积累风险。我们建议在沙盒环境中进行充分测试，并配备必要的保护措施。

**使用场景：**

- 编程助手Agent。Agent需要根据人类的指令，操作IDE/文件系统/命令行等工具，完成代码的编写/修改/调试等任务b画

- DeepResearch Agent。Agent需要根据人类的指令，操作浏览器/文件系统/命令行等工具，完成文献调研/数据分析/报告撰写等任务；

### 总结与思考



在LLM领域，成功的关键不是构建最复杂的系统，而是为您的需求打造**最合适**的系统。应从简单的结构入手，经过全面评估后进行优化，只有在简单方案无法满足需求时，才引入多步骤的智能体系统。

在实现智能体时，我们始终遵循三条核心原则：

1. 在智能体设计中保持**简单性**。

2. 优先考虑**透明度**，通过清晰展示智能体的规划步骤。

3. 通过完善的工具**文档和测试**，精心打造您的智能体\-计算机接口（ACI）。

框架可以帮助您快速起步，但在进入生产阶段时，**请果断减少抽象层**，转而使用基础组件进行构建。遵循这些原则，您将能够打造出既强大又可靠、易于维护且值得用户信赖的智能体。



