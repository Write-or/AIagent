# AI Agent 开发 · 84 天完整学习规划

> 制定日期：2026-09-28
> 节奏：每天 2–3 小时，共 12 周 / 84 天，8 个阶段，7 个里程碑。
> 目标：结束时能独立交付一个**可上线**的 Agent 应用，并具备面试/求职所需的技术叙事。

---

## 0. 使用说明

### 每日固定节奏（2.5h 模板）
| 时段 | 时长 | 内容 |
|---|---|---|
| 读 | 40 min | 官方文档 / 论文，只读当前任务需要的部分，不做"收藏式学习" |
| 写 | 80 min | 动手写代码，必须有可运行产出 |
| 记 | 20 min | 写 `notes/day-XX.md`：今天学到什么、卡在哪、怎么解决的 |
| 复 | 10 min | 更新进度表，标注明天的第一个动作 |

### 三条铁律
1. **产出驱动**：每天必须有 commit。没有产出的学习日 = 无效日。
2. **先手写再上框架**：Day 16 会要求你不用任何框架手写 Agent 循环。跳过这步，后面调 bug 全靠玄学。
3. **不追框架版本**：LangChain 之类的 API 每季度都在变。学**模式**（工具循环、状态机、检索管线），框架只是实现。

### 每周日固定动作
- 跑通本周所有 demo（回归验证，别让代码腐烂）
- 写周复盘：本周最大认知更新是什么？
- 若落后 > 2 天：砍掉当周的"进阶"内容，保里程碑。

---

## 1. 阶段总览

| 阶段 | 天数 | 主题 | 里程碑 |
|---|---|---|---|
| P0 | 1–5 | 工程环境与工程化 Python | M0 项目脚手架 |
| P1 | 6–14 | LLM API、Prompt、上下文工程 | M1 可用的 CLI Agent |
| P2 | 15–26 | Agent 核心原理（手写实现） | M2 自研迷你 Agent 框架 |
| P3 | 27–38 | 框架、协议、可观测性 | M3 生产级单 Agent 服务 |
| P4 | 39–50 | RAG 与检索系统 | M4 文档问答系统（带评测） |
| P5 | 51–62 | 多 Agent、多模态、安全 | M5 多 Agent 协作系统 |
| P6 | 63–72 | 生产化、部署、成本、评测回归 | M6 上线真服务 |
| P7 | 73–84 | 综合项目、开源、求职 | 作品集 + 技术复盘 |

---

## P0 · 工程环境（Day 1–5）

### Day 1 · 环境搭建与项目骨架
- **学**：为什么用 `uv`（比 pip/conda 快且能锁依赖）；`pyproject.toml` 标准；Python 3.12 vs 3.14 的兼容现实
- **做**：装 Python 3.12（`uv python install 3.12`）→ `uv init` 建项目 → 配 ruff + mypy + pre-commit → git init 并首推
- **产出**：仓库骨架（`src/`、`tests/`、`.env.example`、`Makefile`）
- **验收**：`uv run python -c "import sys;print(sys.version)"` 输出 3.12.x；`git log` 有第一个 commit

### Day 2 · 现代 Python 与数据建模
- **学**：类型注解、`dataclass` vs `pydantic v2`、`Protocol`、`Literal`/`Enum`、`field_validator`
- **做**：用 pydantic 定义 `Message`、`ToolSpec`、`ToolCall`、`AgentState` 四个模型，导出 JSON Schema
- **产出**：`src/schemas.py` + 单测
- **验收**：非法输入被拒绝且报错信息可读；`model_json_schema()` 输出可读 schema

### Day 3 · 异步与并发
- **学**：`asyncio`、`TaskGroup`、`asyncio.timeout`、`tenacity` 重试、`httpx` 异步客户端
- **做**：并发请求 5 个接口，加入超时 + 指数退避重试 + 部分失败降级
- **产出**：`src/async_demo.py`
- **验收**：注入人为故障（超时/500），程序仍能返回部分结果而不是崩

### Day 4 · HTTP、流式与 SSE
- **学**：HTTP 语义、chunked 传输、SSE 协议格式、LLM 流式响应的 `data: {...}` 分片
- **做**：**手写** SSE 解析器（不用库），解析多行 data、`[DONE]` 结束、跨 chunk 断包
- **产出**：`src/sse.py` + 用真实接口验证
- **验收**：能逐 token 打印，且断包场景不丢字符（构造测试用例）

