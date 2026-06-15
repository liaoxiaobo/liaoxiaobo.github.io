# AI 智能失败归因 — 方案设计文档

# AI 智能失败归因 — 方案设计文档



> https://mp\.weixin\.qq\.com/s/TCMSYDG4q5U5Vfk5Esir5Q
> 
> 

## 一、现状分析



### 1\.1 项目现状



|维度|现状|
|---|---|
|测试框架|pytest \+ Playwright \+ Allure|
|CI/CD|Jenkins Pipeline \+ Docker|
|通知渠道|飞书 Webhook（仅发送构建结果链接）|
|报告|Allure HTML 报告（需人工查看）|
|AI 能力|已有 ai\_report\.py：CLI 工具，调用 DeepSeek/DashScope 分析失败用例，输出 Markdown|
|Prompt|已有 test\_failure\_analysis\_prompt\.md：三分类（环境问题/用例问题/产品缺陷），结构化输出|



### 1\.2 已有 ai\_report\.py 能力边界



**已有能力**：Allure 结果解析、日志片段提取、附件摘要、LLM 调用、Markdown 输出



**核心瓶颈**：



- **被动触发**：需人工执行 python ai\_report\.py，未嵌入 CI 流程

- **信息维度单一**：仅消费 Allure JSON \+ 日志文本，缺少截图、网络请求、代码变更上下文

- **单轮分析**：一次 LLM 调用处理所有失败用例，Token 限制下分析深度不足

- **无反馈闭环**：分析结果散落为 Markdown 文件，未回写 Allure / 未推送通知 / 无历史沉淀

- **无聚类能力**：相同根因的多个失败用例被逐条独立分析，无法自动聚合

    

### 1\.3 数据源选择说明



三个方案均基于 **allure\-result 中间产物** 进行归因分析，而非最终生成的 Allure HTML 报告。



|对比维度|allure\-result（中间产物）|Allure HTML Report（最终报告）|
|---|---|---|
|数据格式|结构化 JSON \+ 纯文本附件|编译后的 JS Bundle \+ 压缩数据|
|可解析性|直接 json\.load\(\) 读取|需要浏览器渲染或逆向 JS 数据格式|
|完整性|包含原始附件（截图、日志原文）|截图被移动/重命名，日志可能被截断|
|时效性|测试运行中即可实时消费|必须等 allure generate 完成后|
|稳定性|Allure 官方稳定的数据契约|前端版本升级可能改变数据嵌入方式|



**结论**：Allure HTML 报告是给人看的，allure\-result 是给程序看的。归因分析必须基于结构化的中间产物。



### 1\.4 核心提效目标



|目标|量化指标|
|---|---|
|减少人工排查时间|失败用例首次归因时间从 60min 降到 5min|
|提高归因准确率|Top\-1 归因准确率不低于 75%（对比人工标注）|
|归因自动化覆盖率|大于 90% 的失败用例自动产出归因结论|
|缩短反馈链路|测试完成到归因报告推送不超过 3min|



---



## 二、方案设计



### 方案架构总览



```Plain Text
方案一：轻量增强（1周）
  CI 嵌入 + 截图分析 + 飞书推送
        │
        │ 演进
        ▼
方案二：智能归因平台（3-4周）
  多轮分析 + 聚类归因 + 知识沉淀 + Web 展示
        │
        │ 演进
        ▼
方案三：Agent 驱动深度诊断（6-8周）
  多模态 + 代码关联 + Agent 工具链 + 自动提单
```



---



### 2\.1 方案一：轻量增强 — CI 嵌入 \+ 截图分析 \+ 通知推送



> **定位**：在现有 ai\_report\.py 基础上最小改动，实现「测试完即归因，归因即推送」
> 
> 



#### 核心架构



```Plain Text
pytest(CI) → allure-result+screenshots → ai_report.py(增强版) → 飞书/钉钉 Webhook
                                                │
                                          ┌─────┴──────┐
                                          │截图→Base64  │
                                          │多模态输入    │
                                          │Prompt微调   │
                                          │聚类预处理   │
                                          └────────────┘
```



#### 关键改动点



**1\. Jenkins 嵌入自动归因**



