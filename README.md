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
  <b>AgentCraft-Claw</b> — 下一代 Agentic 智能知识系统。以目标驱动的 Craft Engine 为核心，融合多 Agent 协作编排、七步 RAG 检索链与 MCP 工具生态，打造具备自主工作能力的 AI 智能体平台。
</p>


---

## 核心能力

AgentCraft-Claw 突破传统“文档问答”边界，构建具备自主规划、执行与交付能力的智能体系统：

- **智能问答（Chat）** — 基于七步检索链（Vector + BM25 + RRF + Rerank + Compression）的精准 RAG；新版 `CraftEngine` 流式生成采用完全异步 `.subscribe()` 不阻塞 Tomcat 线程，旧版 `RagPipeline` 仍使用 `.blockLast()` 同步等待生成结束，支持 SSE 流式输出、来源可溯与答案句子级溯源
- **任务工坊（Craft）** — 目标驱动的任务执行引擎，支持三种执行模式：简单查询、复杂任务（OTAC 循环 + Swarm 多 Agent 协作）、人机协作（检查点暂停确认）
- **知识库管理** — 支持 PDF/DOCX/MD/TXT 多格式上传，基于独立线程池异步解析、智能分片、向量化索引，具备并发去重与失败重试能力；生命周期管理包含事务化删除
- **联网检索** — 集成 Tavily 实时网页搜索，知识库与互联网双源交叉验证，确保信息时效性
- **语音输入** — 实时 WebSocket 语音转文字（DashScope Paraformer），支持自然语音交互
- **MCP 工具链** — 通过 MCP 协议对外暴露工具能力，供外部 AI 客户端与自动化流程调用
- **AI 配置中心** — 数据库驱动参数动态热更新，API Key、模型选择、联网开关等无需重启服务
- **数据大屏** — `StatsServiceImpl` 统一聚合真实 `qa_message` / `kb_knowledge_base` / `mcp_tool_registry` 数据，Controller 仅做参数校验与结果包装；后端内置 Micrometer 可观测指标
- **iOS 26 Liquid Glass** — 前端采用液态玻璃设计语言，动态流体渐变、多层折射高光与 Spring 动效
- **Tech Blog Hub** — 一句话指令驱动，自动抓取 Anthropic / OpenAI / Google DeepMind / Meta AI 等 AI 大厂技术博客，翻译、摘要、标签化、向量化后生成可离线浏览的静态知识库站点

---

## 架构设计

```
+------------------------------------------------------------------+
|                        接入层 (Access Layer)                      |
|  +--------+  +--------+  +-------------+  +------------------+  |
|  | REST   |  | SSE    |  |  WebSocket  |  | MCP / OpenClaw   |  |
|  | API    |  | Stream |  |  Speech     |  | Protocol         |  |
|  +--------+  +--------+  +-------------+  +------------------+  |
+------------------------------------------------------------------+
|                        编排层 (Orchestration)                     |
|  +--------------+  +--------------+  +---------------------+   |
|  | CraftEngine  |  | AutoPlanner  |  | SwarmOrchestrator  |   |
|  | (OTAC 循环)  |  | (任务规划)   |  | (多 Agent 协作编排) |   |
|  +--------------+  +--------------+  +---------------------+   |
|  +--------------+  +--------------+  +---------------------+   |
|  | LoopEngine   |  | HarnessEngine|  | OpenClawProtocol   |   |
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
|  +-------------------------------------------------------------+|
|  | TechBlogHub（RSS/Sitemap 发现 → 抓取 → 翻译 → 摘要 → 标签 →  |
|  | 索引 → 离线站点导出）                                          |
|  +-------------------------------------------------------------+|
+------------------------------------------------------------------+
|                        记忆层 (Memory)                            |
|  +----------+  +----------+  +----------+  +-----------------+  |
|  | Working  |  | Semantic |  | Episodic |  | Procedural      |  |
|  | Memory   |  | Memory   |  | Memory   |  | Memory          |  |
|  | (Redis)  |  | (Redis)  |  | (Redis)  |  | (Redis)         |  |
|  +----------+  +----------+  +----------+  +-----------------+  |
+------------------------------------------------------------------+
|                        基础设施 (Infrastructure)                  |
|  Spring Boot | Spring AI | MyBatis Plus | MySQL | Redis | Milvus |  |
|  | MinIO |                                                              |
+------------------------------------------------------------------+
```

