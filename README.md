# AgentCraft-Claw

<p align="center">
  <img src="https://img.shields.io/badge/Spring%20Boot-3.4.13-6DB33F?logo=spring-boot" />
  <img src="https://img.shields.io/badge/Vue%203-3.5.29-4FC08D?logo=vue.js" />
  <img src="https://img.shields.io/badge/Spring%20AI-1.1.4-6DB33F?logo=spring" />
  <img src="https://img.shields.io/badge/Milvus-2.3.4-00A3E0?logo=apache" />
  <img src="https://img.shields.io/badge/Java-17-007396?logo=java" />
  <img src="https://img.shields.io/badge/Node-20%2B-339933?logo=node.js" />
</p>


<p align="center">
  <b>AgentCraft-Claw</b> — 下一代 Agentic 智能知识系统。以目标驱动的 Craft Engine 为核心，融合多 Agent 协作编排、七步 RAG 检索链、多模态理解、私有化模型部署、MCP 工具生态与 TechBlogHub 自动知识库，打造具备自主工作能力的 AI 智能体平台。
</p>


---

## 核心能力

AgentCraft-Claw 突破传统"文档问答"边界，构建具备自主规划、执行与交付能力的智能体系统：

- **智能问答（Chat）** — 基于七步检索链（Vector + BM25 + RRF + Rerank + Compression）的精准 RAG；新版 `CraftEngine` 流式生成采用完全异步 `.subscribe()` 不阻塞 Tomcat 线程，旧版 `RagPipeline` 仍保留 `.blockLast()` 同步路径，支持 SSE 流式输出、来源可溯与答案句子级溯源
- **答案句子级溯源（Provenance）** — `ProvenanceService` 将答案按句切分，逐句匹配最相关召回切片，生成 `{sentenceIndex, sentence, references[]}` 结构化溯源；`SourcePayloadFactory` 统一封装 chunkId、来源文档、页码、章节、知识库 ID 与网页 URL
- **智能文档分片** — `TextChunker` 从 AI 配置中心读取 `rag.chunk_size` / `rag.chunk_overlap`，保留 PDF 页码与章节元数据，支持 FAQ / 代码 / 标题感知等分片策略，为检索与溯源提供高质量片段
- **任务工坊（Craft）** — 目标驱动的任务执行引擎，支持三种执行模式：简单查询、复杂任务（OTAC 循环 + Swarm 多 Agent 协作）、人机协作（检查点暂停确认）
- **RAG 评测平台（管理员）** — 构建评测数据集、批量导入标准问题与期望命中切片，异步执行 RAG 流水线并计算 Recall@K / MRR / Context Precision / Answer Relevance / Faithfulness，支持 CSV / JSON 结果导出
- **统一审计中心（管理员）** — 按时间、用户、会话、反馈、兜底状态等多维度查询 `qa_message`，内容预览自动对手机号与邮箱脱敏，支持 CSV / JSON 导出
- **Trace 追踪可视化（管理员）** — 查看单条消息的 Agent Trace、MCP 调用、Reflection Log、Provenance 以及 Craft 执行轨迹、使用 Agent / 工具与耗时
- **知识库管理** — 支持 PDF/DOCX/MD/TXT 多格式上传，基于独立线程池异步解析、智能分片、向量化索引，具备并发去重与失败重试能力；引入 `KbDocumentVersion` 版本表与文件内容哈希，实现增量更新（增删改 diff），避免全量删除重建
- **知识库版本管理（前端）** — 版本列表、发布、回滚、Diff 对比页面（`KbVersionView.vue`），行级 Diff 高亮基于 LCS 算法，增删改可视化，UI 采用 iOS 26 Liquid Glass 玻璃材质设计
- **联网检索** — 集成 Tavily 实时网页搜索，知识库与互联网双源交叉验证，确保信息时效性
- **WebBridge 浏览器自动化** — 基于 Playwright 的真实 Chromium 驱动，支持网页浏览、内容提取、截图、点击、填写、滚动与联网搜索；通过 `BrowserClaw` 以 MCP 工具形式暴露给 Agent 与外部客户端，URL 白名单 + HarnessEngine 双重拦截
- **A/B 实验平台** — 管理员可创建多类型实验与多 variant，稳定哈希按用户 ID 分流；`AiConfigHolder` 按 variant 缓存独立 `ChatModel`，实现模型/温度等参数在线实验；前端 `/ab-experiments` 管理后台实时查看实验指标
- **语音输入** — 实时 WebSocket 语音转文字（DashScope Paraformer），支持自然语音交互
- **多模态 RAG** — 支持图片、PDF 截图作为问题输入，OCR 文字提取（Tesseract / Ollama 多模态），多模态 Embedding（图片+文本联合向量），文本检索与视觉检索并行融合（RRF），Milvus 双 Collection（文本 + 图片）
- **私有化模型一体机** — 支持 vLLM / Ollama / TGI 本地大模型部署，通过 OpenAI-compatible API 统一接入；`AiConfigHolder` 扩展 `providerType`（CLOUD / VLLM / OLLAMA / LOCAL），双层缓存（variantModelCache + localModelCache）；TEI（Text Embeddings Inference）私有化 Embedding 服务；Spring Boot Actuator 健康检查（`LocalModelHealthIndicator`）
- **自适应规划（Self-Evolving Planner）** — 基于历史任务成功率自动优化 Planner Prompt 与工具选择策略；`PlannerMetricsCollector` 收集计划生成耗时、步骤成功率、工具成功率矩阵；`PlannerEvolutionScheduler` 定时进化（每天凌晨 3:17）+ 阈值自动触发（成功率<70%）；进化日志版本管理支持 A/B 对比与回滚；策略写入 Procedural Memory（全局策略模板）
- **Agent 间协作协议** — 与 CrewAI、AutoGen、Dify 等外部 Agent 平台互调；`AgentRegistryService` 实现外部 Agent 注册/发现/能力匹配（Redis 存储 + 心跳检测）；`AgentTaskDelegator` 支持任务委托（同步轮询 + 异步回调），委托前经过 HarnessEngine 安全检查；平台适配器：CrewAIAdapter、AutoGenAdapter、DifyAdapter；回调签名采用 HMAC-SHA256 防伪造
- **企业级 RBAC 与 SSO** — 资源级权限控制：角色层级继承、权限树、数据权限（ALL / OWN / DEPT / TEAM）；自定义注解 `@RequirePermission` + AOP 拦截；SSO 支持 LDAP/AD、OAuth2/OIDC；自动用户同步（SSO 登录后自动创建本地账号）
- **MCP 工具链** — 通过 MCP 协议对外暴露工具能力，供外部 AI 客户端与自动化流程调用；`OpenClawController` 提供外部任务提交入口
- **AI 配置中心** — 数据库驱动参数动态热更新，API Key、模型选择、联网开关等无需重启服务
- **数据大屏** — `StatsServiceImpl` 统一聚合真实 `qa_message` / `kb_knowledge_base` / `mcp_tool_registry` / `tech_blog_article` 数据，Controller 仅做参数校验与结果包装；后端内置 Micrometer 可观测指标
- **iOS 26 Liquid Glass** — 前端采用液态玻璃设计语言，动态流体渐变、多层折射高光与 Spring 动效
- **Tech Blog Hub** — 一句话指令驱动，自动抓取 Anthropic / OpenAI / Google DeepMind / Meta AI 等 AI 大厂技术博客，经翻译、摘要、标签化、Reviewer 评分、向量化后生成可离线浏览的静态知识库站点

---

## 架构设计

```
+------------------------------------------------------------------+
|                        接入层 (Access Layer)                      |
|  +--------+  +--------+  +-------------+  +------------------+  |
|  | REST   |  | SSE    |  |  WebSocket  |  | MCP / OpenClaw   |  |
|  | API    |  | Stream |  |  Speech     |  | Protocol         |  |
|  +--------+  +--------+  +-------------+  +------------------+  |
|  +-------------------+  +------------------------------------+  |
|  | Multimodal Input  |  | Agent Interop Gateway              |  |
|  | (Image / PDF)     |  | (CrewAI / AutoGen / Dify)         |  |
|  +-------------------+  +------------------------------------+  |
+------------------------------------------------------------------+
|                        编排层 (Orchestration)                     |
|  +--------------+  +------------------+  +---------------------+   |
|  | CraftEngine  |  | SelfEvolvingPlanner|  | SwarmOrchestrator  |   |
|  | (OTAC 循环)  |  | (自适应规划)       |  | (多 Agent 协作编排) |   |
|  +--------------+  +------------------+  +---------------------+   |
|  +--------------+  +--------------+  +---------------------+   |
|  | LoopEngine   |  | HarnessEngine|  | OpenClawGateway    |   |
|  | (执行循环)   |  | (约束治理)   |  | (对外工具协议)      |   |
|  +--------------+  +--------------+  +---------------------+   |
+------------------------------------------------------------------+
|                        智能体层 (Agent Layer)                     |
|  Planner | Researcher | Analyst | Writer | Coder | Reviewer |  |
|  | Browser |                                                          |
+------------------------------------------------------------------+
|                        工具层 (Claw Tools)                        |
|  +--------------+  +------------+  +----------+  +------------+|
|  | KnowledgeClaw|  | ResearchClaw|  | CodeClaw |  | BrowserClaw||
|  | (七步检索链)  |  | (多源研究)  |  | (代码)   |  | (浏览器)   ||
|  +--------------+  +------------+  +----------+  +------------+|
|  +-------------------+  +-------------------------------------+|
|  | MultimodalClaw    |  | TechBlogHub（RSS/Sitemap/API 发现   |
|  | (OCR+视觉检索)     |  | → 抓取 → 翻译 → 摘要 → 标签 → 索引   |
|  +-------------------+  | → 离线站点导出）                       |
|                         +-------------------------------------+|
+------------------------------------------------------------------+
|                        记忆层 (Memory)                            |
|  +----------+  +----------+  +----------+  +-----------------+  |
|  | Working  |  | Semantic |  | Episodic |  | Procedural      |  |
|  | Memory   |  | Memory   |  | Memory   |  | Memory          |  |
|  | (Redis)  |  | (MySQL)  |  | (MySQL)  |  | (MySQL)         |  |
|  +----------+  +----------+  +----------+  +-----------------+  |
+------------------------------------------------------------------+
|                        基础设施 (Infrastructure)                  |
|  Spring Boot | Spring AI | MyBatis Plus | MySQL | Redis | Milvus |  |
|  | MinIO | Prometheus | Grafana | Loki | Alertmanager |            |
|  | vLLM / Ollama / TGI / TEI (Local LLM) |                        |
+------------------------------------------------------------------+
```

### 三种执行模式

| 模式              | 触发条件                     | 执行流程                                                     | 适用场景                   |
| ----------------- | ---------------------------- | ------------------------------------------------------------ | -------------------------- |
| **SIMPLE**        | 短问题、疑问词开头           | 直接调用 ReAct Agent 引擎 -> 知识库检索 -> 即时回答          | "什么是 RAG？"             |
| **COMPLEX**       | 包含分析/生成/报告等复杂动词 | AutoPlanner -> Swarm 编排 -> 多 Agent 协作 -> LoopEngine 迭代 -> 结果综合 | "生成一份 AI 行业研究报告" |
| **COLLABORATIVE** | 包含"帮我确认""先给我看看"   | 计划 -> 检查点暂停 -> 人类确认 -> 继续执行                   | "帮我确认这个方案是否可行" |

### Swarm 深度执行

复杂任务（COMPLEX 模式）由 `SwarmOrchestrator` 驱动多 Agent 协作，核心机制包括：