### Day 5 · 测试与工程习惯（M0）
- **学**：pytest、`pytest-asyncio`、`respx` 录制 HTTP、`structlog` 结构化日志
- **做**：给 Day3/4 的代码补测试；日志统一加 `trace_id`
- **产出**：测试覆盖率 > 60%；**M0 脚手架完成**
- **验收**：`make test` 一条命令全绿

---

## P1 · LLM 与 Prompt 工程（Day 6–14）

### Day 6 · LLM 原理速通（够用即可）
- **学**：tokenization、上下文窗口、采样参数（temperature/top_p/penalty）、KV Cache、prefill vs decode、为什么"长上下文 ≠ 记得住"
- **做**：用 tiktoken 统计中英文 token 差异，实测同一问题不同 temperature 的输出方差
- **产出**：`notes/day-06.md` 含实测数据
- **验收**：能解释"为什么 Agent 里 temperature 通常设 0~0.3"

### Day 7 · 第一个 API 调用与流式 CLI
- **学**：Chat Completions / Messages API 差异；国内可用通道（DeepSeek、通义、智谱、OpenRouter）；多模态消息格式
- **做**：写流式 CLI 聊天，支持多轮历史、Ctrl+C 中断、token 与成本统计
- **产出**：`src/cli_chat.py`
- **验收**：连续 10 轮对话不丢上下文，成本统计与官方账单量级一致

### Day 8 · Prompt 工程
- **学**：系统提示的层次结构、few-shot、CoT、分隔符防混淆、指令优先级
- **做**：为同一任务写 5 个 prompt 变体，做 A/B 对比（20 条测试用例，人工打分）
- **产出**：`prompts/` 目录 + 对比报告
- **验收**：能说出每个变体提升/下降的具体原因

### Day 9 · 结构化输出
- **学**：JSON mode vs Structured Outputs（JSON Schema 约束）、`response_format`、校验失败后的修复重试
- **做**：让模型从简历文本抽取结构化字段，Pydantic 校验 + 失败自动重试 2 次
- **产出**：`src/extract.py`
- **验收**：50 条样本成功率 > 95%

### Day 10 · 工具调用（Function Calling）原理
- **学**：工具定义 schema、并行工具调用、`tool_call_id` 回填、工具错误的表达方式
- **做**：实现"天气 + 汇率 + 计算器"三工具，观察模型并行调用的行为
- **产出**：`src/tools_demo.py`
- **验收**：模型能自主选择正确工具，工具报错时能自我修正重试

### Day 11 · 上下文工程（Context Engineering）
- **学**：上下文预算分配、历史压缩/摘要、子代理隔离、just-in-time 检索、prompt caching
- **做**：给 CLI 加"历史超过 N token 自动摘要压缩"机制
- **产出**：`src/context.py`
- **验收**：100 轮对话后仍能答出第 1 轮的关键信息

### Day 12 · 评估入门
- **学**：测试集构建、LLM-as-judge 的偏差与校准、`promptfoo` 用法
- **做**：建 30 条黄金测试集，接 promptfoo 跑回归
- **产出**：`evals/` + 一键评测脚本
- **验收**：改 prompt 后能一眼看出通过率变化

### Day 13 · 成本与延迟优化
- **学**：prompt cache、语义缓存、模型路由（小模型分流）、批处理、并行化
- **做**：接入语义缓存（embedding 相似度阈值），实测命中率与成本下降
- **产出**：`src/cache.py` + 成本对比表
- **验收**：重复类请求成本下降 > 50%

### Day 14 · 里程碑 M1 复盘
- **做**：整合本周成果为**一个** CLI Agent：多轮 + 流式 + 3 个工具 + 上下文压缩 + 成本统计 + 测试集
- **产出**：`m1-cli-agent` 打 tag，写 README
- **验收**：能录一段 3 分钟演示；周复盘文档完成

---

## P2 · Agent 核心原理 · 手写实现（Day 15–26）

### Day 15 · ReAct 与 Agent 循环
- **学**：ReAct 论文、Thought-Action-Observation 循环、与 CoT 的区别、循环终止条件
- **做**：用伪代码/流程图画出 Agent 主循环，标出所有可能死循环的点
- **产出**：`notes/day-15.md` 含流程图
- **验收**：能讲清"Agent 与工作流的本质区别是控制权归属"