在 Jenkinsfile 的 post 阶段插入归因步骤，确保在 allure\-result 被清理之前执行：



```Groovy
post('Send Report') {
    always {
        // ... 现有 allure 生成逻辑 ...

        // 新增：AI 智能归因（必须在 allure-result 清理之前执行）
        script {
            try {
                sh """
                    python sugon_web/tools/ai_report.py \
                        --results-dir allure-result \
                        --provider dashscope \
                        --max-failures 10 \
                        --output reports/ai-test-summary.md
                """
                // 读取归因结果推送飞书
                def summary = readFile('reports/ai-test-summary.md')
                sendAISummary(summary)
            } catch (Exception e) {
                echo "AI 归因分析失败: ${e}"
            }
        }

        // 归因完成后再清理
        sh "rm -f allure-result/* || true"
    }
}
```



> 注意：当前 Jenkinsfile 中 allure\-result 会在 post 阶段被 rm 清空，AI 归因必须在 rm 之前执行。
> 
> 



**2\. 截图多模态分析**



当前 ai\_report\.py 对截图只记录"二进制附件"路径，未实际使用。增强为将截图编码后送入多模态模型：



```Python
def encode_screenshot(results_dir: Path, source: str) -> str | None:
    """将截图文件编码为 Base64"""
    img_path = results_dir / source
    if not img_path.exists():
        return None
    import base64
    return base64.b64encode(img_path.read_bytes()).decode("utf-8")

def build_failure_payload_v2(cases, results_dir, log_text, max_failures):
    """构建多模态分析载荷"""
    failed_cases = [c for c in cases if c.status in {"failed", "broken"}][:max_failures]
    parts = []
    for case in failed_cases:
        parts.append(f"## 失败用例: {case.name}")
        parts.append(f"模块: {case.feature or case.suite or '未分类'}")
        if case.status_message:
            parts.append(f"报错信息:\n{snippet(case.status_message, 1500)}")
        if case.status_trace:
            parts.append(f"堆栈信息:\n{snippet(case.status_trace, 2500)}")

        # 新增：截图多模态输入
        for att in case.attachments:
            if att.type.startswith("image"):
                b64 = encode_screenshot(results_dir, att.source)
                if b64:
                    parts.append(f"[截图附件: {att.name}]（已编码为多模态输入）")

        # 文本附件
        attach_preview = attachment_text(
            results_dir, case.attachments, max_items=3, max_chars=1200
        )
        if attach_preview:
            parts.append(f"附件内容:\n{attach_preview}")

        # 日志片段
        log_excerpt = find_case_log_excerpt(log_text, case.name)
        if log_excerpt:
            parts.append(f"相关日志片段:\n{snippet(log_excerpt, 1800)}")

        parts.append("")
    return "\n".join(parts).strip()
```



**3\. 失败用例聚类预处理**



在送入 LLM 前，先按错误模式做简单聚类，减少重复分析：



```Python
import re

def extract_error_pattern(text: str) -> str:
    """提取错误消息中的关键模式"""
    if not text:
        return "UNKNOWN"
    if re.search(r'TimeoutError|timeout|timed?\s*out', text, re.IGNORECASE):
        return "TIMEOUT"
    if re.search(r'ElementNotFound|waiting for selector|No element found', text, re.IGNORECASE):
        return "ELEMENT_NOT_FOUND"
    if re.search(r'AssertionError|assert\s+False|Assertion failed', text, re.IGNORECASE):
        return "ASSERTION"
    if re.search(r'ConnectionError|net::ERR|Connection refused', text, re.IGNORECASE):
        return "NETWORK"
    if re.search(r'HTTP.*5\d{2}|Internal Server Error|服务异常', text, re.IGNORECASE):
        return "API_ERROR"
    return "OTHER"

def cluster_failures(cases: list[TestCaseResult]) -> dict[str, list[TestCaseResult]]:
    """按错误消息的相似度聚类失败用例"""
    clusters = {}
    for case in cases:
        pattern = extract_error_pattern(case.status_message or case.status_trace)
        clusters.setdefault(pattern, []).append(case)
    return clusters
```