- **DAG 拓扑并行**：`AutoPlanner` 生成的步骤列表经 `DagScheduler` 使用 Kahn 算法分层，无依赖步骤在同一层并行执行（`CompletableFuture` + 固定线程池），依赖步骤按层串行等待。
- **Reviewer 深度校验**：检查点步骤触发 Reviewer Agent，按 **事实性 / 完整性 / 安全性 / 格式** 四个维度输出结构化评分与改进建议；未通过时可暂停等待人工介入（COLLABORATIVE 模式）或跳过非关键步骤。
- **长期记忆召回**：每个步骤执行前，`MemoryRecallService` 从 **Working（当前 Craft 上下文）、Semantic（事实/偏好/约束，MySQL 持久化）、Episodic（历史步骤经验，MySQL 持久化）、Procedural（全局策略模板，MySQL 持久化）** 四层记忆中多路召回，经 RRF 融合后写入工作上下文；执行结束后自动沉淀语义事实与情景经验。

```
+-----------+     +-----------+     +-----------+
|  Step A   |---->|  Step C   |---->|  Step E   |
+-----------+     +-----------+     +-----------+
       |                ^
       v                |
+-----------+     +-----------+
|  Step B   |---->|  Step D   |
+-----------+     +-----------+

层 1: A, B 并行
层 2: C, D 并行（依赖 A/B）
层 3: E 串行（依赖 C/D）
```

### 执行循环与约束治理

#### LoopEngine（执行循环）

复杂任务在执行过程中由 `LoopEngine` 持续监控，防止无限循环与预算失控：

- **Otac Loop**：按 Observe-Think-Act-Check 四步循环推进任务，每次 Act 后检查执行结果是否满足退出条件。
- **Reviewer Loop**：Reviewer Agent 对关键输出进行事实性 / 完整性 / 安全性 / 格式四维度评分；未通过时生成改进建议并重新执行。
- **Budget Loop**：按 Token 数量、步骤数、耗时三个维度跟踪预算，超过阈值时触发降级或暂停。
- **Reflection Loop**：自纠错循环，最多执行 2 轮，每轮评估答案置信度，未通过则补充检索或改写问题。

Loop 优先级：`OtacLoop` > `ReviewerLoop` > `ReflectionLoop` > `BudgetLoop`。

#### HarnessEngine（约束治理）

`HarnessEngine` 在输入进入执行体前、输出发送给用户前、以及执行各关键节点分别进行约束检查：

- **PRE_INPUT**：敏感词过滤、Prompt 注入检测、权限校验、输入长度与格式约束。
- **PRE_EXECUTION**：计划/URL 预检查、预算与配额预校验。
- **PER_STEP**：步骤级预算、输出、URL 白名单检查。
- **POST_OUTPUT**：事实一致性校验、安全兜底、输出格式检查、禁忌词二次过滤。
- 任一阶段触发约束时返回结构化 `HarnessDecision`，上层根据 `BLOCK` / `HUMAN` / `DEGRADE` / `WARN` / `ALLOW` 决定放行、告警或拦截。

#### Loop / Harness 给系统带来的收益

| 维度         | 收益                                                         | 典型场景                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **可靠性**   | LoopEngine 防止任务陷入死循环，Reviewer 与 Reflection 保证输出质量下限 | 复杂报告生成时自动重试低质量步骤                             |
| **可控性**   | BudgetTracker 按 Token / 步骤 / 耗时三维设限，避免单次请求耗尽资源 | 用户提交"分析全站日志"等可能产生超长链的任务                 |
| **安全性**   | HarnessEngine 在 LLM 前后各加一道闸，Prompt 注入与有害输出被拦截 | 用户尝试越狱或询问违禁内容                                   |
| **可观测性** | 每次 Loop 轮次、Harness 决策都通过 Micrometer 埋点，可在大盘追踪 | Grafana 中观察 `craft.loop.round`、`harness.decision.total` 等指标 |
| **可审计**   | Harness 决策结果与 Loop 日志写入 `qa_message` 的 `agent_trace` / `reflection_log` | 事后复盘某次回答为何被拦截或重试                             |

#### 与其他组件的关联

```
┌─────────────────────────────────────────────────────────────────┐
│                          用户 / 外部客户端                        │
└─────────────┬───────────────────────────────────────────────────┘
              │ 请求
              ▼
┌─────────────────────────┐    PRE_INPUT    ┌─────────────────────┐
│   CraftChatController   │ ───────────────> │   HarnessEngine     │
│    /api/v2/chat/stream  │                  │  输入约束治理        │
└─────────────┬───────────┘                  └──────────┬──────────┘
              │                                         │ 通过/拦截
              ▼                                         ▼
┌─────────────────────────┐                         ┌─────────────┐
│      CraftEngine        │ <────── 驱动 OTAC ────> │ LoopEngine  │
│  (SIMPLE/COMPLEX/COLLAB)│        Reviewer/Budget  │ 执行循环    │
└─────────────┬───────────┘                         └─────────────┘
              │
              ▼
┌─────────────────────────┐      POST_OUTPUT     ┌─────────────────────┐
│   SwarmOrchestrator     │ ──────────────────>  │   HarnessEngine     │
│  (多 Agent DAG 并行)     │                      │  输出约束治理        │
└─────────────────────────┘                      └──────────┬──────────┘
                                                            │ 通过/拦截
                                                            ▼
┌─────────────────────────┐      工具调用       ┌─────────────────────┐
│     McpToolRegistry     │ <──────────────── │   OpenClaw/MCP      │
│  (@Tool 注解注册)        │                   │  对外协议层          │
└─────────────────────────┘                   └─────────────────────┘
```

- **与 CraftEngine**：`CraftEngine` 在 SIMPLE/COMPLEX/COLLABORATIVE 三种模式的执行入口调用 `LoopEngine.run()`，由 Loop 决定继续、重试还是暂停；同时把用户输入与最终输出交给 `HarnessEngine` 校验。
- **与 SwarmOrchestrator**：复杂任务进入 Swarm 后，每个 DAG 层执行前后、`ReviewerLoop` 审查前后都会经过 Harness 检查；Harness 拦截会终止当前层并返回决策给 `CraftEngine`。
- **与 SafetyGuard / GuardRailService**：Harness 的 PRE_INPUT / POST_OUTPUT 约束底层复用 `SafetyGuard` 的紧急词检测与 `GuardRailService` 的语义护栏，形成"规则 + 语义 + 策略"三层防护。
- **与 MCP / OpenClaw**：外部通过 OpenClaw 调用的工具请求同样先经过 Harness 输入检查，工具结果返回前再经过输出检查；违规调用会被记录到审计日志。
- **与 MetricsService / Grafana**：`LoopEngine` 上报 `craft.loop.round`、`craft.reviewer.retry`；`HarnessEngine` 上报 `harness.decision.total`（按 `level`、`decision` 标签），与 RAG、LLM、TechBlogHub 指标在同一大盘展示。
- **与 Memory**：Loop 的 Reviewer 结果与 Reflection 日志写入 `SharedMemoryBus` 的当前 Craft 上下文，并持久化到 `qa_message`，支持长期经验回放。

### LLM 调用韧性治理

大模型调用是系统的核心依赖，也是最容易出现延迟飙升、配额耗尽或服务不可用的环节。系统通过 `ResilientChatModel` 对 `ChatModel` 做统一包装，接入 Resilience4j 的多层保护：

- **熔断（CircuitBreaker）**：按失败率/慢调用率自动打开熔断，避免故障扩散；触发后返回结构化降级响应。
- **限流（RateLimiter）**：按配置限制单位时间内的 LLM 请求数，超出时返回友好提示。
- **隔离（Bulkhead）**：限制 LLM 并发调用数，防止单点依赖耗尽线程资源。
- **超时（TimeLimiter）**：对同步 `call()` 设置可配置超时，超时时触发降级。
- **降级（Fallback）**：熔断/限流/隔离/超时触发后，统一返回带提示语的 `ChatResponse`，不向上抛异常导致 SSE 断连。

`ResilientChatModel` 同时覆盖同步 `call(Prompt)` 与流式 `stream(Prompt)` 两条路径：同步路径使用 `Callable` 装饰器链；流式路径使用 Resilience4j Reactor Operator 在 Flux 上应用限流、隔离与熔断。`LlmResilienceConfig` 暴露 `llmCircuitBreaker`、`llmRateLimiter`、`llmBulkhead`、`llmTimeLimiter` 四个命名实例，所有阈值均通过 `llm.resilience.*` 配置项外置到 `application.yml`，支持运行时调整而无需改代码。

### 多模态 RAG

系统支持将图片、PDF 截图直接作为问题输入，实现"看图问答"能力：

- **OCR 文字提取**：支持 Tesseract 本地 OCR 与 Ollama 多模态模型（如 LLaVA）提取图片中的文字内容，按 `MULTIMODAL_OCR_ENGINE` 环境变量切换
- **多模态 Embedding**：图片经多模态 Embedding 服务（图片+文本联合向量）生成向量，与文本向量对齐到同一语义空间
- **双 Collection 存储**：Milvus 中独立维护文本 Collection（`kb_text_collection`）与图片 Collection（`kb_image_collection`），图片记录 `image_vector`、`ocr_text`、`source_doc_id`、`page_number`
- **并行检索融合**：文本检索（Vector + BM25）与视觉检索（图片向量相似度）并行执行，经 RRF 融合后统一送入重排序与上下文压缩
- **接口**：`POST /api/v2/multimodal/chat` SSE 流式输出，请求体支持 `image`（Base64 或 URL）与 `message` 组合

### 私有化模型接入

系统支持通过 vLLM、Ollama、TGI 等框架在本地或私有云部署大模型，降低对外部云 API 的依赖：

- **统一 OpenAI-compatible API**：本地模型通过 OpenAI 兼容 API 暴露，`AiConfigHolder` 统一封装为 `ChatModel`，对上层完全透明
- **ProviderType 扩展**：`AiConfigHolder` 新增 `providerType` 枚举（CLOUD / VLLM / OLLAMA / LOCAL），支持同一系统同时接入云端与多种本地模型
- **双层缓存**：`variantModelCache`（A/B 实验 variant 级别）+ `localModelCache`（本地模型实例级别），避免重复构建 `ChatModel`；配置变更时自动失效重载
- **TEI 私有化 Embedding**：通过 HuggingFace Text Embeddings Inference（TEI）服务私有化部署 Embedding 模型，替代云端 Embedding API
- **健康检查**：`LocalModelHealthIndicator` 实现 Spring Boot Actuator `HealthIndicator`，周期性探测本地模型连通性，状态异常时自动从路由池摘除
- **管理员接口**：本地模型 CRUD（`POST /api/admin/local-models` 等）+ 连通性测试（`POST /api/admin/local-models/{id}/test`）

### 自适应规划（Self-Evolving Planner）

`SelfEvolvingPlanner` 让任务规划能力随系统运行自我进化，持续优化 Planner Prompt 与工具选择策略：

- **指标收集**：`PlannerMetricsCollector` 收集计划生成耗时、步骤成功率、工具成功率矩阵、任务整体成功率等维度指标
- **定时进化**：`PlannerEvolutionScheduler` 每天凌晨 3:17 执行一次进化任务，自动分析近期指标并生成优化后的 Prompt 模板与工具选择权重
- **阈值触发**：当整体成功率低于 70% 时，自动触发紧急进化，无需等待定时窗口
- **版本管理**：`PlannerEvolutionLog` 记录每次进化前后的策略、指标对比、生效时间，支持 A/B 对比与一键回滚
- **策略沉淀**：进化后的策略模板写入 Procedural Memory（全局策略模板），供所有后续 Craft 任务共享
- **安全边界**：进化过程受 HarnessEngine 约束，新策略需通过 Reviewer 校验后才可生效，防止策略退化

### Agent 协作协议

系统通过 `AgentInteropGateway` 与 CrewAI、AutoGen、Dify 等外部 Agent 平台实现互调，构建开放的 Agent 生态：

