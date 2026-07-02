# Internal AI Request Triage Agent 实施计划

> **给之后执行任务的 agent / 开发者：** 如果用 agent 自动执行，请使用 `superpowers:subagent-driven-development` 或 `superpowers:executing-plans`。本计划按 checkbox 拆分，方便一步步完成。

**目标：** 基于 `Zamil00/ai-operations-control-center`，做一个 FastAPI + React 的内部 AI / 数据 / 自动化需求分诊平台。

**整体架构：** 先跑通原始项目，再逐步魔改。基础版保留 request、review、audit、dashboard 这些业务闭环；之后加入 Auth、PostgreSQL、AI triage、React 页面拆分；最后再考虑 Notion、RAG、Agent workflow。

**技术栈：** FastAPI、SQLAlchemy、PostgreSQL、Alembic、Pydantic、JWT、pytest、React、Vite、fetch/Axios。

## 全局约束

- 主底座仓库：`https://github.com/Zamil00/ai-operations-control-center`
- 工作目录：`/Users/ke.chen/開発/practice/internal-ai-request-triage-agent`
- 先完成基础版，再做 Agent/RAG。
- 第一版必须能本地跑起来。
- 第一版不要做拖拽 workflow editor。
- 第一版不要做复杂多 Agent。
- 第一版不要做 Slack/Notion 自动写入。
- 每完成一个小功能就 commit。

---

## 最终目录目标

不用一开始就改成这个结构，但最终可以整理成：

```text
internal-ai-request-triage-agent/
├── backend 或 app/
│   ├── api/
│   │   ├── routes_auth.py
│   │   ├── routes_requests.py
│   │   ├── routes_audit.py
│   │   ├── routes_metrics.py
│   │   └── routes_triage.py
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── dependencies.py
│   ├── models.py 或 models/
│   ├── schemas.py 或 schemas/
│   ├── services/
│   │   ├── audit.py
│   │   ├── triage.py
│   │   ├── auth.py
│   │   └── metrics.py
│   └── main.py
├── ui/
│   ├── src/
│   │   ├── api/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── types.ts
│   │   ├── App.tsx
│   │   └── main.tsx
│   └── package.json
├── tests/
├── alembic/
├── README.md
└── .env.example
```

如果原仓库结构不同，优先跟随原仓库，不要一开始大重构。

---

# Phase 0：准备和理解原项目

## Task 1：clone 并跑通原始项目

目标：确认原项目可以在本地跑起来，先不要改代码。

- [ ] 进入 practice 目录。

```bash
cd /Users/ke.chen/開発/practice
```

- [ ] clone 原仓库。

```bash
git clone https://github.com/Zamil00/ai-operations-control-center internal-ai-request-triage-agent
cd internal-ai-request-triage-agent
```

- [ ] 创建 Python 虚拟环境。

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

- [ ] 启动后端。

```bash
python -m uvicorn app.main:app --reload
```

预期：

```text
http://127.0.0.1:8000
http://127.0.0.1:8000/docs
```

- [ ] 启动前端。

```bash
cd /Users/ke.chen/開発/practice/internal-ai-request-triage-agent/ui
npm install
npm run dev
```

预期：

```text
http://127.0.0.1:5173
```

- [ ] 打开浏览器确认 dashboard 能显示。

- [ ] commit 原始 baseline。

```bash
cd /Users/ke.chen/開発/practice/internal-ai-request-triage-agent
git add .
git commit -m "chore: import ai operations control center baseline"
```

## Task 2：阅读原始项目结构

目标：先理解代码，不急着改。

- [ ] 阅读后端入口。

```text
app/main.py
```

重点看：

- FastAPI app 怎么创建。
- router 怎么注册。
- CORS 怎么配置。

- [ ] 阅读数据库和模型。

```text
app/database.py
app/models.py
app/schemas.py
```

重点看：

- SQLAlchemy model 怎么写。
- Pydantic schema 怎么写。
- SQLite 数据库怎么连接。

- [ ] 阅读 request API。

```text
app/api/routes_requests.py
```

重点看：

- `GET /requests`
- `POST /requests`
- `PATCH /requests/{id}`
- `POST /requests/{id}/approve`
- `POST /requests/{id}/reject`

- [ ] 阅读 audit 和 metrics。

```text
app/api/routes_audit.py
app/api/routes_metrics.py
app/services/audit.py
```

- [ ] 阅读前端主文件。

```text
ui/src/App.tsx
ui/src/types.ts
```

重点看：