聚类后，同一组仅分析 1 个代表用例，其余用例引用结论，节省 Token。



**4\. 飞书富文本推送**



将归因结果格式化为飞书卡片消息：



```Python
def format_feishu_card(summary_md: str, build_info: dict) -> dict:
    """将 Markdown 归因结果格式化为飞书交互卡片"""
    failed = build_info["failed"]
    if failed > 3:
        risk_level, color = "高风险", "red"
    elif failed > 0:
        risk_level, color = "中风险", "orange"
    else:
        risk_level, color = "低风险", "green"

    return {
        "msg_type": "interactive",
        "card": {
            "header": {
                "title": {
                    "tag": "plain_text",
                    "content": f"AI 归因报告 | {build_info['job_name']} #{build_info['build_number']}"
                },
                "template": color
            },
            "elements": [
                {"tag": "markdown", "content": summary_md},
                {
                    "tag": "action",
                    "actions": [
                        {
                            "tag": "button",
                            "text": {"tag": "plain_text", "content": "查看 Allure 报告"},
                            "url": build_info["report_url"],
                            "type": "primary"
                        },
                        {
                            "tag": "button",
                            "text": {"tag": "plain_text", "content": "查看 AI 完整报告"},
                            "url": build_info["ai_report_url"],
                            "type": "default"
                        }
                    ]
                }
            ]
        }
    }
```



#### 方案一评估



|维度|评分|说明|
|---|---|---|
|开发周期|5/5|3\-5 天|
|改动量|5/5|仅改动 ai\_report\.py \+ Jenkinsfile|
|归因深度|3/5|单轮分析，无历史对比|
|可扩展性|2/5|单体脚本，难以扩展|
|提效效果|4/5|解决"人工触发"和"无通知"两大痛点|



---



### 2\.2 方案二：智能归因平台 — 多轮分析 \+ 聚类归因 \+ 知识沉淀



> **定位**：构建独立的归因服务，具备多轮分析、历史对比、知识积累、Web 可视化能力
> 
> 



#### 核心架构



```Plain Text
┌──────────────────────────────────────────────────┐
│            智能归因平台 (FastAPI)                   │
│                                                    │
│  ┌──────────┐    ┌────────────────────┐           │
│  │ 归因引擎  │    │   知识库服务        │           │
│  │ Analyzer │    │ KnowledgeService   │           │
│  └────┬─────┘    └────────┬───────────┘           │
│       │                   │                        │
│  ┌────┴─────┐    ┌───────┴──────────┐            │
│  │ Prompt   │    │  向量数据库        │            │
│  │ Manager  │    │ (ChromaDB/FAISS)  │            │
│  └──────────┘    └──────────────────┘             │
│                                                    │
│  ┌──────────┐    ┌────────────────────┐           │
│  │ 聚类服务  │    │  历史对比服务       │           │
│  │Clusterer │    │ HistoryService    │            │
│  └──────────┘    └────────────────────┘           │
└────────────┬─────────────────────────────────────┘
             │
   ┌─────────┼──────────┐
   │         │          │
┌──┴───┐ ┌──┴───────┐ ┌┴────────┐
│Jenkins│ │Dashboard │ │通知服务  │
│触发归因│ │可视化展示 │ │飞书/钉钉│
└──────┘ └──────────┘ └─────────┘
```



#### 核心模块设计



**模块 1：多轮归因引擎 \(Analyzer\)**



将现有的单轮调用改为「初筛 \- 深入 \- 确认」三阶段：