- **注册与发现**：`AgentRegistryService` 提供外部 Agent 注册、发现与能力匹配；注册信息存储于 Redis，支持 TTL 心跳检测，失效 Agent 自动下线
- **能力匹配**：委托任务前按能力标签（`capabilityTags`）与信任评分匹配最适合的外部 Agent
- **任务委托**：`AgentTaskDelegator` 支持同步轮询（阻塞等待结果）与异步回调（委托后通过 callback 接收结果）两种模式；委托前任务描述经过 HarnessEngine 安全检查，防止敏感信息外泄
- **平台适配器**：`CrewAIAdapter`、`AutoGenAdapter`、`DifyAdapter` 封装各平台特有的调用协议与鉴权方式，上层统一通过 `AgentInteropGateway` 调用
- **回调安全**：异步回调携带 HMAC-SHA256 签名（`X-Interop-Signature`），接收端校验防伪造
- **对外端点**：
  - `POST /api/openclaw/agents/delegate` — 提交跨平台委托任务
  - `POST /api/openclaw/agents/callback` — 接收外部 Agent 回调结果

### RBAC 权限模型

系统实现企业级资源级权限控制，满足多租户与组织架构需求：

- **角色层级继承**：角色支持父子继承（`parent_role_id`），子角色自动继承父角色的权限，避免重复配置
- **权限树**：权限以树形结构组织（`permission_tree` 表），支持菜单、按钮、API、数据四级粒度
- **数据权限**：在角色-权限基础上增加数据范围（`data_scope`）：ALL（全部）、OWN（仅自己）、DEPT（本部门）、TEAM（本团队），与组织架构联动
- **注解驱动**：`@RequirePermission` 自定义注解 + AOP 拦截，直接标注在 Controller 方法上，声明式控制访问权限；支持 `logical = AND / OR` 组合多个权限
- **动态鉴权**：权限变更后通过 Redis 发布订阅实时刷新各节点缓存，无需重启服务
- **管理员接口**：角色管理、权限树管理、用户角色分配、数据权限配置

### SSO 单点登录

系统支持与企业现有身份体系对接，实现无缝登录体验：

- **LDAP/AD 认证**：支持标准 LDAP / Active Directory 协议，配置 `LDAP_URL`、`LDAP_BASE_DN`、`LDAP_BIND_CREDENTIAL` 等参数即可接入
- **OAuth2/OIDC**：支持标准 OAuth2 授权码模式与 OIDC 身份令牌，可对接企业微信、钉钉、飞书、Keycloak 等身份提供商
- **自动用户同步**：SSO 登录成功后，系统自动查询用户信息并在本地创建/更新账号（`sys_user`），首次登录自动赋予默认角色
- **配置化**：SSO 开关与提供商参数通过 `.env` 环境变量配置，无需改代码即可切换身份源
- **管理员接口**：SSO 配置查看与连通性测试（`POST /api/admin/sso/test`）

### OpenClaw 协议

OpenClaw 是系统对外暴露工具能力的开放协议，兼容 MCP（Model Context Protocol）规范：

- **工具注册**：后端通过 `@Tool` 注解将能力注册到 `McpToolRegistry`，运行时自动发现；TechBlogHub 工具（`BlogDiscoveryTool`、`ContentFetchTool`、`TranslationTool`、`SummarizeTool`、`TaggingTool`）也统一注册。
- **工具发现**：外部客户端可通过 `GET /api/mcp/tools` 获取工具元数据、参数模式与示例。
- **工具调用**：`POST /api/mcp/tools/{id}/test` 支持在线测试，`/api/mcp/invoke` 供授权客户端以 JSON-RPC 风格调用。
- **外部任务提交**：`POST /api/openclaw/tasks` 允许外部 Agent 提交目标，由 `OpenClawGateway` 转发至 `CraftEngine` 执行，并通过 callbackUrl 回推事件。
- **安全边界**：所有外部调用均需 JWT 认证，敏感工具额外校验角色权限；调用结果写入审计日志。

---

## 技术栈

### 后端 (Spring Boot 3.4)

| 层               | 技术                                                         | 说明                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 框架             | Spring Boot 3.4.13 + Spring AI 1.1.4                         | AI 原生开发框架                                              |
| 安全             | Spring Security + JWT (jjwt 0.12.5) + RBAC/SSO               | 无状态认证，BCrypt 加密，HttpOnly Cookie，SSE 一次性票据，LDAP/AD/OAuth2/OIDC |
| ORM              | MyBatis Plus 3.5.7                                           | 高效 CRUD + 条件构造器                                       |
| 向量数据库       | Milvus 2.3.4                                                 | 1024 维向量检索，双 Collection（文本 + 图片）                |
| 缓存             | Redis 7 + Lettuce                                            | 频率缓存 + 会话记忆 + SSE 票据 + Agent 注册表                |
| 对象存储         | MinIO 8.5.7                                                  | 文档文件存储                                                 |
| 文档解析         | Apache PDFBox 3.0 + POI 5.2                                  | PDF/DOCX 提取                                                |
| 关键词搜索       | Lucene 8.11 + smartcn 分词                                   | 中文 BM25 检索                                               |
| OCR 引擎         | Tesseract 5.x / Ollama 多模态                                | 图片/PDF 截图文字提取                                        |
| 多模态 Embedding | Ollama 多模态 / 兼容服务                                     | 图片+文本联合向量                                            |
| 本地大模型       | vLLM / Ollama / TGI                                          | OpenAI-compatible API 私有化部署                             |
| 本地 Embedding   | TEI (Text Embeddings Inference)                              | HuggingFace 私有化 Embedding 服务                            |
| 韧性治理         | Resilience4j 2.1.0                                           | LLM / 搜索 / 工具调用的熔断、限流、隔离、超时与降级          |
| 语音识别         | DashScope Paraformer                                         | 实时流式 ASR                                                 |
| 联网搜索         | Tavily API                                                   | 实时网页检索                                                 |
| AI 配置          | AiConfigHolder + AiConfigInitializer                         | 数据库驱动热更新，原子替换模型实例，支持 CLOUD/VLLM/OLLAMA/LOCAL 多 provider |
| 博客生态         | ROME 2.1.0 / jsoup 1.17.2 / flexmark 0.64.8 / Playwright 1.44.0 | RSS/Sitemap/正文抓取/翻译/摘要                               |
| 可观测性         | Micrometer Prometheus + Loki Logback Appender 1.5.2          | 指标与日志                                                   |

### 前端 (Vue 3 + Vite)

| 层       | 技术                                   | 说明                                                      |
| -------- | -------------------------------------- | --------------------------------------------------------- |
| 框架     | Vue 3.5.29 + Vite 7.3                  | 组合式 API                                                |
| UI 库    | Element Plus 2.13.5                    | 组件按需自动导入（unplugin-vue-components），减少打包体积 |
| 状态管理 | Pinia 3.0.4                            | 类型安全 Store                                            |
| 路由     | Vue Router 5.0.3                       | 动态路由 + 管理员守卫                                     |
| 设计系统 | iOS 26 Liquid Glass                    | 液态玻璃、动态渐变、彩色光晕                              |
| 图表     | ECharts 6.0                            | 数据可视化                                                |
| Markdown | marked 17.0 + highlight.js + DOMPurify | 富文本渲染 + XSS 过滤                                     |
| 工具库   | VueUse 14.2                            | 组合式工具                                                |

### 基础设施 (Docker Compose)

| 服务                | 容器端口            | 宿主机端口 | 说明                                  |
| ------------------- | ------------------- | ---------- | ------------------------------------- |
| MySQL               | 3306                | 3307       | 业务数据库                            |
| Redis               | 6379                | 6380       | 缓存 + 会话 + SSE 票据 + Agent 注册表 |
| Milvus              | 19530               | 19530      | 向量数据库                            |
| MinIO API           | 9000                | 9002       | 对象存储 API                          |
| MinIO 控制台        | 9001                | 9003       | 对象存储控制台                        |
| 后端                | 8080                | 8080       | REST + SSE API                        |
| 前端                | 80                  | 80         | 主入口                                |
| Prometheus          | 9090                | 9090       | 指标抓取与查询                        |
| Grafana             | 3000                | 3000       | 可观测性大盘                          |
| Loki                | 3100                | 3100       | 日志接收端                            |
| Alertmanager        | 9093                | 9093       | Prometheus 告警通知管理               |
| etcd                | 2379                | —          | Milvus 元数据依赖                     |
| vLLM / Ollama / TGI | 8000 / 11434 / 8080 | 按需映射   | 私有化大模型服务（可选）              |
| TEI                 | 8080                | 按需映射   | 私有化 Embedding 服务（可选）         |

---

## 项目结构