- `useState`
- `useEffect`
- `fetch`
- 表单提交
- 列表渲染
- 条件渲染

- [ ] 写一个自己的学习笔记。

建议新建：

```text
docs/learning-notes.md
```

写清楚：

- 原项目有哪些 API。
- 数据库里有哪些表。
- 前端页面有哪些状态。
- 你准备怎么改。

---

# Phase 1：先把项目包装成你的方向

## Task 3：改项目名和 README

目标：从原来的 “AI Operations Control Center” 改成你的项目。

- [ ] 修改 `README.md` 顶部标题。

改成：

```markdown
# Internal AI Request Triage Agent

一个 FastAPI + React 内部工具，用来收集 AI、数据分析、自动化相关需求，通过 AI 辅助分诊和人工 review，形成可追踪的请求处理流程和 audit log。
```

- [ ] 修改 `ui/src/App.tsx` 里的页面标题。

把：

```tsx
AI Operations Control Center
```

改成：

```tsx
Internal AI Request Triage Agent
```

- [ ] 修改 sidebar / 页面说明文案。

建议文案：

```text
Internal platform for triaging AI, data, and workflow automation requests with human review and audit visibility.
```

- [ ] 跑前端 build。

```bash
cd /Users/ke.chen/開発/practice/internal-ai-request-triage-agent/ui
npm run build
```

- [ ] commit。

```bash
git add README.md ui/src/App.tsx
git commit -m "docs: rename project for internal request triage"
```

---

# Phase 2：扩展 request 数据模型

## Task 4：给 request 加业务字段

目标：让 request 不只是普通 AI 请求，而是内部业务自动化需求。

需要修改：

```text
app/models.py
app/schemas.py
app/services/seed.py
```

- [ ] 在 `RequestRecord` 加字段。

建议字段：

```python
source_type: Mapped[str] = mapped_column(String(30), default="manual")
source_url: Mapped[str | None] = mapped_column(Text, nullable=True)
requester: Mapped[str] = mapped_column(String(120), default="Unknown")
category: Mapped[str] = mapped_column(String(50), default="other")
ai_summary: Mapped[str | None] = mapped_column(Text, nullable=True)
ai_suggested_owner: Mapped[str | None] = mapped_column(String(120), nullable=True)
ai_next_action: Mapped[str | None] = mapped_column(Text, nullable=True)
ai_reasoning: Mapped[str | None] = mapped_column(Text, nullable=True)
ai_confidence: Mapped[float | None] = mapped_column(Float, nullable=True)
```

- [ ] 在 Pydantic schema 加对应字段。

建议加到 create/update/response 相关 schema：

```python
source_type: str = "manual"
source_url: str | None = None
requester: str = "Unknown"
category: str = "other"
ai_summary: str | None = None
ai_suggested_owner: str | None = None
ai_next_action: str | None = None
ai_reasoning: str | None = None
ai_confidence: float | None = None
```

- [ ] 修改 seed 数据，让它像真实内部需求。

示例：

```python
{
    "title": "Automate weekly DaaS status reporting",
    "business_goal": "Reduce manual reporting time by turning weekly updates into a structured digest.",
    "model": "gpt-4o-mini",
    "priority": "medium",
    "risk_level": "medium",
    "estimated_cost": 8.0,
    "owner": "DataMind Ops",
    "requester": "Business Team",
    "category": "reporting",
    "status": "new",
}
```

- [ ] 如果本地 SQLite schema 不匹配，删除本地生成的 sqlite db。

先查看路径：

```bash
rg "sqlite|DATABASE" app
```

确认后再删生成文件，例如：

```bash
rm app.db
```

不要删除源码。

- [ ] 启动后端，在 Swagger 看 `/requests` 是否出现新字段。

- [ ] commit。

```bash
git add app/models.py app/schemas.py app/services/seed.py
git commit -m "feat: extend request model for triage metadata"
```

## Task 5：定义 request 状态生命周期

目标：让项目有真实业务状态流转。

状态使用：

```text
new
triaged
pending_review
approved
rejected
in_progress
done
ignored
```

- [ ] 在 `app/schemas.py` 里定义：

```python
VALID_STATUSES = {
    "new",
    "triaged",
    "pending_review",
    "approved",
    "rejected",
    "in_progress",
    "done",
    "ignored",
}
```

- [ ] 修改 create request 默认状态。

把：

```python
status="pending"
```

改成：

```python
status="new"
```

- [ ] 增加状态修改 API。