### 三种执行模式

| 模式              | 触发条件                     | 执行流程                                               | 适用场景                   |
| ----------------- | ---------------------------- | ------------------------------------------------------ | -------------------------- |
| **SIMPLE**        | 短问题、疑问词开头           | 直接调用 ReAct Agent 引擎 -> 知识库检索 -> 即时回答    | "什么是 RAG？"             |
| **COMPLEX**       | 包含分析/生成/报告等复杂动词 | AutoPlanner -> Swarm 编排 -> 多 Agent 协作 -> 结果综合 | "生成一份 AI 行业研究报告" |
| **COLLABORATIVE** | 包含"帮我确认""先给我看看"   | 计划 -> 检查点暂停 -> 人类确认 -> 继续执行             | "帮我确认这个方案是否可行" |

### Swarm 深度执行

复杂任务（COMPLEX 模式）由 `SwarmOrchestrator` 驱动多 Agent 协作，核心机制包括：

- **DAG 拓扑并行**：`AutoPlanner` 生成的步骤列表经 `DagScheduler` 使用 Kahn 算法分层，无依赖步骤在同一层并行执行（`CompletableFuture` + 固定线程池），依赖步骤按层串行等待。
- **Reviewer 深度校验**：检查点步骤触发 Reviewer Agent，按 **事实性 / 完整性 / 安全性 / 格式** 四个维度输出结构化评分与改进建议；未通过时可暂停等待人工介入（COLLABORATIVE 模式）或跳过非关键步骤。
- **长期记忆召回**：每个步骤执行前，`MemoryRecallService` 从 **Semantic（事实）、Episodic（经验）、Procedural（策略模板）** 三层记忆中召回相关内容，写入工作上下文；执行结束后自动保存当前经验到 Episodic Memory。

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
- **BudgetTracker**：按 Token 数量、步骤数、耗时三个维度跟踪预算，超过阈值时触发降级或暂停。
- **Reflection Loop**：自纠错循环，最多执行 2 轮，每轮评估答案置信度，未通过则补充检索或改写问题。

#### HarnessEngine（约束治理）

`HarnessEngine` 在输入进入执行体前与输出发送给用户前分别进行约束检查：

- **PRE_INPUT**：敏感词过滤、Prompt 注入检测、权限校验、输入长度与格式约束。
- **POST_OUTPUT**：事实一致性校验、安全兜底、输出格式检查、禁忌词二次过滤。
- 任一阶段触发约束时返回结构化 `HarnessDecision`，上层根据 `DECISION_BLOCK` / `DECISION_WARN` / `DECISION_ALLOW` 决定放行、告警或拦截。

#### Loop / Harness 给系统带来的收益

| 维度         | 收益                                                         | 典型场景                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **可靠性**   | LoopEngine 防止任务陷入死循环，Reviewer 与 Reflection 保证输出质量下限 | 复杂报告生成时自动重试低质量步骤                             |
| **可控性**   | BudgetTracker 按 Token / 步骤 / 耗时三维设限，避免单次请求耗尽资源 | 用户提交“分析全站日志”等可能产生超长链的任务                 |
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
- **与 SafetyGuard / GuardRailService**：Harness 的 PRE_INPUT / POST_OUTPUT 约束底层复用 `SafetyGuard` 的紧急词检测与 `GuardRailService` 的语义护栏，形成“规则 + 语义 + 策略”三层防护。
- **与 MCP / OpenClaw**：外部通过 OpenClaw 调用的工具请求同样先经过 Harness 输入检查，工具结果返回前再经过输出检查；违规调用会被记录到审计日志。
- **与 MetricsService / Grafana**：`LoopEngine` 上报 `craft.loop.round`、`craft.reviewer.retry`；`HarnessEngine` 上报 `harness.decision.total`（按 `level`、`decision` 标签），与 RAG、LLM 指标在同一大盘展示。
- **与 Memory**：Loop 的 Reviewer 结果与 Reflection 日志写入 `SharedMemoryBus` 的当前 Craft 上下文，并持久化到 `qa_message`，支持长期经验回放。

### LLM 调用韧性治理