### Day 16 · 手写 Agent 循环（关键日，禁止用框架）
- **学**：不用 LangChain，用原生 API 实现 tool-calling 循环
- **做**：写 `run_agent(messages, tools, max_steps)`，含步数上限、工具异常捕获、循环检测
- **产出**：`src/mini_agent.py`（< 200 行）
- **验收**：能完成"查资料→计算→给结论"的多步任务

### Day 17 · 工具设计原则
- **学**：工具粒度、命名与描述即 prompt、幂等性、可恢复错误、返回值大小控制
- **做**：重构 Day16 工具，把 1 个"万能工具"拆成 4 个语义清晰的工具，对比成功率
- **产出**：对比报告
- **验收**：拆细后任务成功率提升可量化

### Day 18 · 记忆系统
- **学**：短期记忆（窗口/摘要）、长期记忆（向量/结构化/KV）、写入时机与遗忘策略
- **做**：给 mini agent 加长期记忆（用户偏好持久化到 SQLite + 向量召回）
- **产出**：`src/memory.py`
- **验收**：跨会话记得用户上次的偏好设置

### Day 19 · 规划：Plan-and-Execute
- **学**：任务分解、计划-执行-重规划、与 ReAct 的取舍、计划的质量评估
- **做**：实现 planner 输出 DAG 计划 + executor 按序执行 + 失败重规划
- **产出**：`src/planner.py`
- **验收**：5 步复杂任务端到端成功率 > 70%

### Day 20 · 反思与自我纠错
- **学**：Reflexion、self-critique、verifier 模式、何时反思才划算（成本 vs 收益）
- **做**：加 critic 节点，对比"无反思 / 1 次反思 / 2 次反思"的效果与成本
- **产出**：实验数据表
- **验收**：能给出"反思次数收益递减"的实测结论

### Day 21 · 状态机与图思维
- **学**：状态、节点、边、条件路由、检查点（checkpoint）、为什么图 > 链
- **做**：用显式状态机重写 mini agent（`state` dict + `nodes` + `router`）
- **产出**：`src/graph_agent.py`
- **验收**：能画出状态图并与代码一一对应

### Day 22 · Human-in-the-loop
- **学**：审批点设计、中断与恢复（interrupt/resume）、危险操作白名单
- **做**：给"发邮件/删数据"类工具加人工审批，实现中断后恢复
- **产出**：`src/hitl.py`
- **验收**：中断后重启进程仍能从断点继续

### Day 23 · 会话持久化
- **学**：会话/线程模型、消息存储 schema、并发会话隔离、多租户
- **做**：把状态存到 SQLite（后续迁 Postgres），支持多会话并发
- **产出**：`src/store.py`
- **验收**：并发 20 个会话互不串扰

### Day 24 · 健壮性工程
- **学**：超时、重试、幂等键、熔断、循环检测（重复动作/重复输出）、最大步数与预算上限
- **做**：故障注入测试：让模型返回垃圾、工具永久失败、陷入循环，验证系统能优雅退出
- **产出**：`tests/test_resilience.py`
- **验收**：所有故障场景都有明确兜底行为，无无限循环

### Day 25 · 质量/成本/延迟三角
- **学**：三者权衡的量化方法、何时用大模型、何时用小模型、何时不用 LLM
- **做**：为 mini agent 做一次完整评测：成功率、P50/P95 延迟、单任务成本
- **产出**：性能报告
- **验收**：报告能支撑"某节点换小模型"的决策

### Day 26 · 里程碑 M2
- **做**：把 Day15–25 的成果整合成**自研迷你 Agent 框架**（< 500 行），附 3 个 demo（问答 / 数据分析 / 自动化操作）
- **产出**：`m2-mini-agent-framework` tag + 设计文档
- **验收**：不看代码能讲清架构；能对比说明它与 LangGraph 的差异

---

## P3 · 框架、协议与可观测性（Day 27–38）

### Day 27 · LangGraph 基础
- **学**：`StateGraph`、reducer、条件边、`compile()`、与自研状态机的对应关系
- **做**：用 LangGraph 重写 Day21 的 graph agent，逐行对照
- **产出**：`src/lg_basic.py`
- **验收**：能说出 LangGraph 帮你解决了哪些你自己写时踩过的坑

### Day 28 · LangGraph 进阶
- **学**：`checkpointer`（内存/SQLite/Postgres）、`interrupt`、`Command`、子图（subgraph）、`Send` 并行
- **做**：接 SQLite checkpointer，实现中断-恢复 + 并行分支
- **产出**：`src/lg_advanced.py`
- **验收**：进程重启后能恢复中断的图