在 `routes_requests.py` 中加：

```python
@router.post("/{request_id}/status", response_model=RequestResponse)
def change_request_status(
    request_id: int,
    payload: RequestUpdate,
    db: Session = Depends(get_db),
) -> RequestResponse:
    record = db.get(RequestRecord, request_id)
    if not record:
        raise HTTPException(status_code=404, detail="Request not found")
    if payload.status is None:
        raise HTTPException(status_code=400, detail="status is required")
    if payload.status not in VALID_STATUSES:
        raise HTTPException(status_code=400, detail="Invalid status")

    old_status = record.status
    record.status = payload.status
    record.updated_at = datetime.utcnow()
    db.add(record)
    db.commit()
    db.refresh(record)

    write_audit_log(
        db,
        actor=record.owner,
        action="status_changed",
        request_code=record.request_code,
        result="success",
        details=f"{old_status} -> {record.status}",
    )
    return record
```

- [ ] 在 Swagger 测试：

```text
POST /requests/{id}/status
```

body：

```json
{
  "status": "in_progress"
}
```

预期：

- request 状态改变。
- audit log 出现 `status_changed`。

- [ ] commit。

```bash
git add app/api/routes_requests.py app/schemas.py
git commit -m "feat: add explicit request status lifecycle"
```

---

# Phase 3：加入 AI triage

## Task 6：先做 mock triage，不接真实 LLM

目标：先把完整流程跑通，不被 API key 和模型调用卡住。

新增：

```text
app/services/triage.py
app/api/routes_triage.py
```

- [ ] 在 `app/schemas.py` 加：

```python
class TriageSuggestion(BaseModel):
    category: str
    priority: str
    risk_level: str
    suggested_owner: str
    next_action: str
    reasoning: str
    confidence: float
```

- [ ] 创建 `app/services/triage.py`。

```python
from app.models import RequestRecord
from app.schemas import TriageSuggestion


def triage_request(record: RequestRecord) -> TriageSuggestion:
    text = f"{record.title} {record.business_goal}".lower()

    if "report" in text or "weekly" in text or "dashboard" in text:
        category = "reporting"
        owner = "DataMind Ops"
    elif "notion" in text or "knowledge" in text or "document" in text:
        category = "knowledge management"
        owner = "Knowledge Ops"
    elif "workflow" in text or "automation" in text or "manual" in text:
        category = "workflow automation"
        owner = "Automation Team"
    elif "analysis" in text or "data" in text or "metric" in text:
        category = "data analysis"
        owner = "Data Analyst"
    else:
        category = "other"
        owner = "Platform Team"

    risk = "high" if "customer" in text or "personal" in text or "external" in text else "medium"
    priority = "high" if "urgent" in text or "deadline" in text or "blocked" in text else "medium"

    return TriageSuggestion(
        category=category,
        priority=priority,
        risk_level=risk,
        suggested_owner=owner,
        next_action="Review the request scope, confirm data access, and decide whether this should become an approved workflow.",
        reasoning="Deterministic mock triage based on keywords in the title and business goal.",
        confidence=0.62,
    )
```

- [ ] 创建 `app/api/routes_triage.py`。

```python
from datetime import datetime
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session

from app.database import get_db
from app.models import RequestRecord
from app.schemas import RequestResponse
from app.services.audit import write_audit_log
from app.services.triage import triage_request

router = APIRouter(prefix="/triage", tags=["triage"])


@router.post("/requests/{request_id}", response_model=RequestResponse)
def run_triage(request_id: int, db: Session = Depends(get_db)) -> RequestResponse:
    record = db.get(RequestRecord, request_id)
    if not record:
        raise HTTPException(status_code=404, detail="Request not found")

    suggestion = triage_request(record)
    record.category = suggestion.category
    record.priority = suggestion.priority
    record.risk_level = suggestion.risk_level
    record.ai_suggested_owner = suggestion.suggested_owner
    record.ai_next_action = suggestion.next_action
    record.ai_reasoning = suggestion.reasoning
    record.ai_confidence = suggestion.confidence
    record.status = "pending_review"
    record.updated_at = datetime.utcnow()

    db.add(record)
    db.commit()
    db.refresh(record)

    write_audit_log(
        db,
        actor="system",
        action="triage_completed",
        request_code=record.request_code,
        result="success",
        details=f"category={record.category}, priority={record.priority}, risk={record.risk_level}",
    )
    return record
```

- [ ] 在 `app/main.py` 注册 router。