大模型调用是系统的核心依赖，也是最容易出现延迟飙升、配额耗尽或服务不可用的环节。系统通过 `ResilientChatModel` 对 `ChatModel` 做统一包装，接入 Resilience4j 的多层保护：

- **熔断（CircuitBreaker）**：按失败率/慢调用率自动打开熔断，避免故障扩散；触发后返回结构化降级响应。
- **限流（RateLimiter）**：按配置限制单位时间内的 LLM 请求数，超出时返回友好提示。
- **隔离（Bulkhead）**：限制 LLM 并发调用数，防止单点依赖耗尽线程资源。
- **超时（TimeLimiter）**：对同步 `call()` 设置可配置超时，超时时触发降级。
- **降级（Fallback）**：熔断/限流/隔离/超时触发后，统一返回带提示语的 `ChatResponse`，不向上抛异常导致 SSE 断连。

`ResilientChatModel` 同时覆盖同步 `call(Prompt)` 与流式 `stream(Prompt)` 两条路径：同步路径使用 `Callable` 装饰器链；流式路径使用 Resilience4j Reactor Operator 在 Flux 上应用限流、隔离与熔断。`LlmResilienceConfig` 暴露 `llmCircuitBreaker`、`llmRateLimiter`、`llmBulkhead`、`llmTimeLimiter` 四个命名实例，所有阈值均通过 `llm.resilience.*` 配置项外置到 `application.yml`，支持运行时调整而无需改代码。

### OpenClaw 协议

OpenClaw 是系统对外暴露工具能力的开放协议，兼容 MCP（Model Context Protocol）规范：

- **工具注册**：后端通过 `@Tool` 注解将能力注册到 `McpToolRegistry`，运行时自动发现。
- **工具发现**：外部客户端可通过 `GET /api/mcp/tools` 获取工具元数据、参数模式与示例。
- **工具调用**：`POST /api/mcp/tools/{id}/test` 支持在线测试，`/api/mcp/invoke` 供授权客户端以 JSON-RPC 风格调用。
- **安全边界**：所有外部调用均需 JWT 认证，敏感工具额外校验角色权限；调用结果写入审计日志。

---

## 技术栈

### 后端 (Spring Boot 3.4)

| 层         | 技术                                 | 说明                                                     |
| ---------- | ------------------------------------ | -------------------------------------------------------- |
| 框架       | Spring Boot 3.4.13 + Spring AI 1.1.4 | AI 原生开发框架                                          |
| 安全       | Spring Security + JWT (jjwt 0.12.5)  | 无状态认证，BCrypt 加密，HttpOnly Cookie，SSE 一次性票据 |
| ORM        | MyBatis Plus 3.5.7                   | 高效 CRUD + 条件构造器                                   |
| 向量数据库 | Milvus 2.3.4                         | 1024 维向量检索                                          |
| 缓存       | Redis 7 + Lettuce                    | 频率缓存 + 会话记忆 + SSE 票据                           |
| 对象存储   | MinIO 8.5.7                          | 文档文件存储                                             |
| 文档解析   | Apache PDFBox 3.0 + POI 5.2          | PDF/DOCX 提取                                            |
| 关键词搜索 | Lucene 8.11 + smartcn 分词           | 中文 BM25 检索                                           |
| 韧性治理   | Resilience4j 2.1.0                   | LLM / 搜索 / 工具调用的熔断、限流、隔离、超时与降级      |
| 语音识别   | DashScope Paraformer                 | 实时流式 ASR                                             |
| 联网搜索   | Tavily API                           | 实时网页检索                                             |
| AI 配置    | AiConfigHolder + AiConfigInitializer | 数据库驱动热更新，原子替换模型实例                       |

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

| 服务         | 容器端口 | 宿主机端口 | 说明                    |
| ------------ | -------- | ---------- | ----------------------- |
| MySQL        | 3306     | 3307       | 业务数据库              |
| Redis        | 6379     | 6380       | 缓存 + 会话 + SSE 票据  |
| Milvus       | 19530    | 19530      | 向量数据库              |
| MinIO API    | 9000     | 9002       | 对象存储 API            |
| MinIO 控制台 | 9001     | 9003       | 对象存储控制台          |
| Alertmanager | 9093     | 9093       | Prometheus 告警通知管理 |
| etcd         | 2379     | —          | Milvus 元数据依赖       |
| 后端         | 8080     | 8080       | REST + SSE API          |
| 前端         | 80       | 80         | 主入口                  |