### Day 29 · 其他框架横向对比
- **学**：PydanticAI、OpenAI Agents SDK、CrewAI、AutoGen、Google ADK 的定位差异
- **做**：用 PydanticAI 和 OpenAI Agents SDK 各实现同一任务，写对比表（类型安全、调试体验、可观测性、锁定风险）
- **产出**：`notes/day-29.md` 选型报告
- **验收**：能给出针对不同场景的选型建议

### Day 30 · MCP（Model Context Protocol）
- **学**：MCP 协议（tools/resources/prompts）、stdio vs SSE/HTTP transport、客户端-服务端模型
- **做**：写一个 MCP server 暴露你自己的工具；用 Claude Desktop / 客户端连上
- **产出**：`mcp_server/`
- **验收**：外部客户端能发现并调用你的工具

### Day 31 · MCP 进阶与生态
- **学**：认证、权限、多 server 编排、把 MCP 接进你自己的 agent
- **做**：让 mini agent 作为 MCP client 动态加载工具
- **产出**：`src/mcp_client.py`
- **验收**：新增 MCP server 后无需改代码即可获得新能力

### Day 32 · 可观测性：Tracing
- **学**：LLM 可观测性为什么特殊、span/嵌套 span、OTel 语义约定、LangSmith / Langfuse 用法
- **做**：接 Langfuse（或 Phoenix），给 agent 全链路加 trace
- **产出**：可访问的 trace 面板截图
- **验收**：能定位"哪一步慢、哪一步贵、哪一步错"

### Day 33 · 评测体系（进阶）
- **学**：离线 vs 在线评测、组件级 vs 端到端、回归门禁、LLM-judge 的校准与成本
- **做**：给 agent 建组件级评测（工具选择准确率、步骤数、任务成功率）+ CI 门禁
- **产出**：`evals/agent_eval.py`
- **验收**：PR 时自动跑评测并在指标下降时失败

### Day 34 · 提示词与配置管理
- **学**：prompt 版本化、配置与代码分离、灰度与回滚、环境隔离
- **做**：把 prompt 抽到配置文件 + 版本号，支持按版本 A/B
- **产出**：`configs/`
- **验收**：不改代码即可切换 prompt 版本

### Day 35 · 缓存与性能
- **学**：prompt caching（Anthropic/OpenAI）、KV/语义缓存、批处理、并发控制（限流器）
- **做**：加限流 + 并发控制 + 二级缓存，压测 QPS 与延迟
- **产出**：压测报告
- **验收**：P95 延迟有明确改善数据

### Day 36 · 安全入门（为 P5 铺垫）
- **学**：Prompt Injection、间接注入（来自网页/文档）、越权工具调用、数据外泄
- **做**：对 agent 做红队测试，构造 10 个攻击用例，记录哪些成功
- **产出**：`security/redteam.md`
- **验收**：至少复现 3 种攻击，并给出初步缓解

### Day 37 · 生产级单 Agent 服务
- **做**：把 agent 包成 FastAPI 服务：流式 SSE 接口、会话管理、鉴权、错误码、健康检查
- **产出**：`api/` + Dockerfile
- **验收**：`docker run` 后能通过 curl 完整对话

### Day 38 · 里程碑 M3
- **做**：整合为**生产级单 Agent 服务**，含 tracing、评测、限流、缓存、Docker
- **产出**：`m3-production-agent` tag
- **验收**：写一篇技术文档说明架构与取舍；能回答"如果 QPS 涨 10 倍怎么办"

---

## P4 · RAG 与检索系统（Day 39–50）

### Day 39 · 向量与 Embedding
- **学**：embedding 原理、余弦相似度 vs 点积、维度与归一化、模型选型（中文场景重点）
- **做**：用 2 个 embedding 模型对同一批中文语料做检索对比（召回率）
- **产出**：`rag/embed_compare.md`
- **验收**：能解释"为什么中文场景不能随便用英文模型"

### Day 40 · 切分与索引
- **学**：chunking 策略（定长/递归/语义/父子文档）、overlap、元数据设计
- **做**：同一文档用 3 种切分方式建索引，对比问答质量
- **产出**：`rag/chunking.py`
- **验收**：能给出针对不同文档类型的切分建议

