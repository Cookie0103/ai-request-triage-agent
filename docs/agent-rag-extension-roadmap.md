# Agent / RAG 进阶路线图

这个文档只在基础版完成后再看。

基础版应该已经完成：

- Auth
- Request CRUD
- PostgreSQL
- AI triage
- Human review
- Audit log
- Dashboard

在基础版跑通之前，不要开始做这里的内容。

## 进阶 1：Notion Source Adapter

目标：

把 Notion 页面或数据库中的内容导入系统，作为 request candidate。

适合你的真实数据：

- Datamind `AI Weekly`
- Datamind `Cube定例議事録データベース`
- 其他业务会议记录、项目记录、行动项页面

第一版不要做复杂同步，只做手动导入：

```text
用户粘贴 Notion 页面 URL
-> 后端抓取或接收页面内容
-> LLM 抽取 request candidates
-> 用户确认哪些 candidate 变成正式 request
```

建议新增表：

```text
source_pages
import_runs
request_candidates
```

建议新增 API：

```text
POST /imports/notion
GET /imports/runs
GET /request-candidates
POST /request-candidates/{id}/promote
POST /request-candidates/{id}/ignore
```

portfolio 价值：

- 数据来源更真实。
- 项目不再像纯 demo。
- 仍然保留人工确认，不会变成不可信的自动化。

## 进阶 2：RAG Context

目标：

让 triage 不只看 request 本身，还能参考内部文档、SOP、历史请求和知识库。

主要参考仓库：

- GitHub：`https://github.com/nehalvaghasiya/agentic-rag`
- 本地源码：`/Users/ke.chen/.repo-source-cache-project-choice/nehalvaghasiya__agentic-rag`

重点读这些文件：

```text
backend/routes.py
backend/indexing/loader.py
backend/indexing/chunker.py
backend/indexing/embedder.py
backend/storage/kb.py
backend/storage/vector.py
backend/querying/nodes.py
backend/querying/stream.py
frontend/src/components/features/knowledge-base/CreateKnowledgeBaseModal.jsx
frontend/src/components/features/chat/SourceList.jsx
```

先做小版本：

```text
上传 SOP / policy / 项目说明文档
-> chunk
-> embedding
-> triage 时检索 top-k 相关片段
-> 把片段放进 LLM prompt
-> 保存引用片段，供 reviewer 查看
```

建议新增表：

```text
knowledge_documents
knowledge_chunks
triage_sources
```

建议新增 API：

```text
POST /knowledge/documents
GET /knowledge/documents
POST /knowledge/search
```

portfolio 价值：

- 从“单次 LLM 分类”升级为“有依据的决策辅助”。
- 可以讲清楚 RAG、chunking、embedding、retrieval、citation。

## 进阶 3：Weekly Digest Generator

目标：

根据系统里的 request、risk、pending review、in progress 项目，生成一份可以发到 Slack 的 weekly digest。

流程：

```text
读取 approved / in_progress / high-risk requests
-> 按 category 和 owner 分组
-> 总结进展和 blocker
-> 生成 Slack-ready digest
-> 保存 digest 版本
```

建议新增表：

```text
weekly_digests
```

建议新增 API：

```text
POST /digests/weekly
GET /digests
GET /digests/{id}
```

Digest 格式：

```markdown
## Weekly AI Ops Digest

### Key Requests
- ...

### Pending Review
- ...

### Risks
- ...

### Recommended Follow-ups
- ...
```

portfolio 价值：

- 把内部系统数据转成业务沟通产物。
- 很好 demo。
- 之后可以自然接 Slack。

## 进阶 4：Follow-up Recommendation Agent

目标：

针对卡住的、高风险的、长时间没人处理的 request，自动给出下一步建议。

Agent 可以拥有这些工具：

```text
get_request
list_related_requests
search_knowledge_base
generate_follow_up_message
```

最小 agent loop：

```text
输入 request_id
1. 读取 request。
2. 检查 status、risk、owner、创建时间。
3. 检索相关知识库。
4. 生成 recommended next action。
5. 保存 recommendation。
```

建议新增表：

```text
agent_recommendations
```

建议新增 API：

```text
POST /agent/recommendations/{request_id}
GET /agent/recommendations
```

portfolio 价值：

- 开始进入 tool use。
- 不需要一开始做复杂多 Agent。
- 可以讲 planning、memory、recommendation。

## 进阶 5：轻量 Workflow Execution

目标：

对 approved request 执行一些固定 workflow template。

参考仓库：

- GitHub：`https://github.com/VMK-004/AI-workflow-automation`
- 本地源码：`/Users/ke.chen/.repo-source-cache-project-choice/VMK-004__AI-workflow-automation`

重点读这些文件：

```text
backend/app/services/execution_service.py
backend/app/services/graph_service.py
backend/app/node_handlers/base.py
backend/app/node_handlers/llm_call.py
backend/app/node_handlers/http_request.py
backend/app/api/routes/execution.py
frontend/src/pages/WorkflowEditorPage.tsx
frontend/src/components/editor/Canvas.tsx
frontend/src/components/editor/NodeSidebar.tsx
```

不要一开始复制完整拖拽编辑器。

先做固定模板：

```text
Template: Triage Digest
Input request -> LLM summary -> DB save

Template: Knowledge Lookup
Input question -> RAG search -> answer with sources

Template: Follow-up Draft
Input request -> retrieve context -> draft Slack message
```

建议新增表：

```text
workflow_templates
workflow_runs
workflow_run_steps
```

portfolio 价值：

- 从 CRUD app 升级到 LLM workflow platform。
- 为后面 Agent / tool use 课程做准备。

## 推荐学习顺序

1. 完成基础版。
2. 加 Notion Source Adapter。
3. 加 Weekly Digest Generator。
4. 加 RAG Context。
5. 加 Follow-up Recommendation Agent。
6. 加 Lightweight Workflow Execution。

## 暂时不要做

基础版完成前，不要碰：

- 完整拖拽 workflow builder
- 自动写 Slack / Notion
- 多租户权限系统
- Celery / Redis 后台任务
- Kubernetes
- 很复杂的向量数据库基础设施

## 最终包装故事

README 和简历可以这样写：

```text
Built an internal AI request triage platform that collects business automation requests, classifies them with an LLM-backed triage service, routes them through human review, records audit logs, and later extends into RAG-grounded recommendations and weekly operations digests.
```

基础版简历 bullet：

```text
Built a FastAPI + React internal request triage platform with PostgreSQL persistence, JWT authentication, request lifecycle management, human review workflow, AI-assisted categorization, and audit logging.
```

RAG 进阶后简历 bullet：

```text
Extended triage recommendations with RAG over internal documentation, adding document ingestion, chunking, vector retrieval, source-grounded reasoning, and saved evidence snippets for reviewer auditability.
```