```
AgentCraft-Claw/
|
|-- backend/                             # 后端 (Spring Boot)
|   |-- pom.xml
|   |-- Dockerfile                        # 后端容器镜像（多阶段构建）
|   |-- Dockerfile.debug                  # 后端调试镜像
|   |-- docker-compose.dev.yml            # 仅基础设施（MySQL/Redis/MinIO/Milvus/etcd）
|   |-- src/main/java/com/simon/agentcraft/
|   |   |-- AgentCraftApplication.java
|   |   |-- abtest/                       # A/B 实验平台（分流路由、实验管理、事件埋点、指标聚合）
|   |   |-- agent/                        # ReAct Agent 核心（单轮对话 Chat 链路）
|   |   |-- claw/                         # Claw 统一工具抽象与注册中心
|   |   |-- common/                       # 结果封装、异常、常量、JWT
|   |   |-- config/                       # 配置中心（AI 热更新、MCP、Milvus、MinIO、Redis、RestTemplate）
|   |   |-- controller/                   # REST + SSE + MCP 控制器
|   |   |-- craft/                        # Craft Engine（OTAC 任务工坊）+ AutoPlanner + SelfEvolvingPlanner
|   |   |   |-- planner/                  # 自适应规划（PlannerMetricsCollector / PlannerEvolutionScheduler）
|   |   |-- entity/                       # 实体 + DTO + VO
|   |   |-- harness/                      # HarnessEngine（PRE_INPUT/PRE_EXECUTION/PER_STEP/POST_OUTPUT 约束治理）
|   |   |-- llm/                          # LLM 接入与韧性治理
|   |   |   |-- local/                    # 私有化模型管理（LocalModelService / LocalModelHealthIndicator）
|   |   |   |-- resilience/               # Resilience4j 包装的韧性 ChatModel
|   |   |-- loop/                         # LoopEngine（Otac/Reviewer/Budget/Reflection）
|   |   |-- mapper/                       # MyBatis Plus Mapper
|   |   |-- mcp/                          # MCP 工具（@Tool 注解暴露）
|   |   |-- memory/                       # 四级记忆（Working/Semantic/Episodic/Procedural）
|   |   |-- metrics/                      # Micrometer 指标封装
|   |   |-- multimodal/                   # 多模态 RAG（OCR / 多模态 Embedding / 视觉检索融合）
|   |   |-- openclaw/                     # OpenClaw 协议（对外暴露任务提交能力）
|   |   |   |-- interop/                  # Agent 协作协议（AgentRegistryService / AgentTaskDelegator / 平台适配器）
|   |   |-- reflection/                   # CheckpointService（人机协作检查点）
|   |   |-- safety/                       # GuardRailService（多维安检）
|   |   |-- security/                     # JWT 认证 + Spring Security + RBAC + SSO
|   |   |   |-- rbac/                     # RBAC 权限（角色继承、权限树、数据权限、@RequirePermission）
|   |   |   |-- sso/                      # SSO 单点登录（LDAP/AD / OAuth2/OIDC）
|   |   |-- service/                      # 业务服务层
|   |   |   |-- alert/                    # 告警通道 SPI 与实现
|   |   |   |-- code/                     # 代码生成/审查
|   |   |   |-- impl/                     # Service 实现
|   |   |   |-- knowledge/                # 文档处理 + Milvus + MinIO
|   |   |   |-- rag/                      # RAG 七步检索链
|   |   |-- swarm/                        # SwarmOrchestrator（多角色协作）
|   |   |-- techblog/                     # TechBlogHub 模块（抓取/翻译/摘要/标签/索引/导出）
|   |-- src/main/resources/
|   |   |-- application.yml               # 主配置（LLM/DB/Redis/Milvus/MinIO/TechBlogHub/多模态/本地模型）
|   |   |-- logback-spring.xml            # Loki 日志推送配置
|   |   |-- prompts/                      # 安全审查、查询改写、知识问答、TechBlogHub、Planner 进化提示词
|   |-- src/test/                         # 单元测试 + 集成测试
|   |   |-- java/com/simon/agentcraft/
|   |   |-- resources/application-test.yml
|   |   |-- resources/schema.sql          # H2 测试库表结构
|
|-- frontend/                             # 前端 (Vue 3)
|   |-- package.json
|   |-- vite.config.ts                    # 开发代理、代码分割
|   |-- Dockerfile                        # 前端容器镜像（Node 22 -> nginx）
|   |-- nginx.conf                        # 反向代理 /api，SSE 无缓冲，history 模式
|   |-- src/
|   |   |-- api/                          # Axios API 封装（chat/craft/knowledge/mcp/techBlog 等）
|   |   |-- components/                   # 组件（含 Liquid Glass）
|   |   |-- layout/                       # 主布局
|   |   |-- router/                       # 路由配置（含管理员守卫）
|   |   |-- stores/                       # Pinia 状态（user/theme）
|   |   |-- styles/                       # 全局样式
|   |   |-- utils/                        # 工具函数
|   |   |-- views/                        # 页面视图
|   |   |   |-- admin/                    # AI 配置、告警中心、本地模型管理、SSO 配置、RBAC 管理
|   |   |   |-- auth/                     # 登录/注册/找回密码/SSO 登录
|   |   |   |-- chat/                     # 智能问答（含多模态输入）
|   |   |   |-- command/                  # 任务指挥中心
|   |   |   |-- dashboard/                # 数据大屏
|   |   |   |-- knowledge/                # 知识库管理 + 版本管理（KbVersionView.vue）
|   |   |   |-- mcp/                      # MCP 控制台
|   |   |   |-- tech-blog/                # TechBlogHub 管理后台
|   |   |   |-- user/                     # 个人中心
|
|-- docs/
|   |-- agentcraft.sql                    # 完整数据库脚本（含 sys_ai_config、tech_blog_*、planner_evolution_log、agent_registry 等表）
|   |-- operations-runbook.md             # 生产运维 Runbook
|-- infra/                                # 可观测性与告警配置
|   |-- prometheus/
|   |   |-- prometheus.yml                # Prometheus 抓取与告警管理器配置
|   |   |-- alerts.yml                    # Prometheus 告警规则
|   |-- grafana/                          # Grafana 数据源、大盘与插件预配置
|   |   |-- provisioning/
|   |   |-- dashboards/agentcraft-claw.json
|   |-- alertmanager/
|   |   |-- alertmanager.yml              # Alertmanager 路由与 webhook 接收器
|-- docker-compose.yml                    # 全栈一键部署
|-- .env.example                          # 环境变量示例
|-- meta-mcp-config.json                  # MCP 工具配置示例
|-- README.md                             # 本文档
```

---

## 快速开始

### 环境要求

- Java 17+（本地开发）
- Node.js 20+（本地开发）
- Docker & Docker Compose（推荐，一键部署）

> 数据库、缓存、向量检索等依赖均可通过 Docker Compose 自动启动，无需单独安装。

---

### 方式一：Docker 一键部署（推荐）

仅需 Docker 即可启动完整系统，无需本地安装 Java / Node.js。

```bash
# 1. 克隆项目并进入目录
cd AgentCraft-Claw

# 2. 复制环境变量模板并填写真实值（BAILIAN_API_KEY 与 JWT_SECRET 必填）
cp .env.example .env
# 编辑 .env：BAILIAN_API_KEY、JWT_SECRET、MYSQL_ROOT_PASSWORD 等

# 3. 一键启动（首次构建需要 3-5 分钟）
docker compose up -d --build

# 4. 查看后端日志等待启动完成
docker compose logs -f backend
```

**端口映射（避免与本地服务冲突）**：

| 服务         | 容器端口 | 宿主机端口 | 说明                    |
| ------------ | -------- | ---------- | ----------------------- |
| 前端         | 80       | 80         | 主入口                  |
| 后端 API     | 8080     | 8080       | REST + SSE API          |
| MySQL        | 3306     | 3307       | 业务数据库              |
| Redis        | 6379     | 6380       | 缓存 + 会话             |
| MinIO API    | 9000     | 9002       | 对象存储 API            |
| MinIO 控制台 | 9001     | 9003       | 对象存储控制台          |
| Milvus       | 19530    | 19530      | 向量数据库              |
| Prometheus   | 9090     | 9090       | 指标抓取与查询          |
| Grafana      | 3000     | 3000       | 可观测性大盘            |
| Loki         | 3100     | 3100       | 日志接收端              |
| Alertmanager | 9093     | 9093       | Prometheus 告警通知管理 |

**访问地址**：

| 服务         | 地址                                 | 说明                                             |
| ------------ | ------------------------------------ | ------------------------------------------------ |
| 前端         | http://localhost                     | 主入口                                           |
| 后端 API     | http://localhost/api                 | REST API                                         |
| 健康检查     | http://localhost/api/actuator/health | 无需认证                                         |
| Prometheus   | http://localhost:9090                | 指标抓取与查询（建议生产环境限制访问）           |
| Grafana      | http://localhost:3000                | 可观测性大盘（默认 admin / admin，生产必须修改） |
| Loki         | http://localhost:3100                | 日志接收端（建议生产环境限制访问）               |
| Alertmanager | http://localhost:9093                | Prometheus 告警路由与静默管理                    |
| MinIO 控制台 | http://localhost:9003                | 默认账号见 .env，生产必须修改                    |

**首次部署注意事项**：

1. **MySQL 初始化**：首次启动会自动执行 `docs/agentcraft.sql` 初始化数据库，包含所有表结构和初始数据（含 `sys_ai_config` 默认配置与 TechBlogHub 厂商源）
2. **环境变量**：必须设置 `BAILIAN_API_KEY`（LLM/Embedding/Rerank）和 `JWT_SECRET`（长度 ≥ 32 字节），否则后端无法启动
3. **端口冲突**：如果本地已运行 MySQL/Redis 等，已映射到 3307/6380 等端口避免冲突
4. **Docker 网络**：基础设施运行在 `agentcraft_default` 网络，后端同时连接 `agentcraft_default` 和 `agentcraft-claw_default`，确保容器间 DNS 解析正常

```bash
# 常用命令
# 停止并清理数据（包括数据库卷）
docker compose down -v

# 仅启动基础设施（不启动前后端）
docker compose -f backend/docker-compose.dev.yml up -d

# 重建前端（修改代码后）
docker compose up -d --build frontend

# 重建后端（修改代码后）
docker compose up -d --build backend

# 查看所有服务状态
docker compose ps

# 查看特定服务日志
docker compose logs -f backend
docker compose logs -f frontend
```

---

## 部署后验证

Docker Compose 启动完成后，按以下清单确认系统健康：

### 1. 服务状态

```bash
docker compose ps
# 期望：backend / frontend / mysql / redis / minio / milvus / prometheus / grafana / loki / alertmanager / etcd 均为 healthy
```

### 2. 后端健康检查

```bash
curl http://localhost/api/actuator/health
```

### 3. Prometheus Targets

访问 http://localhost:9090/targets，确认 `agentcraft-backend` 状态为 **UP**。

### 4. Grafana 数据源

访问 http://localhost:3000（默认 admin / admin），进入 **Connections -> Data sources**，确认 Prometheus 与 Loki 测试通过。

### 5. Alertmanager 状态

```bash
curl http://localhost:9093/api/v2/status
```

### 6. 告警全链路验证

向后端 webhook 推送模拟 Alertmanager 负载：

```bash
curl -X POST http://localhost/api/internal/alerts/webhook \
  -H "Content-Type: application/json" \
  -d '{
    "version": "4",
    "status": "firing",
    "alerts": [{
      "status": "firing",
      "labels": {"alertname": "AgentCraftTestAlert", "severity": "warning"},
      "annotations": {"summary": "测试告警", "description": "用于验证告警通道"},
      "startsAt": "2024-01-01T00:00:00Z"
    }]
  }'
```

管理员登录前端后访问 `/alerts`，应能看到刚才推送的测试告警。

### 7. 真实通知通道验证

配置 `.env` 中的钉钉/飞书/邮件等凭据后，重新加载 Alertmanager，再次推送 `severity: critical` 的测试告警，确认各通道收到通知。

> 完整运维 Runbook 见 `docs/operations-runbook.md`。

---

### 方式二：本地开发

#### 1. 启动基础设施

```bash
cd backend
docker compose -f docker-compose.dev.yml up -d
```

#### 2. 初始化数据库

```bash
mysql -u root -p -h localhost -P 3307 < ../docs/agentcraft.sql
```

#### 3. 配置后端环境变量

```bash
export BAILIAN_API_KEY=sk-your-api-key
export JWT_SECRET=$(openssl rand -base64 32)
export MYSQL_ROOT_PASSWORD=root
# 可选
export TAVILY_API_KEY=tvly-your-key
export CORS_ALLOWED_ORIGINS=http://localhost:5173
```

> 也可以将上述变量写入 `backend/.env` 后由 IDE 自动加载。

#### 4. 启动后端

```bash
cd backend
mvn clean spring-boot:run
# 或
mvn clean package -DskipTests
java -jar target/AgentCraft-Claw-0.0.1-SNAPSHOT.jar
```

#### 5. 启动前端

```bash
cd frontend
npm install
npm run dev
# 默认访问 http://localhost:5173
```

---

## 环境变量说明

复制 `.env.example` 为 `.env` 并按需填写。生产环境必须修改所有默认密码。