---

## 项目结构

```
AgentCraft-Claw/
|
|-- backend/                             # 后端 (Spring Boot)
|   |-- pom.xml
|   |-- Dockerfile                        # 后端容器镜像（多阶段构建）
|   |-- docker-compose.dev.yml            # 仅基础设施（MySQL/Redis/MinIO/Milvus/etcd）
|   |-- src/main/java/com/simon/agentcraft/
|   |   |-- AgentCraftApplication.java
|   |   |-- agent/                        # ReAct Agent 核心
|   |   |-- common/                       # 结果封装、异常、JWT
|   |   |-- config/                       # 配置中心（AI 热更新、MCP、Milvus、MinIO、Redis）
|   |   |-- controller/                   # REST + WebSocket + MCP 控制器
|   |   |-- craft/                        # Craft Engine + AutoPlanner
|   |   |-- entity/                       # 实体 + DTO + VO
|   |   |-- harness/                      # HarnessEngine（PRE_INPUT/POST_OUTPUT 约束治理）
|   |   |-- loop/                         # LoopEngine（Otac/Reviewer/Budget/Reflection）
|   |   |-- mapper/                       # MyBatis Plus Mapper
|   |   |-- mcp/                          # MCP 工具（@Tool 注解暴露）
|   |   |-- memory/                       # SharedMemoryBus（四级记忆）
|   |   |-- metrics/                      # Micrometer 指标与数据大屏服务
|   |   |-- openclaw/                     # OpenClaw 协议（对外暴露工具能力）
|   |   |-- reflection/                   # CheckpointService（人机协作检查点）
|   |   |-- safety/                       # GuardRailService（多维安检）
|   |   |-- security/                     # JWT 认证 + SSE 票据 + Spring Security
|   |   |-- service/                      # 业务服务
|   |   |   |-- code/                     # 代码生成/审查
|   |   |   |-- knowledge/                # 文档处理 + Milvus + MinIO
|   |   |   |-- rag/                      # RAG 七步检索链
|   |   |-- swarm/                        # SwarmOrchestrator（多角色协作）
|   |-- src/main/resources/
|   |   |-- application.yml               # 主配置（LLM/DB/Redis/Milvus/MinIO）
|   |   |-- prompts/                      # 安全审查、查询改写、知识问答提示词
|
|-- frontend/                             # 前端 (Vue 3)
|   |-- package.json
|   |-- vite.config.ts                    # 开发代理、代码分割
|   |-- Dockerfile                        # 前端容器镜像（Node 22 -> nginx）
|   |-- nginx.conf                        # 反向代理 /api，SSE 无缓冲，history 模式
|   |-- src/
|   |   |-- api/                          # Axios API 封装
|   |   |-- components/                   # 组件（含 Liquid Glass）
|   |   |-- layout/                       # 主布局
|   |   |-- router/                       # 路由配置（含管理员守卫）
|   |   |-- stores/                       # Pinia 状态
|   |   |-- styles/                       # 全局样式
|   |   |-- utils/                        # 工具函数
|   |   |-- views/                        # 页面视图
|
|-- docs/
|   |-- agentcraft.sql                    # 完整数据库脚本（含 sys_ai_config 表）
|-- infra/                                # 可观测性与告警配置
|   |-- prometheus/
|   |   |-- prometheus.yml                # Prometheus 抓取与告警管理器配置
|   |   |-- alerts.yml                    # Prometheus 告警规则
|   |-- grafana/                          # Grafana 数据源、大盘与插件预配置
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
# 期望：backend / frontend / mysql / redis / minio / milvus / prometheus / grafana / alertmanager 均为 healthy
```

### 2. 后端健康检查

```bash
curl http://localhost/api/actuator/health
```

### 3. Prometheus Targets

访问 http://localhost:9090/targets，确认 `agentcraft-backend` 状态为 **UP**。

### 4. Grafana 数据源

访问 http://localhost:3000（默认 admin / admin），进入 **Configuration -> Data sources**，确认 Prometheus 与 Loki 测试通过。

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
mysql -u root -p -h localhost -P 3307 < docs/agentcraft.sql
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