```python
from app.api import routes_triage

app.include_router(routes_triage.router)
```

- [ ] 在 Swagger 测试：

```text
POST /triage/requests/1
```

预期：

- status 变成 `pending_review`。
- AI 字段被填充。
- audit log 出现 `triage_completed`。

- [ ] commit。

```bash
git add app/services/triage.py app/api/routes_triage.py app/main.py app/schemas.py
git commit -m "feat: add mock ai triage service"
```

## Task 7：再加真实 LLM triage 开关

目标：没有 API key 也能跑，有 API key 时可以调用真实 LLM。

- [ ] 在 `requirements.txt` 加：

```text
openai>=1.0.0
```

- [ ] 在 `.env.example` 加：

```text
USE_LLM_TRIAGE=false
OPENAI_API_KEY=
OPENAI_MODEL=gpt-4o-mini
```

- [ ] 在 `app/config.py` 加对应设置。

按原项目的 config 风格加：

```python
USE_LLM_TRIAGE: bool = False
OPENAI_API_KEY: str | None = None
OPENAI_MODEL: str = "gpt-4o-mini"
```

- [ ] 在 `triage.py` 加 prompt builder。

```python
def build_triage_prompt(record: RequestRecord) -> str:
    return f"""
You are an internal operations reviewer.
Classify this business request into JSON.

Allowed categories:
- AI application
- Data analysis
- Workflow automation
- Knowledge management
- Reporting
- Other

Allowed priorities: low, medium, high, urgent.
Allowed risk levels: low, medium, high.

Request title: {record.title}
Business goal: {record.business_goal}
Requester: {record.requester}
Source type: {record.source_type}

Return JSON with:
category, priority, risk_level, suggested_owner, next_action, reasoning, confidence.
"""
```

- [ ] 实现逻辑：

```text
如果 USE_LLM_TRIAGE=false：用 mock triage。
如果 USE_LLM_TRIAGE=true 且 OPENAI_API_KEY 有值：调用 LLM。
如果 LLM 调用失败：回退到 mock triage，并在 reasoning 里说明 fallback。
```

- [ ] 不设置 API key 测试 fallback。

```bash
python -m uvicorn app.main:app --reload
```

调用：

```text
POST /triage/requests/1
```

预期：仍然能成功。

- [ ] commit。

```bash
git add requirements.txt .env.example app/config.py app/services/triage.py
git commit -m "feat: add optional llm triage mode"
```

---

# Phase 4：把 React 从单文件改成中等复杂度

## Task 8：拆分 React 组件

目标：从一个大 `App.tsx` 拆成多个组件，练 component、props、state、fetch。

新增目录：

```text
ui/src/components/
```

建议组件：

```text
MetricCards.tsx
RequestForm.tsx
RequestList.tsx
RequestDetail.tsx
ReviewPanel.tsx
AuditLogTable.tsx
```

- [ ] 创建 `MetricCards.tsx`。

负责 dashboard KPI。

props：

```tsx
type MetricCardsProps = {
  total: number;
  pending: number;
  approved: number;
  rejected: number;
  highRisk: number;
};
```

- [ ] 创建 `RequestForm.tsx`。

负责创建 request 表单。

props：

```tsx
type RequestFormProps = {
  form: CreateRequestPayload;
  formErrors: Record<string, string | undefined>;
  submitting: boolean;
  onChange: (next: CreateRequestPayload) => void;
  onSubmit: () => void;
};
```

- [ ] 创建 `RequestList.tsx`。

负责 request 列表、搜索、筛选、选中。

props：

```tsx
type RequestListProps = {
  requests: RequestRecord[];
  selectedRequestId?: number;
  searchTerm: string;
  statusFilter: string;
  onSearchChange: (value: string) => void;
  onStatusFilterChange: (value: string) => void;
  onSelect: (request: RequestRecord) => void;
};
```

- [ ] 创建 `RequestDetail.tsx`。

显示：

- title
- business goal
- category
- priority
- risk
- owner
- status
- AI suggestion

- [ ] 创建 `ReviewPanel.tsx`。

负责：

- run triage
- approve
- reject
- review note

- [ ] 创建 `AuditLogTable.tsx`。

负责审计日志表格。

- [ ] 每拆一个组件就跑一次：

```bash
cd /Users/ke.chen/開発/practice/internal-ai-request-triage-agent/ui
npm run build
```

- [ ] commit。

```bash
git add ui/src/App.tsx ui/src/components
git commit -m "refactor: split dashboard into react components"
```