| 变量                                              | 必填 | 说明                                                         |
| ------------------------------------------------- | ---- | ------------------------------------------------------------ |
| `BAILIAN_API_KEY`                                 | 是   | 阿里云百炼 API Key，用于 LLM / Embedding / Rerank            |
| `JWT_SECRET`                                      | 是   | JWT 签名密钥，长度 ≥ 32 字节                                 |
| `MYSQL_ROOT_PASSWORD`                             | 建议 | MySQL root 密码                                              |
| `MINIO_ROOT_PASSWORD`                             | 建议 | MinIO root 密码                                              |
| `GRAFANA_ADMIN_USER` / `GRAFANA_ADMIN_PASSWORD`   | 建议 | Grafana 管理员账号，生产必须修改                             |
| `REDIS_PASSWORD`                                  | 否   | Redis 密码（留空表示无密码）                                 |
| `CORS_ALLOWED_ORIGINS`                            | 否   | 前端访问域名，多个用英文逗号分隔                             |
| `OPENCLAW_API_KEY`                                | 否   | OpenClaw 外部任务接口 API Key，未配置时接口返回 401          |
| `DOCMIND_WEB_SEARCH_ENABLED`                      | 否   | 联网搜索开关，默认 `false`                                   |
| `TAVILY_API_KEY`                                  | 否   | Tavily API Key，开启联网搜索时必填                           |
| `MINIO_ROOT_USER`                                 | 否   | MinIO root 用户名，默认 `minioadmin`                         |
| `MINIO_ROOT_PASSWORD`                             | 建议 | MinIO root 密码                                              |
| `ALERT_EMAIL_ENABLED`                             | 否   | 邮件告警开关，默认 `false`                                   |
| `DINGTALK_ALERT_ENABLED`                          | 否   | 钉钉告警开关，默认 `false`                                   |
| `WECHAT_WORK_ALERT_ENABLED`                       | 否   | 企业微信告警开关，默认 `false`                               |
| `FEISHU_ALERT_ENABLED`                            | 否   | 飞书告警开关，默认 `false`                                   |
| `SMS_ALERT_ENABLED`                               | 否   | 短信告警开关，默认 `false`                                   |
| `TECH_BLOG_MINIO_BUCKET`                          | 否   | TechBlogHub 离线站点 MinIO bucket，默认 `tech-blog-sites`    |
| `TECH_BLOG_REQUEST_INTERVAL_MS`                   | 否   | 博客抓取请求间隔（反爬），默认 `3000`                        |
| `TECH_BLOG_MAX_RETRIES`                           | 否   | 博客抓取最大重试次数，默认 `3`                               |
| `TECH_BLOG_PLAYWRIGHT_ENABLED`                    | 否   | 是否启用 Playwright 动态渲染，默认 `false`                   |
| `TECH_BLOG_REVIEWER_THRESHOLD`                    | 否   | TechBlogHub Reviewer 评分阈值，默认 `0.85`                   |
| `APP_LOG_LEVEL`                                   | 否   | 应用日志级别，默认 `INFO`                                    |
| `SECURITY_LOG_LEVEL`                              | 否   | Spring Security 日志级别，默认 `INFO`                        |
| `LOKI_URL`                                        | 否   | Loki 日志推送地址，默认 `http://localhost:3100/loki/api/v1/push` |
| `ALERT_EMAIL_*`                                   | 否   | 邮件告警 SMTP 配置                                           |
| `DINGTALK_WEBHOOK_URL`                            | 否   | 钉钉群机器人 Webhook                                         |
| `WECHAT_WORK_WEBHOOK_URL`                         | 否   | 企业微信群机器人 Webhook                                     |
| `FEISHU_WEBHOOK_URL`                              | 否   | 飞书群机器人 Webhook                                         |
| `SMS_WEBHOOK_URL` / `SMS_API_KEY` / `SMS_TO_LIST` | 否   | 短信网关配置                                                 |
| `MULTIMODAL_OCR_ENGINE`                           | 否   | 多模态 OCR 引擎选择：`tesseract` / `ollama`，默认 `tesseract` |
| `MULTIMODAL_EMBEDDING_URL`                        | 否   | 多模态 Embedding 服务地址（Ollama 或兼容服务）               |
| `LOCAL_MODEL_BASE_URL`                            | 否   | 本地模型 OpenAI-compatible API base URL                      |
| `TEI_BASE_URL`                                    | 否   | TEI（Text Embeddings Inference）服务地址                     |
| `SSO_ENABLED`                                     | 否   | SSO 单点登录开关，默认 `false`                               |
| `SSO_PROVIDER_TYPE`                               | 否   | SSO 提供商类型：`ldap` / `oauth2` / `oidc`，默认 `oauth2`    |
| `LDAP_URL`                                        | 否   | LDAP 服务器地址，如 `ldap://localhost:389`                   |
| `LDAP_BASE_DN`                                    | 否   | LDAP 基础 DN，如 `dc=example,dc=com`                         |
| `LDAP_BIND_DN`                                    | 否   | LDAP 绑定账号 DN                                             |
| `LDAP_BIND_CREDENTIAL`                            | 否   | LDAP 绑定账号密码                                            |
| `OAUTH2_CLIENT_ID`                                | 否   | OAuth2 Client ID                                             |
| `OAUTH2_CLIENT_SECRET`                            | 否   | OAuth2 Client Secret                                         |
| `OAUTH2_AUTHORIZATION_URI`                        | 否   | OAuth2 授权端点                                              |
| `OAUTH2_TOKEN_URI`                                | 否   | OAuth2 令牌端点                                              |
| `OAUTH2_USER_INFO_URI`                            | 否   | OAuth2 用户信息端点                                          |
| `OIDC_ISSUER_URI`                                 | 否   | OIDC Issuer URI                                              |
| `AGENT_INTEROP_CALLBACK_SECRET`                   | 否   | Agent 协作回调 HMAC 签名密钥                                 |

---

## 核心配置

### AI 参数配置（动态热更新）

系统支持通过 `sys_ai_config` 表动态调整参数，无需重启服务。启动时由 `AiConfigInitializer` 加载到 `AiConfigHolder` 内存快照，配置变更后原子替换 `ChatModel` 实例，正在进行的 SSE 请求持有旧引用自然完成，新请求自动使用新配置。

| 参数                            | 默认值                                           | 说明                                               |
| ------------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| `llm.api_key`                   | —                                                | LLM API Key（覆盖环境变量）                        |
| `llm.base_url`                  | `https://dashscope.aliyuncs.com/compatible-mode` | LLM 基础 URL                                       |
| `llm.model`                     | `qwen-plus`                                      | 对话模型                                           |
| `llm.chat_temperature`          | `0.7`                                            | 对话温度                                           |
| `llm.timeout_seconds`           | `60`                                             | 请求超时（秒）                                     |
| `llm.provider_type`             | `CLOUD`                                          | 模型提供商：CLOUD / VLLM / OLLAMA / LOCAL          |
| `rag.vector_top_k`              | `20`                                             | 向量检索 Top-K                                     |
| `rag.bm25_top_k`                | `20`                                             | BM25 检索 Top-K                                    |
| `rag.rrf_top_n`                 | `25`                                             | RRF 融合后 Top-N                                   |
| `rag.rerank_top_k`              | `5`                                              | 重排序后 Top-K                                     |
| `reranker.remote.enabled`       | `true`                                           | 是否启用 DashScope 远程重排序                      |
| `reranker.model`                | `gte-rerank`                                     | 远程重排序模型                                     |
| `reranker.local.enabled`        | `true`                                           | 远程失败时是否启用本地语义重排序 fallback          |
| `reranker.local.keyword_weight` | `0.35`                                           | 本地重排序中关键词覆盖分的权重（其余为语义相似度） |
| `web_search.enabled`            | `false`                                          | 联网搜索开关                                       |
| `web_search.tavily_api_key`     | —                                                | Tavily API Key                                     |
| `cache.freq_threshold`          | `3`                                              | 缓存频率阈值                                       |
| `safety.confidence_threshold`   | `0.3`                                            | 安全置信度阈值                                     |
| `guardrail.semantic.enabled`    | `true`                                           | 是否开启语义层护栏                                 |
| `guardrail.semantic.threshold`  | `0.82`                                           | 语义相似度拦截阈值（0~1）                          |
| `async.document.core-pool-size` | `4`                                              | 文档处理异步线程池核心线程数                       |
| `async.document.max-pool-size`  | `10`                                             | 文档处理异步线程池最大线程数                       |
| `async.document.queue-capacity` | `100`                                            | 文档处理异步线程池队列容量                         |
| `tech_blog.request_interval_ms` | `3000`                                           | TechBlogHub 抓取请求间隔                           |
| `tech_blog.max_retries`         | `3`                                              | TechBlogHub 抓取最大重试次数                       |
| `tech_blog.playwright.enabled`  | `false`                                          | 是否启用 Playwright 动态渲染                       |
| `tech_blog.reviewer.threshold`  | `0.85`                                           | TechBlogHub Reviewer 评分阈值                      |
| `multimodal.ocr_engine`         | `tesseract`                                      | 多模态 OCR 引擎：tesseract / ollama                |
| `multimodal.embedding_url`      | —                                                | 多模态 Embedding 服务地址                          |
| `local_model.base_url`          | —                                                | 本地模型 OpenAI-compatible API 地址                |
| `local_model.model`             | —                                                | 本地模型名称                                       |
| `tei.base_url`                  | —                                                | TEI 私有化 Embedding 服务地址                      |
| `planner.evolution.enabled`     | `true`                                           | 自适应规划进化开关                                 |
| `planner.evolution.threshold`   | `0.7`                                            | 自动进化触发成功率阈值                             |
| `planner.evolution.cron`        | `0 17 3 * * ?`                                   | 定时进化 Cron 表达式（默认每天 3:17）              |
| `agent.interop.enabled`         | `true`                                           | Agent 协作协议开关                                 |
| `agent.interop.callback_secret` | —                                                | 回调 HMAC 签名密钥                                 |

---

## RAG 七步检索链

```
用户问题
    |
    v
[1] QueryRewriter -- 查询改写（扩展、去歧义、意图增强）
    |
    v
[2] VectorRetriever -- Milvus 语义检索
    |
    v
[3] BM25Retriever -- Lucene 关键词检索（smartcn 分词）
    |
    v
[4] RRFFusion -- 互惠排名融合（RRF K=60）
    |
    v
[5] CrossEncoderReranker -- gte-rerank 交叉编码器重排序，远程失败时自动降级到 LocalReranker（本地 Embedding 语义重排序 + 关键词覆盖）
    |
    v
[6] ContextCompressor -- 上下文压缩（按 Token 预算截断）
    |
    v
[7] PromptAssembler -- 组装最终 Prompt（上下文 + 历史 + 记忆 + 系统指令）
    |
    v
LLM 生成
    |
    v
SelfReflection -- 自检（幻觉、相关性、完整性，最多 2 轮）
    |
    v
来源标注（文档名、章节、页码、相关性分数）
```

### 智能文档切片

文档切片质量直接决定 RAG 召回上限。系统不再使用固定长度切片，而是引入结构感知切片引擎：

- **结构化解析**：`StructuredDocumentParser` 识别标题、表格、代码块、FAQ 问答对、列表、定义/术语等语义块
- **差异化策略**：
  - 标题单独成块，建立章节索引
  - 表格 / 代码块 / FAQ 保持完整，避免破坏行关系或语法结构
  - 普通段落按目标大小合并，超长按句子切分并保留重叠窗口
- **动态参数**：切片大小与重叠长度从 `AiConfigHolder` 实时读取（`rag.chunk_size`、`rag.chunk_overlap`），支持 AI 配置中心热更新
- **元数据增强**：每个切片自动携带 `contentType`（heading/table/code/faq/list/definition/general）、`chapter`、`pageNumber`，用于 BM25 过滤与答案溯源

### 答案句子级溯源

为提升答案可信度，系统在生成后对每个答案句子与检索片段建立映射：

- 后端 `ProvenanceService` 按中英文句末标点拆分答案，计算句子与 chunk 的 token 覆盖度，返回 `sentence → references` 列表
- 每条引用包含 `chunkId`、`source`（knowledge_base/web）、`documentName`、`score`、`snippet`
- SSE `done` 事件与 `qa_message.provenance` 字段均携带完整溯源 JSON
- 前端 Chat 页面新增「答案溯源」面板：
  - 列出所有被引用的句子
  - 每个句子展示 Top-3 来源徽章（含相似度）
  - 点击来源徽章自动滚动到对应参考来源卡片并高亮
  - 来源卡片新增 `chunkId` 锚点，支持双向定位

---

## 知识库版本与增量更新

为支持大文档反复迭代与回滚，知识库引入版本化设计：

- **版本记录**：每次上传生成一条 `KbDocumentVersion`，记录 `version_no`、`file_hash`、`file_size`、`file_url`、`status`、`change_summary` 与操作人；`kb_knowledge_base.current_version_id` 指向当前生效版本
- **增量 diff**：`DocumentProcessTask` 按 `versionId` 处理，提取新切片后与旧版本切片按内容哈希比对，仅对新增/修改/删除的切片执行 Milvus 写入或删除，保留未变更切片
- **细粒度向量删除**：`MilvusService.insertVectors` 返回生成的 `vector_id` 列表并回填到 `kb_chunk.vector_id`；`deleteByVectorIds` 支持按 ID 列表精确删除，避免全库清空
- **失败重试与重处理**：上传或处理失败时可通过 `/api/v2/kb/{id}/reprocess` 重新创建版本并异步处理；删除知识库时同步清理 DB、MinIO 对象与 Milvus 向量