| 变量                                              | 必填 | 说明                                                      |
| ------------------------------------------------- | ---- | --------------------------------------------------------- |
| `BAILIAN_API_KEY`                                 | 是   | 阿里云百炼 API Key，用于 LLM / Embedding / Rerank         |
| `JWT_SECRET`                                      | 是   | JWT 签名密钥，长度 ≥ 32 字节                              |
| `MYSQL_ROOT_PASSWORD`                             | 建议 | MySQL root 密码                                           |
| `MINIO_ROOT_PASSWORD`                             | 建议 | MinIO root 密码                                           |
| `GRAFANA_ADMIN_USER` / `GRAFANA_ADMIN_PASSWORD`   | 建议 | Grafana 管理员账号，生产必须修改                          |
| `REDIS_PASSWORD`                                  | 否   | Redis 密码（留空表示无密码）                              |
| `CORS_ALLOWED_ORIGINS`                            | 否   | 前端访问域名，多个用英文逗号分隔                          |
| `DOCMIND_WEB_SEARCH_ENABLED`                      | 否   | 联网搜索开关，默认 `false`                                |
| `TAVILY_API_KEY`                                  | 否   | Tavily API Key，开启联网搜索时必填                        |
| `TECH_BLOG_MINIO_BUCKET`                          | 否   | TechBlogHub 离线站点 MinIO bucket，默认 `tech-blog-sites` |
| `TECH_BLOG_REQUEST_INTERVAL_MS`                   | 否   | 博客抓取请求间隔（反爬），默认 `3000`                     |
| `TECH_BLOG_MAX_RETRIES`                           | 否   | 博客抓取最大重试次数，默认 `3`                            |
| `TECH_BLOG_PLAYWRIGHT_ENABLED`                    | 否   | 是否启用 Playwright 动态渲染，默认 `false`                |
| `TECH_BLOG_REVIEWER_THRESHOLD`                    | 否   | TechBlogHub Reviewer 评分阈值，默认 `0.85`                |
| `APP_LOG_LEVEL`                                   | 否   | 日志级别，默认 `INFO`                                     |
| `ALERT_EMAIL_*`                                   | 否   | 邮件告警 SMTP 配置                                        |
| `DINGTALK_WEBHOOK_URL`                            | 否   | 钉钉群机器人 Webhook                                      |
| `WECHAT_WORK_WEBHOOK_URL`                         | 否   | 企业微信群机器人 Webhook                                  |
| `FEISHU_WEBHOOK_URL`                              | 否   | 飞书群机器人 Webhook                                      |
| `SMS_WEBHOOK_URL` / `SMS_API_KEY` / `SMS_TO_LIST` | 否   | 短信网关配置                                              |

---

## 核心配置

### AI 参数配置（动态热更新）

系统支持通过 `sys_ai_config` 表动态调整参数，无需重启服务。启动时由 `AiConfigInitializer` 加载到 `AiConfigHolder` 内存快照，配置变更后原子替换 `ChatModel` 实例，正在进行的 SSE 请求持有旧引用自然完成，新请求自动使用新配置。

| 参数                            | 默认值                                           | 说明                                               |
| ------------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| `llm.api_key`                   | —                                                | LLM API Key（覆盖环境变量）                        |
| `llm.base_url`                  | `https://dashscope.aliyuncs.com/compatible-mode` | LLM 基础 URL                                       |
| `llm.model`                     | `qwen-plus`                                      | 对话模型                                           |
| `llm.chat_temperature`          | `0.1`                                            | 对话温度                                           |
| `llm.timeout_seconds`           | `60`                                             | 请求超时（秒）                                     |
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

---

## API 概览

### 智能问答

```http
GET /api/v2/chat/stream?message=What%20is%20RAG&ticket=xxx&conversationId=&kbIds=1,2
Accept: text/event-stream
```

> SSE 接口使用一次性票据 `ticket` 进行认证，避免 JWT 暴露在 URL 查询参数中。前端通过 `POST /api/sse/ticket` 获取票据。

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

### 任务工坊