```Python
class AnalyzerEngine:
    """多轮归因引擎"""

    def analyze(self, case: TestCaseResult, context: AnalysisContext) -> AnalysisResult:
        # 第一轮：初筛分类（轻量，消耗少量 Token）
        classification = self._classify(case, context)

        # 第二轮：基于分类定向深入（每类问题有不同的分析侧重点）
        deep_result = self._deep_dive(case, context, classification)

        # 第三轮：与历史同类案例交叉验证
        verified = self._cross_validate(deep_result, context)

        return verified

    def _classify(self, case, context):
        """轻量分类：仅判断 环境问题/用例问题/产品缺陷"""
        prompt = CLASSIFICATION_PROMPT.format(
            case_name=case.name,
            error_message=snippet(case.status_message, 500),
            error_trace=snippet(case.status_trace, 800),
        )
        return self.llm.chat(prompt)

    def _deep_dive(self, case, context, classification):
        """基于分类定向深入"""
        if classification.category == "环境问题":
            prompt = ENV_DEEP_DIVE_PROMPT    # 关注环境指标、资源状态、网络连通
        elif classification.category == "用例问题":
            prompt = SCRIPT_DEEP_DIVE_PROMPT  # 关注定位器、等待策略、断言逻辑
        else:
            prompt = BUG_DEEP_DIVE_PROMPT     # 关注功能逻辑、状态流转、数据一致性
        return self.llm.chat(prompt, images=context.screenshots)

    def _cross_validate(self, result, context):
        """与历史知识库交叉验证"""
        similar_cases = self.knowledge.search(result.summary, top_k=3)
        if similar_cases:
            validation_prompt = VALIDATION_PROMPT.format(
                current=result.summary,
                historical=[c.summary for c in similar_cases],
            )
            return self.llm.chat(validation_prompt)
        return result
```



**模块 2：聚类归因服务 \(Clusterer\)**



当多个用例失败时，先聚类再分析，避免重复消耗 Token：



```Python
class FailureClusterer:
    """失败用例聚类服务"""

    def cluster(self, cases: list[TestCaseResult]) -> list[FailureCluster]:
        clusters = []

        # 1. 按错误模式硬聚类（快速分组）
        pattern_groups = self._group_by_error_pattern(cases)

        # 2. 对每组内的用例，使用 LLM 做语义聚类
        for pattern, group in pattern_groups.items():
            if len(group) <= 2:
                clusters.append(FailureCluster(
                    root_pattern=pattern,
                    cases=group,
                    representative=group[0],
                ))
            else:
                sub_clusters = self._semantic_cluster(group)
                clusters.extend(sub_clusters)

        return clusters
```



聚类策略示意：



```Plain Text
失败用例 12 个
    │
    ▼ 硬聚类（按错误模式）
┌──────────┬──────────┬──────────┬──────────┐
│ TIMEOUT  │ ASSERT   │ ELEMENT  │ NETWORK  │
│ 5 个     │ 3 个     │ 2 个     │ 2 个     │
└────┬─────┴────┬─────┴────┬─────┴────┬─────┘
     │          │          │          │
     ▼          ▼          ▼          ▼
  LLM语义    直接分析    直接分析    LLM语义
  子聚类x2     1个代表     1个代表     子聚类x1
  (各1代表)                          (1个代表)
     │          │          │          │
     ▼          ▼          ▼          ▼
  共 5 次 LLM 调用（而非 12 次），Token 节省约 60%
```



**模块 3：知识库服务 \(KnowledgeService\)**



积累历史归因结论，为新案例提供参照：



```Python
class KnowledgeService:
    """归因知识库"""

    def __init__(self):
        self.vector_store = ChromaDB(collection="failure_knowledge")
        self.embedding_model = DashScopeEmbedding()

    def store(self, result: AnalysisResult):
        """存储归因结论"""
        doc = Document(
            content=result.summary,
            metadata={
                "category": result.category,       # 环境问题/用例问题/产品缺陷
                "module": result.module,            # 数据库/计算/存储/网络
                "confidence": result.confidence,    # 高/中/低
                "timestamp": result.timestamp,
                "case_name": result.case_name,
            }
        )
        self.vector_store.add(doc)

    def search(self, query: str, top_k: int = 3) -> list[Document]:
        """检索相似历史案例"""
        return self.vector_store.query(query, top_k=top_k)

    def get_stats(self) -> dict:
        """获取知识库统计：各类问题占比、高频根因 Top10"""
        ...
```



**模块 4：历史对比服务 \(HistoryService\)**



对比本次与上次运行结果，识别新增失败和回归：



```Python
class HistoryService:
    """历史对比服务"""

    def compare(self, current: TestRun, previous: TestRun) -> ComparisonResult:
        return ComparisonResult(
            new_failures=self._find_new_failures(current, previous),    # 新增失败
            regressions=self._find_regressions(current, previous),      # 回归失败
            flaky_cases=self._find_flaky(current, previous),            # 不稳定用例
            fixed_cases=self._find_fixed(current, previous),            # 已修复用例
        )
```