## Task 9：加入 Review Queue UX

目标：前端可以一键执行 AI triage，然后人工 approve/reject。

- [ ] 在 `App.tsx` 加 API 调用：

```tsx
async function runTriage(requestId: number) {
  setError('');
  setSuccessMessage('');
  try {
    const response = await fetch(`${API_BASE}/triage/requests/${requestId}`, {
      method: 'POST',
    });
    if (!response.ok) {
      throw new Error('Failed to run triage.');
    }
    setSuccessMessage('AI triage completed. Review the suggestion before approval.');
    await loadData();
  } catch (err) {
    setError(err instanceof Error ? err.message : 'Failed to run triage.');
  }
}
```

- [ ] 在 `ReviewPanel.tsx` 加按钮：

```tsx
<button onClick={() => onRunTriage(selectedRequest.id)}>
  Run AI Triage
</button>
```

- [ ] 显示这些 AI 字段：

```text
ai_suggested_owner
ai_next_action
ai_reasoning
ai_confidence
```

- [ ] 手动测试完整流程：

```text
Create request
-> select request
-> Run AI Triage
-> status becomes pending_review
-> approve or reject
-> audit log updates
```

- [ ] commit。

```bash
git add ui/src/App.tsx ui/src/components ui/src/types.ts
git commit -m "feat: add review queue for triage suggestions"
```

---

# Phase 5：加 Auth

## Task 10：实现注册 / 登录 / JWT

目标：让项目不再是裸 dashboard。

参考仓库：

```text
/Users/ke.chen/.repo-source-cache-project-choice/VMK-004__AI-workflow-automation/backend/app/api/routes/auth.py
/Users/ke.chen/.repo-source-cache-project-choice/VMK-004__AI-workflow-automation/backend/app/core/security.py
/Users/ke.chen/.repo-source-cache-project-choice/VMK-004__AI-workflow-automation/backend/app/core/dependencies.py
```

- [ ] 先读上面 3 个文件。

- [ ] 在 `requirements.txt` 加：

```text
python-jose[cryptography]
passlib[bcrypt]
python-multipart
```

- [ ] 新增 User model。

字段：

```text
id
email
username
hashed_password
created_at
```

- [ ] 新增 auth API。

```text
POST /auth/register
POST /auth/login
GET /auth/me
```

- [ ] 新增 password hashing。

需要函数：

```python
hash_password(password: str) -> str
verify_password(plain_password: str, hashed_password: str) -> bool
create_access_token(data: dict) -> str
```

- [ ] 保护写操作。

需要 token 的接口：

```text
POST /requests
PATCH /requests/{id}
POST /requests/{id}/approve
POST /requests/{id}/reject
POST /triage/requests/{id}
```

- [ ] 前端新增 login 页面。

需要练习：

- 表单输入。
- submit。
- fetch login。
- localStorage 保存 token。
- API 请求加 Authorization header。

保存 token：

```ts
localStorage.setItem('access_token', token.access_token);
```

请求 header：

```ts
Authorization: `Bearer ${localStorage.getItem('access_token')}`
```

- [ ] 手动测试：

```text
register
-> login
-> create request
-> run triage
-> approve
```

- [ ] commit。

```bash
git add app ui requirements.txt
git commit -m "feat: add jwt authentication"
```

---

# Phase 6：接 PostgreSQL 和 Alembic

## Task 11：从 SQLite 迁移到 PostgreSQL

目标：练真实项目数据库结构。

- [ ] 在 `requirements.txt` 加：

```text
alembic
psycopg2-binary
```

- [ ] 在 `.env.example` 加：

```text
DATABASE_URL=postgresql+psycopg2://postgres:postgres@localhost:5432/internal_ai_triage
```

- [ ] 创建本地数据库。

```bash
createdb internal_ai_triage
```

- [ ] 初始化 Alembic。

```bash
cd /Users/ke.chen/開発/practice/internal-ai-request-triage-agent
alembic init alembic
```

- [ ] 修改 `alembic/env.py`。

加入：

```python
from app.database import Base
target_metadata = Base.metadata
```

- [ ] 生成 migration。

```bash
alembic revision --autogenerate -m "create request triage tables"
```

- [ ] 执行 migration。

```bash
alembic upgrade head
```

- [ ] 启动后端，确认 CRUD 仍然工作。

```bash
python -m uvicorn app.main:app --reload
```

- [ ] commit。

```bash
git add app/database.py alembic alembic.ini .env.example requirements.txt
git commit -m "feat: migrate persistence to postgresql and alembic"
```