### 前端版本管理

前端提供完整的版本管理页面（`KbVersionView.vue`），管理员可：

- 查看版本列表（版本号、文件大小、变更摘要、状态、操作人、时间）
- 发布新版本、回滚到历史版本
- **行级 Diff 对比**：基于 LCS（最长公共子序列）算法，对相邻版本切片进行行级比对，增删改以不同颜色高亮显示，直观呈现变更内容
- UI 采用 iOS 26 Liquid Glass 玻璃材质设计，与系统整体风格一致

---

## RAG 评估平台

为量化优化效果，系统内置 RAG Evaluation Platform：

- **评测集管理**：管理员可创建评测集并批量导入标准问题（含期望答案、期望 chunk IDs、相关 kbIds）
- **自动运行**：对评测集中每个问题执行完整 RAG 链路（改写 → 向量/BM25 → RRF → 重排 → 生成）
- **核心指标**：
  - `Recall@5`：前 5 个检索结果命中期望 chunk 的比例
  - `MRR`：首个相关 chunk 的平均倒数排名
  - `Context Precision`：检索结果的相关密度
  - `Answer Relevance`：LLM 评委对生成答案相关性的 0-5 评分
  - `Faithfulness`：LLM 评委对答案忠实度的 0-5 评分
- **对比与导出**：支持查看历史运行结果、导出 CSV/JSON 报告

管理员入口：`/rag-eval`

---

## 统一审计中心

所有问答行为统一落入审计中心，满足企业合规需求：

- **查询维度**：按时间范围、用户 ID、会话 ID、是否兜底、用户反馈、关键词检索 `qa_message`
- **脱敏导出**：支持导出 CSV/JSON，自动对手机号、邮箱等敏感信息进行 `***` 掩码
- **字段覆盖**：消息内容、来源数量、是否兜底、反馈、响应时间、Agent 推理链、MCP 调用、反思日志

管理员入口：`/audit`

---

## Trace 可视化

为让用户和管理员理解 "AI 为什么这样回答"，系统提供消息级 Trace 追踪：

- `GET /api/v2/chat/trace/{messageId}` 聚合 `qa_message` 中的 `agentTrace`、`mcpCalls`、`reflectionLog`、`provenance`、`sources`
- `GET /api/craft/{craftId}/trace` 查询任务工坊执行记录
- 前端 Trace 页面以时间线方式展示：
  - Agent 思考 → 行动 → 观察
  - MCP 工具调用与耗时
  - Reviewer / Reflection 审查结果
  - 答案溯源句子卡片

入口：`/trace`

---

## API 概览

### 智能问答

```http
GET /api/v2/chat/stream?message=What%20is%20RAG&conversationId=&kbIds=1,2
Accept: text/event-stream
Cookie: access_token={jwt}
```

> SSE 接口依赖 HttpOnly Cookie 中的 `access_token` 或 `Authorization: Bearer` Header 进行 JWT 认证。EventSource 会自动携带同域 Cookie。

SSE 响应事件：

```
event: start
event: intent         # 意图识别结果
event: rewrite        # 查询改写结果
event: retrieval      # 检索结果摘要
event: rerank         # 重排序结果
event: token          # 流式生成 Token
event: reflection     # 自检结果
event: done
```

### 多模态问答

```http
POST /api/v2/multimodal/chat
Content-Type: application/json
Cookie: access_token={jwt}

{
  "message": "这张图讲了什么？",
  "image": "data:image/png;base64,iVBORw0KGgo...",
  "conversationId": "",
  "kbIds": [1, 2]
}
```

> 支持 `image`（Base64）或 `imageUrl`（公开 URL）两种传图方式；SSE 流式输出，事件类型与 `/api/v2/chat/stream` 一致。

### 任务工坊

```http
GET /api/craft/stream?goal=Generate%20AI%20Research%20Report&kbIds=1,2
Accept: text/event-stream
Cookie: access_token={jwt}
```

SSE 响应：

```
event: plan           # 执行计划
event: step           # 步骤状态更新
event: agent          # Agent 状态
event: message        # 中间消息
event: checkpoint     # 检查点（暂停等待人类输入）
event: result         # 最终结果
```

恢复执行：

```http
POST /api/craft/resume/{craftId}
Content-Type: application/json

{ "response": "Confirm to continue" }
```

### MCP 工具链

```http
GET /api/mcp/tools        # 工具列表
GET /api/mcp/tools/{id}   # 工具详情
GET /api/mcp/stats        # 调用统计
POST /api/mcp/tools/{id}/test   # 工具测试
PUT /api/mcp/tools/{id}/status  # 启停工具
```

### OpenClaw 外部任务

```http
POST /api/openclaw/tasks
Content-Type: application/json
X-API-Key: {openclaw_api_key}

{
  "callerId": "external-agent-001",
  "correlationId": "req-20240627-001",
  "goal": "生成一份 AI 行业研究报告",
  "kbIds": [1, 2],
  "constraints": {"maxRounds": 5},
  "callbackUrl": "https://example.com/callback"
}
```

> OpenClaw 通过 `X-API-Key` Header 鉴权，API Key 由环境变量 `OPENCLAW_API_KEY` 配置；`callbackUrl` 会在任务关键事件发生时收到 HTTP POST 回调。

### Agent 协作协议

```http
# 委托任务到外部 Agent 平台
POST /api/openclaw/agents/delegate
Content-Type: application/json
X-API-Key: {openclaw_api_key}

{
  "targetPlatform": "crewai",
  "capabilityTags": ["research", "analysis"],
  "task": {
    "goal": "分析竞品定价策略",
    "context": "目标公司为某 SaaS 企业",
    "maxSteps": 5
  },
  "callbackUrl": "https://example.com/callback",
  "timeoutSeconds": 300
}

# 接收外部 Agent 回调结果
POST /api/openclaw/agents/callback
Content-Type: application/json
X-Interop-Signature: {hmac_sha256_signature}

{
  "delegationId": "dlg-001",
  "status": "completed",
  "result": { "summary": "..." }
}
```

### 语音识别

```websocket
wss://host/api/speech/ws
// 二进制 PCM 音频流 -> 实时文本
```

### Tech Blog Hub

```http
# 触发博客抓取任务
POST /api/v2/tech-blog/crawl
{
  "query": "帮我抓 Anthropic 近期 10 篇",
  "sources": ["anthropic"],
  "maxArticles": 10,
  "autoTranslate": true
}

# SSE 进度流
GET /api/v2/tech-blog/crawl/{taskId}/stream
Accept: text/event-stream

# 文章列表 / 搜索 / 筛选
GET /api/v2/tech-blog/articles?sourceId=1&tags=Agent&tags=RAG&sortField=publishDate&sortOrder=desc&page=1&size=20

# 生成离线站点
POST /api/v2/tech-blog/export
{
  "name": "Anthropic-Agent-专题",
  "filters": { "sources": ["anthropic"], "tags": ["Agent"] }
}

# 下载离线站点 ZIP
GET /api/v2/tech-blog/export/{exportId}/download

# 标签统计
GET /api/v2/tech-blog/stats/tags
```

### 数据大屏

```http
GET /api/v2/stats/overview?timeRange=week
```

### RAG 评测平台（管理员）

```http
# 数据集管理
GET    /api/admin/rag-eval/datasets
POST   /api/admin/rag-eval/datasets
GET    /api/admin/rag-eval/datasets/{datasetId}
POST   /api/admin/rag-eval/datasets/{datasetId}/questions

# 评测运行
GET    /api/admin/rag-eval/runs
POST   /api/admin/rag-eval/runs
GET    /api/admin/rag-eval/runs/{runId}
GET    /api/admin/rag-eval/runs/{runId}/results?page=1&size=20
GET    /api/admin/rag-eval/runs/{runId}/export?format=csv
```

- 启动运行后由独立线程池异步执行，状态会实时推进 `running -> completed/failed`
- 指标：Recall@1/3/5、MRR、Context Precision、Answer Relevance（0-5）、Faithfulness（0-5）、平均延迟
- 评测过程直接复用现有 RAG 检索组件，不经过 SSE 与对话持久化，无副作用

### 统一审计中心（管理员）

```http
GET  /api/admin/audit/messages?userId=&conversationId=&feedback=&fallbackOnly=&keyword=&page=1&size=20
POST /api/admin/audit/export
{
  "format": "csv",
  "filters": { "userId": 1, "feedback": 1 }
}
```

- 查询条件支持时间范围、用户 ID、会话 ID、反馈状态、仅兜底、内容关键字
- 导出内容自动对手机号与邮箱脱敏

### Trace 追踪

```http
GET /api/v2/chat/trace/{messageId}     # 消息级：Agent Trace / MCP Calls / Reflection / Provenance
GET /api/craft/{craftId}/trace         # Craft 级：执行轨迹 / Agent / Tools / 耗时
```

- 消息 Trace 校验当前用户是否为会话所有者
- Craft Trace 校验当前用户是否为任务所有者

### A/B 实验平台（管理员）

```http
# 实验列表 / 创建 / 详情 / 更新 / 删除
GET    /api/admin/ab-experiments
POST   /api/admin/ab-experiments
GET    /api/admin/ab-experiments/{id}
PUT    /api/admin/ab-experiments/{id}
DELETE /api/admin/ab-experiments/{id}

# 实验生命周期
POST /api/admin/ab-experiments/{id}/start
POST /api/admin/ab-experiments/{id}/pause
POST /api/admin/ab-experiments/{id}/complete

# 实验指标
GET /api/admin/ab-experiments/{id}/metrics
```

- 仅 `ADMIN` 角色可访问；实验支持 `CHAT_MODEL` / `TEMPERATURE` 等类型，可按流量百分比与 variant 权重分流
- `ExperimentRouter` 使用稳定哈希（MurmurHash3 简化实现）确保同一用户多次请求落在同一 variant
- `AiConfigHolder` 为每个有效 variant 组合缓存独立 `ChatModel`，配置变更时自动失效重载
- 实验事件通过 `ab_experiment_event` 记录，metrics 接口统计曝光/命中/转化

管理员入口：`/ab-experiments`

---

### 告警中心（管理员）

```http
# 查看最近告警
GET /api/admin/alerts/recent?limit=20

# 确认（移除）单条告警
POST /api/admin/alerts/{fingerprint}/acknowledge

# Alertmanager webhook（内部调用，无需鉴权）
POST /api/internal/alerts/webhook
```

---

### 本地模型管理（管理员）

```http
# 本地模型 CRUD
GET    /api/admin/local-models
POST   /api/admin/local-models
GET    /api/admin/local-models/{id}
PUT    /api/admin/local-models/{id}
DELETE /api/admin/local-models/{id}

# 连通性测试
POST /api/admin/local-models/{id}/test

# 健康状态
GET /api/admin/local-models/{id}/health
```

- 支持 vLLM / Ollama / TGI 三种本地模型提供商配置
- 连通性测试会发送一条探测请求，验证模型是否可用
- 健康状态由 `LocalModelHealthIndicator` 周期性检测

### RBAC 权限管理（管理员）

```http
# 角色管理
GET    /api/admin/rbac/roles
POST   /api/admin/rbac/roles
PUT    /api/admin/rbac/roles/{id}
DELETE /api/admin/rbac/roles/{id}

# 权限树
GET /api/admin/rbac/permissions/tree

# 用户角色分配
POST /api/admin/rbac/users/{userId}/roles
GET  /api/admin/rbac/users/{userId}/roles

# 数据权限配置
PUT /api/admin/rbac/roles/{roleId}/data-scope
```