```http
GET /api/craft/stream?goal=Generate%20AI%20Research%20Report&ticket=xxx&kbIds=1,2
Accept: text/event-stream
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
GET /api/v2/tech-blog/articles?source=anthropic&tag=Agent&page=1&size=20

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

## 安全特性

- **JWT 无状态认证**：24h 访问令牌 + 7d 刷新令牌
- **BCrypt 密码哈希**：强度 10 的 bcrypt 加密
- **角色权限控制**：`ROLE_ADMIN` / `ROLE_USER`，管理员路由前端守卫 + 后端 `@PreAuthorize` 双重校验
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
- **依赖安全**：前端已执行 `npm audit fix`，修复 12 个已知依赖漏洞（含 1 个 critical），`npm audit` 当前报告 0 漏洞

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

| 类型     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 单元测试 | `AlertNotificationServiceTest`、`DocumentProcessTaskTest`、`CrossEncoderRerankerTest`、`LocalRerankerTest`、`ProvenanceServiceTest`、`MetricsServiceTest`、`GuardRailServiceTest`、`AlertMessageFormatterTest`、`DingTalkAlertChannelTest` |
| 集成测试 | `JwtAuthenticationIntegrationTest`（JWT 签发/鉴权/过期）、`ControllerIntegrationTest`（公开端点/管理员权限/参数校验）、`RagPipelineIntegrationTest`（RAG 七步链路调用顺序） |
| 组件测试 | `DagSchedulerTest`（DAG 拓扑分层与环检测）、`SwarmOrchestratorTest`（并行执行/记忆召回/Reviewer） |

> 集成测试使用 `test` profile + H2 内存库 + `@MockBean` 模拟外部依赖，无需 Docker 即可运行。

---

## 可观测性

部署后自动包含 Prometheus + Grafana + Loki 栈：

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

### 日志聚合

后端通过 `loki-logback-appender` 将日志推送到 Loki：

- 默认推送地址：`http://localhost:3100/loki/api/v1/push`
- Docker Compose 中自动设置为 `http://loki:3100`
- 可通过环境变量 `LOKI_URL` 覆盖
- `test` profile 下关闭 Loki 推送，避免单元测试产生网络请求

Grafana 中已预置 Loki 数据源，可直接在 Explore 中按 `app=agentcraft-claw`、`level`、`thread` 等标签检索日志。

### 告警规则

Prometheus 已加载 `infra/prometheus/alerts.yml`，预置以下告警：

| 告警名                                | 触发条件                                  | 级别     |
| ------------------------------------- | ----------------------------------------- | -------- |
| `AgentCraftHighRetrievalLatency`      | RAG 检索平均耗时 > 2s，持续 5 分钟        | warning  |
| `AgentCraftHighFirstTokenLatency`     | LLM 首 token 平均延迟 > 5s，持续 5 分钟   | warning  |
| `AgentCraftHighLlmGenerationLatency`  | LLM 完整生成平均耗时 > 30s，持续 5 分钟   | warning  |
| `AgentCraftDocumentProcessingFailing` | 最近 5 分钟文档处理失败率 > 0             | critical |
| `AgentCraftReflectionFailing`         | 自反思不通过速率 > 0.1 次/秒，持续 5 分钟 | warning  |
| `AgentCraftHighJvmHeapUsage`          | JVM 堆内存使用率 > 85%，持续 5 分钟       | critical |

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

---

## 常见问题与故障排查

### 后端构建

#### `mvn test` 在 Windows 上失败或 JAVA_HOME 不正确

Windows 本地若存在多个 JDK 或 Maven 找不到正确 Java 路径，建议直接使用项目提供的 Docker 测试镜像：

```bash
cd backend
MSYS_NO_PATHCONV=1 docker run --rm -v "$(pwd -W)":/app -w /app agentcraft-backend:test mvn test -Dspring.profiles.active=test
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

---

## 前端构建优化

- **Element Plus 按需自动导入**：使用 `unplugin-auto-import` + `unplugin-vue-components` + `ElementPlusResolver`，仅打包实际使用的组件，避免全量引入。
- **手动代码分割**：`vite.config.ts` 中配置 `manualChunks`，将 `vue-vendor`、`echarts`、`markdown` 等拆分为独立 chunk。
- **构建结果**：`npm run build` 通过，无大于 1MB 的 JS chunk 警告，首屏加载更快。
- **代码规范**：项目已内置 ESLint + Prettier，运行 `npm run lint` 检查，`npm run lint:fix` 自动修复，`npm run format` 格式化。

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
- **Apple iOS Design** — Liquid Glass 视觉灵感
