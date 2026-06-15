# Dify学习笔记

## 新建知识库

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZGU0MzU1OTJjODFmOGE4ZDU3ZWNlODcxZWUxMWNhYjJfYzVhOTFmNDlmYTNlNGYzYzE4MmQ0MjhiYzM4ZTM5ZTZfSUQ6NzYwNzEzNjQzOTAyOTAwOTM3Nl8xNzgxMzc0NzM5OjE3ODE0NjExMzlfVjM)

### 文本分块

**分段标识符：**分隔符是用于分隔文本的字符。\\n\\n 和 \\n 是常用于分隔段落和行的分隔符。用逗号连接分隔符（\\n\\n,\\n），当段落超过最大块长度时，会按行进行分割。

**分段重叠长度：**设置分段之间的重叠长度可以保留分段之间的语义关系，提升召回效果。建议设置为最大分段长度的 10%\-25%

**文本预处理规则**



**父子分段：**子块用于检索，父块用作上下文。父块可以理解为一个段落，子块就是段落里的一个句子。

### 索引方式

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MzU0NDMyNzk5MGI2NmUxYmU5ZWIzOTkzNGZlMjVhOWZfMmM5YzQ0OTIwOTY1NzkzOTIwNzg3ZTJmMjkzMTY2ZTFfSUQ6NzYwNzE0ODg0NjAyMzQ0NTcyOF8xNzgxMzc0NzM5OjE3ODE0NjExMzlfVjM)

### 检索设置

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MmU0ODIzYTkyODA1NDRiNDg3YmY2OWVlMTc4MGY1MDBfMjNlMzk0MzIxYjdlOTBhMmRkNTUzOTlmYmQyYzVjNmJfSUQ6NzYwNzE0OTc4NTM5NjU2MzE3Ml8xNzgxMzc0NzM5OjE3ODE0NjExMzlfVjM)

**向量检索**：通过生成查询嵌入并查询与其向量表示最相似的文本分段

**全文检索**：索引文档中的所有词汇，从而允许用户查询任意词汇，并返回包含这些词汇的文本片段

**混合检索：**同时执行全文检索和向量检索，并应用重排序步骤，从两类查询结果中选择匹配用户问题的最佳结果，用户可以选择设置权重或配置重新排序模型。



**Rerank模型**：重排序模型将根据候选文档列表与用户问题语义匹配度进行重新排序，从而改进语义排序的结果

**Top K：**用于筛选与用户问题相似度最高的文本片段。系统同时会根据选用模型上下文窗口大小动态调整分段数量

**Score 阈值**：用于设置文本片段筛选的相似度阈值。



## 常见问题

1、知识检索输出的分块内容质量低，没有获取到想要的答案