- 角色支持层级继承，删除父角色时自动校验子角色引用
- 权限树支持菜单 / 按钮 / API / 数据四级粒度
- 数据权限范围：ALL / OWN / DEPT / TEAM

### SSO 配置管理（管理员）

```http
# 获取当前 SSO 配置
GET /api/admin/sso/config

# 更新 SSO 配置
PUT /api/admin/sso/config

# 连通性测试
POST /api/admin/sso/test
```

- 支持 LDAP/AD 与 OAuth2/OIDC 两种模式配置
- 连通性测试会实际发起认证请求，验证配置正确性

---

## 安全特性

- **JWT 无状态认证**：24h 访问令牌 + 7d 刷新令牌
- **BCrypt 密码哈希**：强度 10 的 bcrypt 加密
- **角色权限控制**：`ROLE_ADMIN` / `ROLE_USER`，管理员路由前端守卫 + 后端 `@PreAuthorize` / `@RequirePermission` 双重校验
- **资源级权限**：RBAC 角色层级继承、权限树、数据权限（ALL / OWN / DEPT / TEAM），注解驱动 AOP 拦截
- **SSO 安全**：LDAP/AD 与 OAuth2/OIDC 单点登录，SSO 登录后自动用户同步，不暴露企业内部密码
- **输入安全审查**：GuardRailService 规则 + 语义双层安检（越狱、注入、敏感信息、暴力、仇恨、非法、隐私），支持 Embedding 向量相似度拦截同义/改写绕过
- **输出安全审查**：规则 + 语义双层有害内容拦截，事实核查、幻觉检测、置信度阈值
- **SSE 安全认证**：
  - 一次性票据（`ticket`）机制，60 秒 TTL，仅可使用一次
  - 同域场景下自动携带 HttpOnly Cookie
  - 彻底避免将长期 JWT 暴露在 URL 查询参数中
- **CORS 跨域配置**：通过 `CORS_ALLOWED_ORIGINS` 环境变量配置白名单，生产环境禁止 `*` + `AllowCredentials` 组合
- **JWT 密钥校验**：启动时校验 `JWT_SECRET` 长度 ≥ 32 字节，防止使用弱密钥
- **Actuator 安全**：`/actuator/health/**`、`/actuator/info` 与 `/actuator/prometheus` 对外放行，便于 Prometheus 抓取；生产环境建议通过反向代理或 IP 白名单限制 `/actuator/prometheus`
- **XSS 防护**：前端 Markdown 渲染使用 DOMPurify 过滤，防止 `v-html` 注入
- **TechBlogHub SSRF 防护**：`UrlWhitelistHarness` 仅允许预定义域名列表（`anthropic.com`、`openai.com`、`deepmind.google`、`ai.meta.com` 等），并通过 HarnessEngine PRE_EXECUTION 拦截非法 URL；`BlogDiscoveryTool` 支持 RSS / Sitemap / 官方 API 与 Tavily 降级发现
- **WebBridge URL 白名单**：浏览器自动化仅允许公开技术域名（`anthropic.com`、`openai.com`、`deepmind.google`、`ai.meta.com`、`wikipedia.org`、`github.com`、`stackoverflow.com`、`docs.spring.io` 等），非法 URL 在 PRE_EXECUTION 阶段被 HarnessEngine 拦截，防止 SSRF 与恶意跳转
- **Agent 协作回调安全**：跨平台回调使用 HMAC-SHA256 签名（`X-Interop-Signature`），接收端校验签名防伪造
- **多模态输入安全**：上传图片/PDF 经过文件类型校验、大小限制、内容扫描，防止恶意文件上传

---

## 前端设计系统

### iOS 26 Liquid Glass 设计语言

前端采用苹果 iOS 26 的 Liquid Glass 设计哲学：

- **液态玻璃材质**：`backdrop-filter: blur(30-60px) saturate(160-200%)` + 多层折射高光
- **动态流体背景**：4 个彩色渐变球 + CSS 漂移动画 + 噪点纹理
- **胶囊按钮**：`border-radius: 9999px` + 渐变填充 + 光晕阴影
- **玻璃卡片**：圆角 24px + 顶部高光层 + 悬停上浮效果
- **色彩系统**：iOS 原生色彩（#0A84FF 蓝、#BF5AF2 紫、#30D158 绿、#FF453A 红）
- **排版**：SF Pro Display + 更大字号、更轻重标体
- **动画**：Spring 缓动曲线 + 入场模糊淡出 + 呼吸光晕

---

## 测试

### 运行方式

```bash
# 后端单元测试 + 集成测试
cd backend
mvn test

# 仅运行集成测试
mvn test -Dtest='*IntegrationTest'

# 后端全量打包（含测试）
mvn clean package

# 前端类型检查
cd frontend
npm run type-check
```

### 测试覆盖

| 类型         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| 单元测试     | `AlertNotificationServiceTest`、`DocumentProcessTaskTest`、`CrossEncoderRerankerTest`、`LocalRerankerTest`、`ProvenanceServiceTest`、`MetricsServiceTest`、`GuardRailServiceTest`、`AlertMessageFormatterTest`、`DingTalkAlertChannelTest`、`StatsServiceImplTest`、`ResilientChatModelTest`、`LoopEngineTest`、`ReviewerLoopTest`、`HarnessEngineTest`、`InputHarnessTest`、`UrlWhitelistHarnessTest`、`TranslationToolTest`、`TaggingToolTest`、`JwtUtilsTest`（JWT 生成/过期/异常解析）、`OpenClawControllerTest`（API Key 鉴权）、`OpenClawGatewayTest`（真实 HTTP 回调）、`CraftControllerTest`（异步任务/恢复/反馈持久化）、`PlannerEvolutionSchedulerTest`（定时进化与阈值触发）、`PlannerMetricsCollectorTest`（指标收集与矩阵生成）、`LocalModelHealthIndicatorTest`（健康检查与状态变更）、`AgentRegistryServiceTest`（注册/发现/心跳）、`AgentTaskDelegatorTest`（同步委托与异步回调）、`RbacServiceTest`（角色继承与权限树）、`SsoConfigServiceTest`（SSO 配置与连通性） |
| 集成测试     | `JwtAuthenticationIntegrationTest`（JWT 签发/鉴权/过期）、`ControllerIntegrationTest`（公开端点/管理员权限/参数校验）、`RagPipelineIntegrationTest`（RAG 七步链路调用顺序）、`TechBlogControllerIntegrationTest`（TechBlogHub 端到端，含 tag 过滤与排序）、`StaticSiteGeneratorToolTest`（离线站点 ZIP 结构与搜索索引）、`AbExperimentControllerIntegrationTest`（A/B 实验 CRUD 与生命周期）、`AuditControllerIntegrationTest`（审计查询与导出）、`MultimodalChatIntegrationTest`（图片上传/OCR/视觉检索融合）、`LocalModelControllerIntegrationTest`（本地模型 CRUD/连通性测试）、`AgentInteropControllerIntegrationTest`（委托/回调/HMAC 签名） |
| 组件测试     | `DagSchedulerTest`（DAG 拓扑分层与环检测）、`SwarmOrchestratorTest`（并行执行/记忆召回/Reviewer） |
| 知识库测试   | `StructuredDocumentParserTest`（列表/标题/FAQ 语义块解析）、`TextChunkerTest`（章节元数据与短段落保留）、`DocumentProcessTaskTest`（增量处理 / 失败重试 / ready 状态跳过） |
| 版本管理测试 | `KbVersionViewTest`（版本列表/发布/回滚/Diff 高亮）          |

> 集成测试使用 `test` profile + H2 内存库 + `@MockitoBean`（Spring Boot 3.4 已弃用 `@MockBean`）模拟外部依赖，无需 Docker 即可运行。

---

## 可观测性

部署后自动包含 Prometheus + Grafana + Loki + Alertmanager 栈：

| 服务         | 地址                  | 说明                                            |
| ------------ | --------------------- | ----------------------------------------------- |
| Prometheus   | http://localhost:9090 | 抓取后端 `/actuator/prometheus`                 |
| Grafana      | http://localhost:3000 | 默认 admin / admin，已预置 AgentCraft-Claw 大盘 |
| Loki         | http://localhost:3100 | 接收后端推送的日志                              |
| Alertmanager | http://localhost:9093 | Prometheus 告警路由，默认推送到后端 webhook     |

### 预置指标

后端通过 Micrometer 埋点，关键指标包括：

- `rag.retrieval.duration` — RAG 检索耗时（按 `mode` 标签区分）
- `llm.generation.duration` — LLM 完整生成耗时
- `llm.first_token.latency` — LLM 首 token 延迟
- `rag.reflection.total` — 自反思审查次数（按 `round`、`passed` 标签）
- `document.process.duration` — 文档处理耗时（按 `status` 标签）
- `craft.step.duration` / `craft.execution.duration` — Craft 任务执行耗时
- `harness.decision.total` — Harness 决策次数（按 `level`、`decision` 标签）
- `craft.loop.round` — Loop 迭代轮次
- `techblog.crawl.duration` / `techblog.article.count` / `techblog.review.score` — TechBlogHub 指标
- `multimodal.ocr.duration` / `multimodal.vision.retrieve.duration` — 多模态 RAG 指标
- `local_model.health.status` — 本地模型健康状态（0=DOWN, 1=UP）
- `planner.evolution.duration` / `planner.evolution.success_rate` — 自适应规划指标
- `agent.interop.delegate.duration` / `agent.interop.callback.total` — Agent 协作指标

### 日志聚合

后端通过 `loki-logback-appender` 将日志推送到 Loki：

- 默认推送地址：`http://localhost:3100/loki/api/v1/push`
- Docker Compose 中自动设置为 `http://loki:3100`
- 可通过环境变量 `LOKI_URL` 覆盖
- `test` profile 下关闭 Loki 推送，避免单元测试产生网络请求

Grafana 中已预置 Loki 数据源，可直接在 Explore 中按 `app=agentcraft-claw`、`level`、`thread` 等标签检索日志。

### 告警规则

Prometheus 已加载 `infra/prometheus/alerts.yml`，预置以下告警：

| 告警名                                  | 触发条件                                                     | 级别     |
| --------------------------------------- | ------------------------------------------------------------ | -------- |
| `AgentCraftHighRetrievalLatency`        | 过去 5 分钟 RAG 平均检索耗时 > 2s，持续 2 分钟               | warning  |
| `AgentCraftHighFirstTokenLatency`       | 过去 5 分钟 LLM 首 token 平均延迟 > 5s，持续 2 分钟          | warning  |
| `AgentCraftHighLlmGenerationLatency`    | 过去 5 分钟 LLM 完整生成平均耗时 > 30s，持续 2 分钟          | warning  |
| `AgentCraftDocumentProcessingFailing`   | 最近 1 分钟文档处理失败率 > 0                                | critical |
| `AgentCraftReflectionFailing`           | 过去 5 分钟自反思不通过速率 > 0.1 次/秒，持续 2 分钟         | warning  |
| `AgentCraftHighJvmHeapUsage`            | JVM 堆内存使用率 > 85%，持续 2 分钟                          | critical |
| `AgentCraftTechBlogCrawlFailing`        | 过去 5 分钟 TechBlogHub 抓取失败率 > 0，持续 2 分钟          | warning  |
| `AgentCraftTechBlogExportSlow`          | 过去 5 分钟 TechBlogHub 站点生成平均耗时 > 120s，持续 2 分钟 | warning  |
| `AgentCraftLocalModelDown`              | 本地模型健康检查连续 3 次失败                                | critical |
| `AgentCraftPlannerEvolutionFailing`     | 自适应规划进化任务连续 2 次失败                              | warning  |
| `AgentCraftAgentInteropCallbackFailing` | Agent 协作回调失败率 > 0.1，持续 5 分钟                      | warning  |

