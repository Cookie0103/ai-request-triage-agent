# Internal AI Request Triage Agent 项目选择与魔改说明

## 最终选择

主底座使用这个仓库：

- GitHub：`https://github.com/Zamil00/ai-operations-control-center`
- 已验证的本地源码：`/Users/ke.chen/.repo-source-cache-project-choice/Zamil00__ai-operations-control-center`
- 建议正式工作目录：`/Users/ke.chen/開発/practice/internal-ai-request-triage-agent`

选择它的原因：

- 已经有一个完整的内部工具雏形：请求创建、审核、批准/拒绝、审计日志、指标 dashboard。
- 后端是 FastAPI，前端是 React，和你的学习目标匹配。
- 代码量不大，适合你现在读懂、跑起来、逐步魔改。
- 业务方向不是 Todo / Blog / Weather，而是公司内部 AI 请求治理和 workflow 分诊。
- 缺少的功能刚好适合你练习：Auth、PostgreSQL、Alembic、LLM triage、React 页面拆分、后续 RAG。

## 不作为主底座的仓库

### 1. `VMK-004/AI-workflow-automation`

- GitHub：`https://github.com/VMK-004/AI-workflow-automation`
- 本地源码：`/Users/ke.chen/.repo-source-cache-project-choice/VMK-004__AI-workflow-automation`
- 结论：作为进阶参考，不作为第一主项目。

优点：

- 有 Auth。
- 有 PostgreSQL。
- 有 Alembic。
- 有 workflow、node、edge、run。
- 有执行引擎。
- 有 React + TypeScript 页面结构。

不推荐你现在直接用它的原因：

- 项目太大，容易变成“看不懂，只能照着跑”。
- 你现在更需要先完整做完一个中等复杂度项目。
- 它适合你基础版做完后，再参考它的 workflow execution 和 Auth 结构。

之后可以重点阅读：

```text
backend/app/api/routes/auth.py
backend/app/core/security.py
backend/app/core/dependencies.py
backend/app/services/execution_service.py
backend/app/services/graph_service.py
backend/app/node_handlers/llm_call.py
frontend/src/pages/WorkflowEditorPage.tsx
frontend/src/store/workflowStore.ts
```

### 2. `nehalvaghasiya/agentic-rag`

- GitHub：`https://github.com/nehalvaghasiya/agentic-rag`
- 本地源码：`/Users/ke.chen/.repo-source-cache-project-choice/nehalvaghasiya__agentic-rag`
- 结论：作为 RAG 扩展参考。

优点：

- 有知识库创建。
- 有文件上传。
- 有 chunking / embedding。
- 有 query rewrite。
- 有 retrieval grading。
- 有 SSE streaming。
- 有带引用来源的 React chat UI。

不作为主项目的原因：

- 主体还是知识库问答。
- 业务 workflow、状态流转、人工 review、审计日志比较弱。
- 容易变成普通 RAG demo。

### 3. `deontsanchez/rag-system-poc`

- GitHub：`https://github.com/deontsanchez/rag-system-poc`
- 本地源码：`/Users/ke.chen/.repo-source-cache-project-choice/deontsanchez__rag-system-poc`
- 结论：作为简单 RAG 小参考。

优点：

- 比 `agentic-rag` 小。
- 有文件上传、ChromaDB、React chat、FastAPI API。

不作为主项目的原因：

- 更像经典 RAG demo。
- 和 Business-oriented Productivity / Internal Workflow Tool 的目标不够贴合。

## 最终项目方向

项目名建议：

```text
Internal AI Request Triage Agent
```

中文理解：

```text
公司内部 AI / 数据 / 自动化需求分诊平台
```

一句话说明：

```text
一个 FastAPI + React 内部工具，用来收集公司内部 AI、数据分析、自动化相关请求，通过 LLM 自动分类和评估风险，再交给人工 reviewer 审核，并记录完整审计日志。
```

## 原仓库功能和你的魔改方向

原仓库现在做的是：

```text
AI request
-> pending review
-> approve / reject
-> dashboard metrics
-> audit log
```

你要魔改成：