---

# Phase 7：补测试

## Task 12：给后端核心流程加 pytest

目标：至少能证明 request 和 triage 没坏。

- [ ] 在 `requirements.txt` 加：

```text
pytest
httpx
```

- [ ] 新建 `tests/test_requests.py`。

```python
from fastapi.testclient import TestClient
from app.main import app


client = TestClient(app)


def test_create_request_returns_new_status():
    payload = {
        "title": "Automate weekly status report",
        "business_goal": "Reduce manual reporting time for internal weekly updates.",
        "model": "gpt-4o-mini",
        "priority": "medium",
        "risk_level": "medium",
        "estimated_cost": 5.0,
        "owner": "DataMind Ops",
        "requester": "Business Team",
        "category": "reporting"
    }
    response = client.post("/requests", json=payload)
    assert response.status_code == 201
    data = response.json()
    assert data["status"] == "new"
    assert data["category"] == "reporting"
```

- [ ] 新建 `tests/test_triage.py`。

```python
from fastapi.testclient import TestClient
from app.main import app


client = TestClient(app)


def test_triage_populates_ai_fields():
    payload = {
        "title": "Create Notion workflow automation",
        "business_goal": "Turn manual Notion updates into a structured review process.",
        "model": "gpt-4o-mini",
        "priority": "medium",
        "risk_level": "medium",
        "estimated_cost": 5.0,
        "owner": "Platform Team",
        "requester": "Business Team"
    }
    created = client.post("/requests", json=payload).json()
    response = client.post(f"/triage/requests/{created['id']}")
    assert response.status_code == 200
    data = response.json()
    assert data["status"] == "pending_review"
    assert data["ai_next_action"]
    assert data["ai_confidence"] is not None
```

- [ ] 跑测试。

```bash
pytest -q
```

预期：测试通过。

- [ ] commit。

```bash
git add tests requirements.txt
git commit -m "test: cover request creation and triage flow"
```

---

# Phase 8：README 和作品包装

## Task 13：把 README 写成作品项目

目标：让别人一眼看懂这是业务型内部工具。

README 结构：

```markdown
# Internal AI Request Triage Agent

## Problem

Internal AI and automation ideas often arrive through scattered messages, docs, and meetings. Without a review workflow, teams lose track of ownership, risk, priority, and follow-up.

## Solution

This app provides a FastAPI + React control center for request intake, AI-assisted triage, human review, and audit visibility.

## Features

- JWT authentication
- Request CRUD
- AI triage suggestions
- Human review queue
- Status lifecycle
- Audit logs
- Metrics dashboard
- PostgreSQL persistence

## Architecture

React frontend -> FastAPI API -> PostgreSQL

The AI triage service is isolated behind a service layer. The base version supports deterministic mock triage, and the optional LLM mode can be enabled with environment variables.

## Agent/RAG Roadmap

- Notion source adapter
- RAG-backed request context
- Weekly digest generation
- Follow-up recommendation agent
```

- [ ] 加本地启动步骤。

后端：

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
createdb internal_ai_triage
alembic upgrade head
python -m uvicorn app.main:app --reload
```

前端：

```bash
cd ui
npm install
npm run dev
```

- [ ] 加截图。

截图放：

```text
docs/screenshots/
```

建议截图：

- Dashboard
- Request form
- Review queue
- Audit log

- [ ] commit。

```bash
git add README.md docs/screenshots
git commit -m "docs: package project for portfolio"
```

---

# 基础版完成检查表

全部完成才进入 Agent/RAG：

- [ ] 原始项目能本地跑。
- [ ] 项目名和 README 已改。
- [ ] Request model 有 source、category、AI suggestion、status lifecycle。
- [ ] Mock triage 可以无 API key 运行。
- [ ] 可选 LLM triage 有 feature flag。
- [ ] React 拆成多个组件。
- [ ] Review queue 可以 run triage / approve / reject。
- [ ] Auth 保护写操作。
- [ ] PostgreSQL + Alembic 可用。
- [ ] pytest 覆盖 request creation 和 triage flow。
- [ ] README 能作为 GitHub portfolio 展示。

---

# 做完基础版之后再做什么

按顺序做：

1. Notion Source Adapter
2. Weekly Digest Generator
3. RAG Context
4. Follow-up Recommendation Agent
5. Lightweight Workflow Execution

对应说明见：

```text
/Users/ke.chen/開発/practice/docs/agent-rag-extension-roadmap.md
```