### Day 41 · 向量数据库
- **学**：HNSW/IVF 索引原理、pgvector vs Qdrant vs Milvus vs Chroma 取舍
- **做**：用 pgvector + Qdrant 各建一套索引，对比写入/查询性能
- **产出**：`rag/vectordb.md` + 两套可运行 demo
- **验收**：能说明选型依据（数据量、运维成本、过滤需求）

### Day 42 · 检索：混合与重排
- **学**：BM25、稠密+稀疏混合（RRF）、cross-encoder rerank、查询改写（HyDE、multi-query）
- **做**：实现 hybrid + rerank 管线，用同一评测集对比纯向量
- **产出**：`rag/retriever.py` + 指标对比表
- **验收**：Recall@10 相比纯向量有可量化提升

### Day 43 · RAG 评测
- **学**：检索指标（Recall/Precision/MRR/NDCG）、生成指标（faithfulness/answer relevancy）、RAGAS
- **做**：建 50 条 QA 评测集（含答案与相关文档标注），跑 RAGAS
- **产出**：`rag/eval_ragas.py`
- **验收**：能区分"检索错"还是"生成错"导致的失败

### Day 44 · 高级 RAG
- **学**：句子窗口、上下文压缩、GraphRAG、RAG-Fusion、自适应检索（该不该检索）
- **做**：实现上下文压缩 + 自适应路由（简单问题不检索）
- **产出**：`rag/advanced.py`
- **验收**：简单问题延迟下降且质量不掉

### Day 45 · Agentic RAG
- **学**：把检索当作工具、多轮检索、检索结果反思与补检、多源（SQL/API/文档）路由
- **做**：实现"文档 + SQL + 网页"三源路由的 agentic RAG
- **产出**：`rag/agentic.py`
- **验收**：能自动选择正确数据源并给出引用

### Day 46 · 引用与可溯源
- **学**：citation 生成、span 定位、幻觉检测、置信度表达
- **做**：输出答案时附带精确到段落的引用，并做引用正确性抽检
- **产出**：引用正确率报告
- **验收**：引用错误率 < 10%

### Day 47 · 多模态检索
- **学**：图像/PDF 版面解析、图表理解、多模态 embedding、ColPali 类方案
- **做**：对含表格和图表的 PDF 做问答（含版面信息）
- **产出**：`rag/multimodal.py`
- **验收**：表格类问题准确率明显优于纯文本切分

### Day 48 · 增量更新与数据管道
- **学**：增量索引、去重、删除同步、文档版本与时效性
- **做**：实现"文档变更 → 增量重建索引"管道
- **产出**：`rag/pipeline.py`
- **验收**：改一个文档不会全量重跑

### Day 49 · 企业场景调优
- **学**：权限过滤（按用户/部门过滤检索结果）、审计、私有化部署
- **做**：加行级权限过滤（向量库 filter），验证越权不可见
- **产出**：权限测试用例
- **验收**：A 用户检索不到 B 用户的私有文档

### Day 50 · 里程碑 M4
- **做**：交付**文档问答系统**：多格式摄入 → hybrid 检索 → rerank → 引用 → 评测报告 → API
- **产出**：`m4-rag-system` tag + README 含评测数据
- **验收**：50 条评测集上 faithfulness > 0.8；能演示"为什么这个答案可信"

---

## P5 · 多 Agent、多模态与安全（Day 51–62）

### Day 51 · 多 Agent 模式
- **学**：Supervisor、Hierarchical、Network、Swarm、Debate 五种拓扑的适用场景
- **做**：画 5 种拓扑图，标注通信成本与失败模式
- **产出**：`notes/day-51.md`
- **验收**：能回答"什么时候**不该**用多 Agent"（多数情况下单 Agent + 好工具更强）

### Day 52 · Supervisor 实现
- **学**：路由、委派、结果聚合、子 agent 上下文隔离
- **做**：实现 supervisor + 3 个专家 agent（研究员/分析师/写手）
- **产出**：`multiagent/supervisor.py`
- **验收**：端到端完成"调研→分析→成文"

### Day 53 · 通信与共享状态
- **学**：消息传递 vs 共享黑板（blackboard）、handoff 机制、状态冲突解决
- **做**：实现共享黑板 + 结构化消息协议，加冲突检测
- **产出**：`multiagent/blackboard.py`
- **验收**：两个 agent 同时写同一字段时行为可预测