历史对比策略：



|场景|含义|归因优先级|
|---|---|---|
|新增失败|上次通过，本次失败|高（可能是新变更引入）|
|持续失败|上次也失败，本次仍失败|中（可能已有缺陷单）|
|回归失败|之前通过，中间通过，又失败|高（回归缺陷）|
|不稳定|最近 5 次运行中失败 \>= 2 次|中（需关注稳定性）|



**模块 5：Web Dashboard**



基于 FastAPI \+ Vue/React 的轻量 Web 界面：



|页面|功能|
|---|---|
|归因总览|本次执行概览 \+ 风险判断 \+ 归因分布饼图|
|失败详情|每个失败用例的 AI 归因卡片（结论 \+ 证据链 \+ 修复建议）|
|历史趋势|失败率趋势 \+ 各类根因占比趋势|
|知识检索|搜索历史归因案例|
|反馈校正|人工标注归因是否准确，反馈回知识库|



#### 数据模型



```Python
@dataclass
class AnalysisResult:
    case_name: str
    full_name: str
    category: str           # 环境问题 / 用例问题 / 产品缺陷
    summary: str            # 1-2句话结论
    confidence: str         # 高 / 中 / 低
    fact_chain: list[str]   # 事实链路
    evidence: list[str]     # 关键证据
    excluded: list[str]     # 排除项
    reason: str             # 判断依据
    fix_short: str          # 短期建议
    fix_long: str           # 长期建议
    gaps: list[str]         # 信息缺口
    cluster_id: str         # 所属聚类
    similar_history: list   # 相似历史案例

@dataclass
class TestRun:
    build_id: str
    timestamp: datetime
    environment: dict       # 主机、版本、架构
    total: int
    passed: int
    failed: int
    results: list[AnalysisResult]
```



#### 方案二评估



|维度|评分|说明|
|---|---|---|
|开发周期|3/5|3\-4 周|
|改动量|2/5|新增服务，需部署|
|归因深度|4/5|多轮分析 \+ 历史对比|
|可扩展性|4/5|模块化服务，可增量扩展|
|提效效果|4/5|知识积累 \+ 可视化 \+ 反馈闭环|



---



### 2\.3 方案三：Agent 驱动深度诊断 — 多模态 \+ 代码关联 \+ 自动提单



> **定位**：将归因从"被动分析"升级为"主动诊断"，Agent 自主调用工具链收集证据，关联代码变更，最终自动提单
> 
> 



#### 核心架构



```Plain Text
┌─────────────────────────────────────────────────────┐
│                AI Agent 诊断引擎                      │
│                                                       │
│  ┌───────────┐    ┌─────────────┐    ┌───────────┐  │
│  │  Planner  │───▶│  Executor   │───▶│ Evaluator │  │
│  │  规划器   │    │  执行器     │    │  评估器   │  │
│  └───────────┘    └──────┬──────┘    └─────┬─────┘  │
│                          │                  │        │
│                   ┌──────┴──────┐    ┌──────┴────┐  │
│                   │  Tool Box   │    │  反馈循环  │  │
│                   │  工具箱     │    │ 不足则补充 │  │
│                   └─────────────┘    └───────────┘  │
└──────────────────────────┬──────────────────────────┘
                           │
     ┌─────────────────────┼───────────────────┐
     │                     │                   │
┌────┴──────┐     ┌────────┴──────┐    ┌──────┴───────┐
│代码变更服务│     │环境诊断服务    │    │ 工单服务     │
│ Git Diff  │     │ SSH/API 探针  │    │ 飞书/JIRA   │
└───────────┘     └───────────────┘    └──────────────┘
```



#### 核心创新点



**1\. Agent 工具链**



Agent 可自主调用以下工具收集证据，而非被动依赖已有数据：