```text
手动输入 / Notion 导入 / CSV 导入
-> AI triage 自动分类
-> human review 人工确认
-> 状态流转
-> owner 和 next action 跟踪
-> audit log
-> weekly digest
-> 后续接 RAG 和 Agent workflow
```

## 你应该保留的东西

从原仓库保留：

- FastAPI 项目结构。
- request CRUD。
- approve / reject 审核动作。
- audit log。
- metrics dashboard。
- seed demo data。
- React dashboard 的基本布局思路。

## 你应该新增或替换的东西

基础版新增：

- 用户注册 / 登录。
- JWT Auth。
- PostgreSQL。
- Alembic migration。
- request 状态生命周期。
- AI triage 字段。
- mock triage service。
- 可选真实 LLM triage。
- React 页面拆分。
- review queue。
- 更完整 README。

进阶版新增：

- Notion source adapter。
- RAG context。
- weekly digest generator。
- follow-up recommendation agent。
- 轻量 workflow execution。

## 产品用户故事

作为业务 / 数据 / 内部运营成员，我可以提交一个 AI、数据分析或自动化需求。系统会自动判断这个请求属于什么类型、风险高不高、优先级如何、应该由谁处理、下一步应该做什么。Reviewer 可以确认、修改、批准或拒绝这个请求。所有操作都会写入 audit log，方便之后追踪。

## 核心对象设计

### Request

表示一个内部请求。

字段：

- `title`
- `description`
- `business_goal`
- `source_type`
- `source_url`
- `requester`
- `owner`
- `category`
- `priority`
- `risk_level`
- `status`
- `ai_summary`
- `ai_suggested_owner`
- `ai_next_action`
- `review_note`

### Audit Log

表示一次重要操作记录。

字段：

- `actor`
- `action`
- `request_id`
- `previous_status`
- `new_status`
- `details`
- `created_at`

### Triage Result

表示 AI 分诊结果。

字段：

- `category`
- `priority`
- `risk_level`
- `suggested_owner`
- `next_action`
- `reasoning`
- `confidence`

## 状态生命周期

基础版使用这个状态流：

```text
new
-> triaged
-> pending_review
-> approved
-> in_progress
-> done
```

拒绝路径：

```text
pending_review
-> rejected
```

导入但不采用的项目：

```text
ignored
```

## 分类

第一版用这些分类：

- `AI application`
- `Data analysis`
- `Workflow automation`
- `Knowledge management`
- `Reporting`
- `Other`

## 优先级

第一版用：

- `low`
- `medium`
- `high`
- `urgent`

## 风险等级

第一版用：

- `low`
- `medium`
- `high`

高风险通常表示：

- 涉及客户数据。
- 涉及个人信息。
- 会影响外部客户。
- 会自动写入外部系统。
- 涉及法律、财务、权限、安全。

## 推荐技术栈

后端：

- Python 3.11+
- FastAPI
- SQLAlchemy 2.x
- PostgreSQL
- Alembic
- Pydantic
- JWT
- pytest

前端：

- React
- Vite
- TypeScript 推荐，但如果卡住可以先用 JavaScript
- React Router
- fetch 或 Axios
- 普通 CSS / Tailwind 都可以

AI：

- 第一版先做 mock triage。
- 然后接 OpenAI 或兼容 LLM API。
- 之后再加 RAG。

## 基础版完成标准

基础版完成时，应该做到：

- 用户可以注册和登录。
- 用户可以创建 request。
- request 存在 PostgreSQL。
- 后端可以对 request 执行 triage。
- reviewer 可以确认、编辑、批准或拒绝。
- request 有清晰状态流转。
- dashboard 显示请求数量、待审核数量、批准数量、风险分布、owner 分布。
- audit log 记录 create、triage、review、status change。
- README 说明业务问题、架构、启动方式、API、截图和后续 Agent/RAG roadmap。

## 基础版不要做的东西

先不要做：

- 拖拽 workflow builder。
- 复杂多 Agent planner。
- 自动发 Slack。
- Notion 双向同步。
- Celery / Redis 后台任务。
- 企业级 RBAC。
- 复杂图表。
- Kubernetes 部署。

这些都可以作为进阶，但不应该挡住第一个 MVP。