### Day 54 · 多 Agent 成本与可靠性
- **学**：token 放大问题、死循环/乒乓对话检测、超时与预算控制、部分失败降级
- **做**：给多 agent 系统加全局预算与循环检测，对比单 agent 的成本/质量
- **产出**：对比实验报告
- **验收**：能给出"该场景用单 agent 还是多 agent"的数据支撑

### Day 55 · 语音 Agent
- **学**：ASR→LLM→TTS 管线、实时语音 API、打断（barge-in）、延迟预算
- **做**：做一个能语音对话的 agent（可先用浏览器 WebRTC）
- **产出**：`voice/`
- **验收**：端到端延迟 < 1.5s，能被打断

### Day 56 · 计算机使用（Computer Use / Browser Use）
- **学**：视觉定位 vs DOM 定位、动作空间设计、沙箱与失败恢复
- **做**：用 Playwright + LLM 实现"打开网页→提取信息→填表"的自动化
- **产出**：`browser_agent/`
- **验收**：在 3 个不同网站上完成任务，失败有截图诊断

### Day 57 · 代码执行与沙箱
- **学**：代码解释器模式、沙箱隔离（Docker/gVisor/子进程）、资源限制、超时
- **做**：实现"LLM 生成代码 → 沙箱执行 → 结果回填"的数据分析 agent
- **产出**：`code_agent/`
- **验收**：恶意代码（`rm -rf`/死循环/大内存）被安全拦截

### Day 58 · Prompt Injection 深度防御
- **学**：直接/间接注入、工具输出污染、数据外泄通道、防御分层（输入过滤/权限最小化/输出审查/人工确认）
- **做**：给 agent 加三层防御，用 Day36 的攻击用例复测
- **产出**：防御前后对比表
- **验收**：攻击成功率显著下降（并诚实记录仍能突破的用例）

### Day 59 · 权限、审计与合规
- **学**：最小权限、工具级 ACL、操作审计日志、PII 处理、数据留存
- **做**：加审计日志（谁、何时、调了什么工具、参数、结果摘要）+ PII 脱敏
- **产出**：`security/audit.py`
- **验收**：能完整还原一次敏感操作链路

### Day 60 · 护栏（Guardrails）
- **学**：输入/输出护栏、话题边界、格式强制、毒性检测、失败时的降级话术
- **做**：接护栏层（规则 + 小模型分类器），测试绕过难度
- **产出**：`security/guardrails.py`
- **验收**：越界请求被拒且拒绝话术自然

### Day 61 · 多 Agent + 安全整合
- **做**：给 Day52 的多 agent 系统全面加安全层（护栏、审计、权限、预算）
- **产出**：`multiagent/secure/`
- **验收**：安全层不显著损害任务成功率（对比数据）

### Day 62 · 里程碑 M5
- **做**：交付**多 Agent 协作系统**（含语音或浏览器能力之一）+ 完整安全层
- **产出**：`m5-multiagent` tag + 架构文档 + 红队报告
- **验收**：能演示端到端任务，并讲清"攻击面在哪、怎么防"

---

## P6 · 生产化与部署（Day 63–72）

### Day 63 · 服务架构
- **学**：无状态服务 + 外部状态、队列与 worker、长任务异步化、幂等
- **做**：把长任务 agent 改成"提交任务→返回 job_id→轮询/推送结果"
- **产出**：`api/async_jobs.py`
- **验收**：任务提交后服务重启不丢任务

### Day 64 · 容器化与本地编排
- **学**：多阶段构建、镜像瘦身、docker-compose 编排（api + db + 向量库 + redis）
- **做**：`docker compose up` 一键起全栈
- **产出**：`docker-compose.yml`
- **验收**：新机器上一条命令跑通全流程

### Day 65 · 云部署
- **学**：Serverless vs 容器、冷启动、区域与合规、密钥管理（不要硬编码 key）
- **做**：部署到真实云（阿里云/腾讯云/AWS/GCP 任选），配域名 + HTTPS
- **产出**：可公开访问的 URL
- **验收**：外部网络可访问且密钥不泄露

### Day 66 · CI/CD
- **学**：GitHub Actions、测试门禁、评测门禁、自动部署、回滚
- **做**：PR 触发 lint+test+eval，合并后自动部署
- **产出**：`.github/workflows/`
- **验收**：一次提交全自动到线上

### Day 67 · 数据库与迁移
- **学**：Postgres、连接池、Alembic 迁移、索引与慢查询、pgvector 集成
- **做**：把 SQLite 全面迁到 Postgres + Alembic 版本化
- **产出**：`migrations/`
- **验收**：能从空库一键迁移到最新版本