|工具名|类别|说明|参数|
|---|---|---|---|
|git\_diff|代码|获取最近 N 次提交的代码变更|count|
|read\_test\_code|代码|读取失败用例的源代码|file\_path, line\_range|
|read\_page\_code|代码|读取 Page Object 代码|module\_name|
|ssh\_run|环境|在目标主机执行诊断命令|command|
|check\_service\_status|环境|检查 K8s Pod/Service 状态|namespace, service|
|check\_network|环境|检查网络连通性|source, target, port|
|search\_log|日志|在服务日志中搜索关键词|service, keyword, time\_range|
|get\_metrics|监控|获取系统指标（CPU/内存/磁盘）|host, time\_range|
|search\_knowledge|知识|在归因知识库中检索相似案例|query, top\_k|



**2\. Agent 诊断流程**



```Python
class DiagnosisAgent:
    """AI Agent 诊断引擎"""

    def diagnose(self, case: TestCaseResult, context: RunContext) -> DiagnosisReport:
        # Step 1: 规划 — Agent 自主决定需要收集哪些证据
        plan = self.planner.plan(
            case_info=case,
            available_tools=AGENT_TOOLS,
            max_steps=5,
        )
        # Agent 可能的规划示例：
        # Plan: [
        #   1. read_test_code  -> 看断言逻辑
        #   2. git_diff        -> 看最近变更
        #   3. ssh_run("kubectl get pod -n xscale") -> 看服务状态
        #   4. search_knowledge -> 查历史同类
        # ]

        # Step 2: 执行 — 按计划调用工具，收集证据
        evidence_chain = []
        for step in plan:
            tool_result = self.executor.execute(step)
            evidence_chain.append(tool_result)

            # Step 3: 评估 — 证据是否充足
            evaluation = self.evaluator.evaluate(evidence_chain)
            if evaluation.sufficient:
                break
            # 证据不足，Agent 自主补充
            plan = self.planner.replan(evaluation.gaps, plan)

        # Step 4: 归因 — 基于完整证据链输出结论
        report = self.evaluator.conclude(evidence_chain)
        return report
```



Agent 诊断示例 \-\- XScale 用户搜索失败：



```Plain Text
Agent 收到失败用例: XScale-用户列表页搜索
    │
    ├─ Step 1: 读取用例源码
    │  └─ 发现断言: assert "user_zm6ug" in search_results
    │
    ├─ Step 2: 读取 Page Object 源码
    │  └─ 发现搜索方法使用 locator.fill() + wait_for_timeout(2000)
    │
    ├─ Step 3: 检查环境状态
    │  └─ ssh_run("kubectl get pod -n xscale") -> Pod 全部 Running
    │
    ├─ Step 4: 查看最近代码变更
    │  └─ git_diff -> 发现 xscale.py 搜索方法最近修改了选择器
    │
    └─ 结论: 用例问题 — 搜索方法选择器变更导致定位不稳定
```



**3\. 代码变更关联**



当检测到失败用例时，自动关联最近的代码变更：



```Python
class CodeChangeCorrelator:
    """代码变更关联器"""

    def correlate(self, case: TestCaseResult, git_info: GitInfo) -> CorrelationResult:
        # 1. 定位失败用例涉及的模块
        module_path = self._resolve_module(case.full_name)
        # 例: test_xscale_basic -> pages/database/xscale.py

        # 2. 获取最近变更
        recent_changes = git_info.get_changes(days=7, paths=[module_path])

        # 3. 让 LLM 判断变更与失败是否相关
        if recent_changes:
            correlation = self.llm.chat(
                CODE_CORRELATION_PROMPT.format(
                    failure=case.status_message,
                    code_changes=recent_changes,
                )
            )
            return CorrelationResult(
                related_changes=correlation.related_commits,
                confidence=correlation.confidence,
                reasoning=correlation.reasoning,
            )
        return CorrelationResult(
            related_changes=[], confidence="low", reasoning="无近期变更"
        )
```



代码关联决策树：



```Plain Text
失败用例 -> 定位模块路径
              │
              ├── 近7天有变更 -> LLM 判断相关性
              │     │
              │     ├── 相关 -> 标记"可能由变更引入"+ 关联 commit
              │     └── 不相关 -> 按常规流程归因
              │
              └── 近7天无变更 -> 排除代码变更因素，侧重环境/产品分析
```