### 告警通知配置

系统已集成 **Alertmanager**，Prometheus 触发告警后会按路由分发到后端 webhook 与真实通知通道：

| 功能              | 路径                                                         |
| ----------------- | ------------------------------------------------------------ |
| Alertmanager 配置 | `infra/alertmanager/alertmanager.yml`                        |
| 后端统一接收端点  | `POST /api/internal/alerts/webhook`（内部白名单，不对外鉴权） |
| 后端通道转发端点  | `POST /api/internal/alerts/notify/{channel}`（channel: dingtalk / wechat_work / feishu / sms） |
| 管理后台查看      | `GET /api/admin/alerts/recent?limit=20`（需要 `ADMIN` 角色） |
| 告警确认          | `POST /api/admin/alerts/{fingerprint}/acknowledge`（需要 `ADMIN` 角色） |

#### 默认路由策略

| 告警级别   | 通知通道                          |
| ---------- | --------------------------------- |
| `critical` | 后端 webhook + 短信 + 钉钉 + 邮件 |
| `warning`  | 后端 webhook + 飞书 + 邮件        |
| 默认       | 后端 webhook                      |

#### 最小可用配置

1. 复制环境变量模板并填写真实凭据：

```bash
cp .env.example .env
```

2. 按需取消 `.env` 中对应行的注释并替换：

```env
# 邮件告警
ALERT_EMAIL_ENABLED=true
ALERT_EMAIL_SMTP_HOST=smtp.example.com
ALERT_EMAIL_SMTP_PORT=587
ALERT_EMAIL_FROM=alertmanager@example.com
ALERT_EMAIL_USERNAME=alertmanager@example.com
ALERT_EMAIL_PASSWORD=your-email-password
ALERT_EMAIL_TO=oncall@example.com

# 钉钉机器人
DINGTALK_ALERT_ENABLED=true
DINGTALK_WEBHOOK_URL=https://oapi.dingtalk.com/robot/send?access_token=your-token
DINGTALK_AT_MOBILES=13800138000

# 企业微信机器人
WECHAT_WORK_ALERT_ENABLED=true
WECHAT_WORK_WEBHOOK_URL=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=your-key

# 飞书机器人
FEISHU_ALERT_ENABLED=true
FEISHU_WEBHOOK_URL=https://open.feishu.cn/open-apis/bot/v2/hook/your-token

# 短信网关
SMS_ALERT_ENABLED=true
SMS_WEBHOOK_URL=https://sms-gateway.example.com/api/v1/send
SMS_API_KEY=your-sms-api-key
SMS_TEMPLATE_CODE=your-template-code
SMS_TO_LIST=13800138000,13900139000
```

3. 重新加载 Alertmanager：

```bash
docker compose restart alertmanager
# 或在线重载
curl -X POST http://localhost:9093/-/reload
```

> 邮件通道由 Alertmanager 原生 `email_configs` 直接发送；钉钉/企业微信/飞书/短信由 Alertmanager 的 `webhook_configs` 推送到后端 `/api/internal/alerts/notify/{channel}`，由后端统一转换为各平台所需的消息格式后转发。这样可以在不修改 Alertmanager 行为的前提下，集中维护模板、签名与限流逻辑。

### 大盘面板

`infra/grafana/dashboards/agentcraft-claw.json` 预置了以下面板：

- RAG 检索耗时（按模式）
- LLM 生成平均耗时
- LLM 首 Token 平均延迟
- 文档处理耗时（按状态）
- 自反思审查速率
- Harness 决策分布
- Loop 迭代轮次
- TechBlogHub 抓取/导出指标
- 多模态 OCR / 视觉检索耗时
- 本地模型健康状态
- 自适应规划进化成功率
- Agent 协作委托与回调指标

---

## 常见问题与故障排查

### 后端构建

#### `mvn test` 在 Windows 上失败或 JAVA_HOME 不正确

Windows 本地若存在多个 JDK 或 Maven 找不到正确 Java 路径，建议直接使用项目提供的 Docker 测试镜像：

```bash
cd backend
MSYS_NO_PATHCONV=1 docker run --rm -v "$(pwd -W)":/app -w /app agentcraft-backend:builder mvn test -Dspring.profiles.active=test
```

> 该镜像已固定 Java 17 与 Maven 版本，挂载当前 `backend` 目录到 `/app` 后执行测试，避免本地环境差异。

#### 集成测试报 `Table "xxx" not found`

测试使用 H2 内存数据库，表结构由 `src/test/resources/schema.sql` 初始化。新增业务表后需同步更新该文件，否则涉及新表的集成测试会失败。

#### 测试报 `ResponseBodyEmitter` 未触发 `onCompletion`

`SseEmitter.complete()` 只有在存在 HTTP handler 时才会触发 callback；单元测试中请使用 `spy(emitter)` 并通过 `verify(emitter).complete()` 断言流程结束，而非等待 callback。

### 前端构建

#### `npm run type-check` 报 `any` 类型警告

项目已配置 ESLint + Prettier（见 `frontend/eslint.config.js`）。开发中应尽量减少 `any` 使用，优先为 API 响应与组件状态声明接口。

#### SSE 连接中断或收到重复错误事件

检查是否同时注册了 `sse.onerror` 与 `sse.addEventListener('error', ...)`。EventSource 的两种形式会同时触发，导致重复清理。保留其一即可，并确保 URL 参数使用 `URLSearchParams.toString()` 进行编码。

### TechBlogHub 抓取失败

#### 目标网站返回 403 / 反爬拦截

1. 检查 `.env` 中 `TECH_BLOG_REQUEST_INTERVAL_MS` 是否 ≥ 3000 ms
2. 确认目标 URL 在 `application.yml` `tech-blog.url-whitelist` 列表中
3. 开启 Playwright 动态渲染：`TECH_BLOG_PLAYWRIGHT_ENABLED=true`（需确保容器/宿主机已安装浏览器二进制）
4. 查看后端日志中 `ContentFetchTool` 的降级路径与 `TechBlogMetrics` 失败标签

#### Reviewer 评分持续不达标

1. 检查 `tech_blog.reviewer.threshold` 配置是否过高（默认 0.85）
2. 在管理后台查看单篇文章的 `fail_reason` 与 Reviewer 四维分数
3. 调整翻译/摘要提示词模板 `backend/src/main/resources/prompts/`

### 多模态 RAG 常见问题

#### OCR 识别率低

1. 检查 `MULTIMODAL_OCR_ENGINE` 设置：Tesseract 适合印刷体，Ollama 多模态适合复杂图文混排
2. 确认图片分辨率 ≥ 300 DPI，过低分辨率会导致文字模糊
3. 查看后端日志中 `OcrService` 的识别耗时与置信度

#### 视觉检索结果不准确

1. 确认 `MULTIMODAL_EMBEDDING_URL` 指向的多模态 Embedding 服务可用
2. 检查 Milvus 图片 Collection 是否已建立索引（`kb_image_collection`）
3. 在 AI 配置中心调整 `rag.vector_top_k` 与 `rag.rrf_top_n`

### 本地模型部署常见问题

#### 本地模型无法连通

1. 确认 `LOCAL_MODEL_BASE_URL` 正确指向 vLLM/Ollama/TGI 的 OpenAI-compatible API 端点
2. 使用管理员接口 `POST /api/admin/local-models/{id}/test` 发送探测请求，查看详细错误
3. 检查本地模型服务是否正常启动：`curl http://localhost:8000/v1/models`
4. 确认防火墙/网络策略允许后端容器访问本地模型服务

#### TEI Embedding 服务异常

1. 确认 `TEI_BASE_URL` 指向 TEI 服务地址，如 `http://localhost:8080`
2. 检查 TEI 服务日志中的模型加载状态
3. 验证 Embedding 维度是否与 Milvus Collection 一致（默认 1024）

### 自适应规划常见问题

####  Planner 进化后效果变差

1. 在管理员后台查看 `PlannerEvolutionLog` 的 A/B 对比，确认进化前后的指标变化
2. 使用回滚功能恢复到上一版本策略
3. 检查 `planner.evolution.threshold` 是否设置过高或过低

#### 进化任务未触发

1. 确认 `planner.evolution.enabled=true`
2. 检查 Cron 表达式 `planner.evolution.cron` 是否正确（默认 `0 17 3 * * ?`，每天 3:17）
3. 查看后端日志中 `PlannerEvolutionScheduler` 的执行记录

### Agent 协作常见问题

#### 外部 Agent 回调失败

1. 确认 `AGENT_INTEROP_CALLBACK_SECRET` 与外部平台配置一致
2. 检查回调请求中的 `X-Interop-Signature` HMAC 签名是否正确生成
3. 查看后端日志中 `AgentTaskDelegator` 的委托与回调处理记录

#### 外部 Agent 注册不上

1. 确认 Redis 可用，`AgentRegistryService` 依赖 Redis 存储注册信息
2. 检查外部 Agent 的心跳是否按时发送（默认 TTL 30 秒）
3. 查看 `GET /api/openclaw/agents/registry` 返回的已注册 Agent 列表

---

## 前端构建优化

- **Element Plus 按需自动导入**：使用 `unplugin-auto-import` + `unplugin-vue-components` + `ElementPlusResolver`，仅打包实际使用的组件，避免全量引入。
- **手动代码分割**：`vite.config.ts` 中配置 `manualChunks`，将 `vue-vendor`、`echarts`、`markdown` 等拆分为独立 chunk。
- **构建结果**：`npm run build` 通过，无大于 1MB 的 JS chunk 警告，首屏加载更快。
- **代码规范**：项目已内置 ESLint + Prettier，运行 `npm run lint` 检查，`npm run lint:fix` 自动修复，`npm run format` 格式化。

---

## 近期演进

以下功能基于 **2026-06-27 演进建议** 已全面落地：

| 功能                   | 状态     | 说明                                                         |
| ---------------------- | -------- | ------------------------------------------------------------ |
| **多模态 RAG**         | ✅ 已上线 | 支持图片/PDF 作为输入，OCR + 多模态 Embedding + 视觉检索融合，Milvus 双 Collection |
| **私有化模型一体机**   | ✅ 已上线 | vLLM/Ollama/TGI 本地部署，OpenAI-compatible API 统一接入，TEI 私有化 Embedding，双层缓存，Actuator 健康检查 |
| **自适应规划**         | ✅ 已上线 | Planner 基于历史成功率自动进化 Prompt 与工具策略，定时进化 + 阈值触发，版本管理与回滚 |
| **Agent 间协作协议**   | ✅ 已上线 | 与 CrewAI/AutoGen/Dify 互调，注册表 + 任务委托 + 异步回调，HMAC-SHA256 签名防伪造 |
| **企业 RBAC 与 SSO**   | ✅ 已上线 | 角色层级继承、权限树、数据权限（ALL/OWN/DEPT/TEAM），LDAP/AD/OAuth2/OIDC 单点登录 |
| **知识库前端版本管理** | ✅ 已上线 | 版本列表/发布/回滚/Diff 对比，LCS 行级高亮，iOS 26 Liquid Glass UI |

---

## 许可证

MIT License (c) 2025 AgentCraft-Claw Team

---

## 致谢

- **Spring AI** — AI 原生应用框架
- **Milvus** — 高性能向量数据库
- **Element Plus** — Vue 3 组件库
- **Tavily** — 实时网页搜索 API
- **DashScope** — 通义千问大模型平台
- **vLLM / Ollama / TGI** — 高性能本地大模型推理框架
- **TEI** — HuggingFace Text Embeddings Inference
- **Apple iOS Design** — Liquid Glass 视觉灵感