### Day 68 · 监控与告警
- **学**：指标（QPS/延迟/token 消耗/失败率）、日志聚合、告警阈值、on-call 思路
- **做**：接 Prometheus + Grafana，配 3 条关键告警
- **产出**：监控面板
- **验收**：人为注入故障时能触发告警

### Day 69 · 成本治理
- **学**：成本归因（按用户/会话/功能）、预算硬限、模型降级策略、缓存收益核算
- **做**：加成本看板 + 单用户预算上限（超限降级到小模型或拒绝）
- **产出**：成本报告
- **验收**：能说出"每 1000 次对话成本多少、优化空间在哪"

### Day 70 · 性能压测与调优
- **学**：负载测试（locust/k6）、瓶颈定位、并发与限流、超时级联
- **做**：压测到瓶颈，优化 2 处（如连接池、并发控制），再压
- **产出**：压测前后对比
- **验收**：P95 延迟有明确数字改善

### Day 71 · 上线演练与故障预案
- **学**：灰度发布、feature flag、降级预案（模型不可用时怎么办）、数据备份
- **做**：写 runbook：3 类故障的处置流程
- **产出**：`docs/runbook.md`
- **验收**：能按 runbook 独立处置一次模拟故障

### Day 72 · 里程碑 M6
- **做**：系统真正上线，含 CI/CD、监控、告警、成本看板、runbook
- **产出**：`m6-live` tag + 线上地址
- **验收**：连续 7 天稳定运行无 P0 故障

---

## P7 · 综合项目与求职（Day 73–84）

### Day 73 · 选题
- **学**：什么项目算"有说服力"（真实用户、真实数据、可量化效果、有取舍叙事）
- **做**：定 1 个综合项目，写 1 页设计文档（问题、用户、方案、评测指标、风险）
- **产出**：`docs/project-design.md`
- **验收**：设计文档能被他人读懂并指出漏洞

### Day 74–76 · 核心功能开发
- **做**：搭骨架 + 打通主链路（可复用 M2/M3/M4/M5 的组件，**不要从零重写**）
- **产出**：可运行 MVP
- **验收**：Day76 结束时主流程端到端可跑

### Day 77–78 · 质量打磨
- **做**：评测集 + 失败案例分析 + 针对性优化；补测试与文档
- **产出**：评测报告（前后对比）
- **验收**：关键指标有可量化提升

### Day 79–80 · 交付与部署
- **做**：部署上线 + 写 README（含架构图、截图/GIF、快速开始、评测数据）
- **产出**：公开仓库
- **验收**：陌生人 5 分钟内能跑起来

### Day 81 · 技术写作
- **做**：写一篇深度博客：讲**一个具体坑**及如何解决（比泛泛综述有价值 10 倍）
- **产出**：发布到掘金/知乎/Medium/个人站
- **验收**：有真实读者反馈

### Day 82 · 简历与作品集
- **做**：简历改写为"问题-方案-指标"格式；整理 GitHub（置顶 3 个仓库、每个都有好 README）
- **产出**：简历 + 作品集页
- **验收**：每条经历都有数字

### Day 83 · 面试准备
- **学**：高频题——ReAct vs Plan-Execute、如何防注入、如何评测、如何控成本、如何做 HITL、上下文超限怎么办、幻觉怎么处理
- **做**：对着镜子/录音讲 5 个项目难点，每个 3 分钟
- **产出**：`docs/interview-qa.md`（20 题自答）
- **验收**：能不看稿讲清架构取舍

### Day 84 · 总复盘与下一阶段
- **做**：写 84 天总结：能力盘点、最薄弱的 2 块、下一步（如 RL 微调、Agent 评测专精、垂直领域深耕）
- **产出**：`docs/retrospective.md`
- **验收**：明确下一个 30 天的 3 个目标

---

## 附录 A · 核心资料清单（按阶段）