**4\. 自动提单**



当归因结论为"产品缺陷"且置信度 \>= 中时，自动创建缺陷单：



```Python
class IssueCreator:
    """自动提单服务"""

    def create_if_needed(self, report: DiagnosisReport) -> str | None:
        if report.category != "产品缺陷" or report.confidence == "低":
            return None

        # 检查是否已有同类未关闭缺陷
        existing = self.issue_tracker.search(
            keywords=report.case_name,
            status=["open", "in_progress"],
        )
        if existing:
            # 追加评论而非重复提单
            self.issue_tracker.add_comment(existing[0].id, report.to_comment())
            return existing[0].id

        # 创建新缺陷单
        issue_id = self.issue_tracker.create(
            title=f"[AI归因] {report.case_name} - {report.summary}",
            description=report.to_issue_description(),
            labels=["ai-attributed", report.module, f"priority-{report.suggested_priority}"],
            attachments=report.screenshot_paths,
        )
        return issue_id
```



**5\. 多模态融合分析**



截图 \+ DOM \+ 网络请求多维度交叉验证：



```Python
class MultiModalAnalyzer:
    """多模态融合分析"""

    def analyze_failure(self, case, screenshots, network_logs, dom_snapshots):
        messages = [
            {"role": "system", "content": MULTI_MODAL_PROMPT},
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": f"用例: {case.name}\n报错: {case.status_message}"},
                    # 失败截图
                    {
                        "type": "image_url",
                        "image_url": {"url": f"data:image/png;base64,{screenshots[0]}"}
                    },
                    # DOM 快照（文本形式）
                    {"type": "text", "text": f"DOM 快照:\n{snippet(dom_snapshots[0], 2000)}"},
                    # 网络请求日志
                    {"type": "text", "text": f"网络请求:\n{self._format_network_logs(network_logs)}"},
                ]
            }
        ]
        return self.llm.chat(messages)
```



多模态分析的价值 \-\- 以「按钮点击无效」为例：



|维度|单模态（仅日志）|多模态（截图\+DOM\+网络）|
|---|---|---|
|日志显示|点击了"删除"按钮|点击了"删除"按钮|
|截图显示|无法看到|按钮实际是 disabled 状态（灰色）|
|DOM 显示|无法看到|按钮有 disabled 属性，前端状态未更新|
|网络显示|无法看到|删除 API 未被调用|
|归因结论|可能是"定位失败"|正确归因："前端状态未更新，产品缺陷"|



#### 方案三评估



|维度|评分|说明|
|---|---|---|
|开发周期|2/5|6\-8 周|
|改动量|1/5|全新系统，需完整部署|
|归因深度|5/5|主动诊断 \+ 代码关联 \+ 自动提单|
|可扩展性|5/5|工具链可无限扩展|
|提效效果|5/5|端到端自动化，人工仅做最终确认|
|风险|\-|Agent 幻觉可能产生误操作；需严格限制工具权限|



---



## 三、方案对比与推荐



### 3\.1 综合对比



|对比维度|方案一：轻量增强|方案二：归因平台|方案三：Agent 诊断|
|---|---|---|---|
|开发周期|3\-5 天|3\-4 周|6\-8 周|
|部署复杂度|零（脚本级）|中（需部署 FastAPI \+ DB）|高（Agent \+ 多服务）|
|归因准确率预期|60\-70%|75\-85%|85\-90%|
|核心依赖|LLM API|LLM API \+ 向量数据库|LLM API \+ 向量数据库 \+ Agent 框架|
|Token 消耗/次|约 3K\-5K|约 8K\-15K（多轮）|约 15K\-30K（含工具调用）|
|适合阶段|快速验证|规模化使用|深度自动化|



### 3\.2 推荐路径：渐进式演进



```Plain Text
Phase 1 (第1周)           Phase 2 (第2-5周)           Phase 3 (第6-12周)
┌──────────────┐      ┌───────────────────┐      ┌──────────────────────┐
│  方案一落地    │─────▶│  方案二建设         │─────▶│  方案三 Agent 升级    │
│  - CI 嵌入    │      │  - 归因平台搭建     │      │  - 工具链扩展         │
│  - 截图分析   │      │  - 多轮分析引擎     │      │  - 代码变更关联       │
│  - 飞书推送   │      │  - 知识库建设       │      │  - 自动提单           │
│  - 验证效果   │      │  - Web Dashboard   │      │  - 多模态融合         │
└──────────────┘      └───────────────────┘      └──────────────────────┘
     验证价值               沉淀能力                  深度自动化
```



**Phase 1 的价值**：用最小成本验证"AI 归因是否真的能提效"。如果方案一已经能覆盖 60%\+ 的场景，再投入 Phase 2/3 才有意义。



**Phase 2 的价值**：解决单轮分析深度不足、无历史参照、分析结果无法积累的痛点。知识库越用越准。



**Phase 3 的价值**：从"被动分析"升级为"主动诊断"。Agent 可以在证据不足时自主补充收集，而非简单标注"信息缺口"。



---



## 四、技术选型建议



|组件|推荐方案|备选|理由|
|---|---|---|---|
|LLM|DashScope \(GLM\-4V\)|DeepSeek\-V3|多模态支持 \+ 国内低延迟|
|Embedding|DashScope text\-embedding|BGE\-M3|与 LLM 同生态|
|向量数据库|ChromaDB（嵌入式）|Milvus|初期数据量小，嵌入式够用|
|归因服务|FastAPI|Flask|异步支持好，自动文档|
|前端|Vue \+ ECharts|React \+ Recharts|团队技术栈匹配|
|Agent 框架|LangChain|自研|工具链生态成熟|
|存储|SQLite 到 PostgreSQL|\-|初期轻量，后续可迁移|



---



## 五、关键风险与对策



|风险|影响|对策|
|---|---|---|
|LLM 幻觉导致误归因|误导排查方向|三分类强制约束 \+ 置信度标注 \+ 人工复核入口|
|Token 成本失控|运营成本高|聚类去重（同类失败只分析1次）\+ 初筛用小模型 \+ 深入用大模型|
|截图/日志过大超 Token|分析截断|分层策略：初筛仅送摘要，深入阶段才送原文|
|环境敏感信息泄露|安全风险|日志脱敏 \+ Prompt 中禁止输出密码/Token|
|Agent 工具误操作|环境破坏|只读工具 \+ 审批机制 \+ 沙箱执行|
|allure\-result 被清理|数据丢失|CI 流程调整：归因在 rm 之前执行|



---



## 六、效果度量体系



|度量指标|目标值|采集方式|
|---|---|---|
|归因准确率（Top\-1）|\>= 75%|人工标注 \+ 反馈校正|
|推送时效|\<= 3min|CI 时间戳差值|
|人工复核率|\<= 30%|Dashboard 反馈数据|
|知识库命中率|\>= 60%|向量检索召回率|
|Token 单次成本|\<= 0\.5 元|API 调用统计|
|提效量化|排查时间降 80%|前后对比人工耗时|



---



## 附录



### A\. 现有 Prompt 模板核心约束



- 根因分类三选一：环境问题 / 用例问题 / 产品缺陷

- 输出七段式结构：结论 \- 事实链路 \- 关键证据 \- 排除项 \- 判断依据 \- 修复建议 \- 信息缺口

- 分析原则：先还原事实再下结论；区分报错表象和真实根因；证据不足不强定性

    

### B\. 现有 allure\-result 数据结构



|文件类型|命名规则|内容|
|---|---|---|
|result 文件|UUID\-result\.json|用例名称、状态、步骤、报错信息、附件引用|
|container 文件|UUID\-container\.json|fixture 层次、before/after 执行状态|
|文本附件|UUID\-attachment\.txt|步骤日志、失败信息、完整运行日志|
|图片附件|UUID\-attachment\.png|失败截图|



### C\. CI 集成注意事项



1. AI 归因必须在 allure\-result 被清理之前执行

2. 归因脚本失败不应阻塞整体流水线（try\-catch 包裹）

3. 归因结果需持久化存储（reports/ 目录），不随 allure\-result 清理而丢失

4. 飞书推送需处理 Markdown 长度限制（超过 4000 字符需截断或分段）