| 阶段 | 资料 |
|---|---|
| P0 | Python 官方文档、pydantic v2 文档、uv 文档、*Fluent Python* 选读 |
| P1 | OpenAI/Anthropic Prompt Engineering 指南、`tiktoken`、promptfoo 文档、Anthropic《Effective context engineering for AI agents》 |
| P2 | ReAct 论文、Reflexion 论文、ReWOO、Plan-and-Solve、Anthropic《Building effective agents》 |
| P3 | LangGraph 官方教程、MCP 规范、Langfuse/Phoenix 文档、OpenAI Agents SDK |
| P4 | RAGAS 文档、pgvector/Qdrant 文档、*Lost in the Middle* 论文、ColPali、Anthropic Contextual Retrieval |
| P5 | OWASP LLM Top 10、Anthropic/OpenAI 的 agent 安全文章、Playwright 文档 |
| P6 | 12-Factor App、FastAPI 文档、GitHub Actions 文档、Prometheus 文档 |
| P7 | 无固定资料，产出为主 |

## 附录 B · 常见误区（省时间用）

1. **只学框架不学原理**：LangChain 版本一变，只会调 API 的人直接归零。
2. **一上来就多 Agent**：绝大多数任务单 Agent + 好工具更便宜更准。
3. **没有评测就调 prompt**：凭感觉优化 = 随机游走。先建评测集。
4. **把 RAG 当"向量库 + 拼接"**：不做 rerank、不做评测、不做权限，一上生产就崩。
5. **忽略成本与延迟**：demo 阶段无所谓，上线后单次请求 0.5 元就没人用。
6. **上下文无限堆**：堆得越多越差，且越贵。要压缩、要隔离、要按需检索。
7. **忽略安全**：间接注入（网页/文档里的恶意指令）是最容易被忽略的真实风险。
8. **不写 runbook**：模型 API 挂了、限流了怎么办？没预案就是事故。

## 附录 C · 每周产出检查表

| 周 | 天数 | 必须交付 |
|---|---|---|
| 1 | 1–7 | 项目脚手架 + 流式 CLI |
| 2 | 8–14 | 结构化输出 + 工具调用 + M1 CLI Agent |
| 3 | 15–21 | 手写 Agent 循环 + 记忆 + 规划 |
| 4 | 22–26 | HITL + 持久化 + M2 迷你框架 |
| 5 | 27–33 | LangGraph + MCP + Tracing + 评测 |
| 6 | 34–38 | 缓存/限流 + 安全入门 + M3 生产服务 |
| 7 | 39–45 | Embedding→混合检索→RAGAS→Agentic RAG |
| 8 | 46–50 | 引用 + 多模态 + 权限 + M4 RAG 系统 |
| 9 | 51–57 | 多 Agent 三连 + 语音/浏览器/代码沙箱 |
| 10 | 58–62 | 安全四连 + M5 多 Agent 系统 |
| 11 | 63–72 | 容器/云/CI-CD/监控/成本 + M6 上线 |
| 12 | 73–84 | 综合项目 + 博客 + 简历 + 面试 + 复盘 |

## 附录 D · 进度追踪表

复制到 `progress.md`，每天勾选。

```
[ ] D1  [ ] D2  [ ] D3  [ ] D4  [ ] D5   | M0
[ ] D6  [ ] D7  [ ] D8  [ ] D9  [ ] D10  [ ] D11  [ ] D12  [ ] D13  [ ] D14 | M1
[ ] D15 [ ] D16 [ ] D17 [ ] D18 [ ] D19  [ ] D20  [ ] D21  [ ] D22  [ ] D23  [ ] D24 [ ] D25 [ ] D26 | M2
[ ] D27 [ ] D28 [ ] D29 [ ] D30 [ ] D31  [ ] D32  [ ] D33  [ ] D34  [ ] D35  [ ] D36 [ ] D37 [ ] D38 | M3
[ ] D39 [ ] D40 [ ] D41 [ ] D42 [ ] D43  [ ] D44  [ ] D45  [ ] D46  [ ] D47  [ ] D48 [ ] D49 [ ] D50 | M4
[ ] D51 [ ] D52 [ ] D53 [ ] D54 [ ] D55  [ ] D56  [ ] D57  [ ] D58  [ ] D59  [ ] D60 [ ] D61 [ ] D62 | M5
[ ] D63 [ ] D64 [ ] D65 [ ] D66 [ ] D67  [ ] D68  [ ] D69  [ ] D70  [ ] D71  [ ] D72 | M6
[ ] D73 ... [ ] D84 | 作品集
```

---

**最后一句话**：这 84 天里，Day 16（手写 Agent 循环）和 Day 43（RAG 评测）是最容易被跳过、也最不该跳过的两天。前者决定你调 bug 是靠推理还是靠玄学，后者决定你优化方向是数据驱动还是凭感觉。
