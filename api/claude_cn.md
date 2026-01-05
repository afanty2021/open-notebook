<<<<<<< HEAD
[根目录](../CLAUDE.md) > **api**

# API 模块文档

> 最后更新：2025-12-09 08:27:02

## 模块职责

API 模块是 Open Notebook 的 REST API 后端，负责处理所有 HTTP 请求，协调各个服务，并提供完整的 RESTful API 接口。该模块基于 FastAPI 框架构建，提供了自动生成的 OpenAPI 文档。

## 入口与启动

### 主入口文件
- **`main.py`** - FastAPI 应用主入口，包含：
  - 应用初始化
  - 中间件配置（CORS、认证）
  - 路由注册
  - 生命周期管理（启动时运行数据库迁移）

### 启动方式
```bash
# 直接运行
uv run run_api.py

# 使用 Make
make api
```

### 应用配置
- **默认端口**: 5055
- **API 文档**: `http://localhost:5055/docs`
- **OpenAPI 规范**: `http://localhost:5055/openapi.json`

## 对外接口

### 路由结构

```
api/
├── routers/
│   ├── auth.py          # 认证相关
│   ├── chat.py          # AI 对话
│   ├── commands.py      # 命令执行
│   ├── config.py        # 配置管理
│   ├── context.py       # 上下文控制
│   ├── embedding.py     # 向量嵌入
│   ├── embedding_rebuild.py  # 重建嵌入
│   ├── episode_profiles.py   # 播客节目配置
│   ├── insights.py      # 洞察生成
│   ├── models.py        # AI 模型管理
│   ├── notebooks.py     # 笔记本管理
│   ├── notes.py         # 笔记管理
│   ├── podcasts.py      # 播客生成
│   ├── search.py        # 搜索功能
│   ├── settings.py      # 系统设置
│   ├── source_chat.py   # 源文件对话
│   ├── sources.py       # 源文件管理
│   ├── speaker_profiles.py  # 说话人配置
│   └── transformations.py   # 内容转换
```

### 核心端点

#### 认证相关 (`/api/auth`)
- `POST /login` - 用户登录
- `POST /logout` - 用户登出
- `GET /me` - 获取当前用户信息

#### 笔记本管理 (`/api/notebooks`)
- `GET /` - 获取所有笔记本
- `POST /` - 创建新笔记本
- `GET /{id}` - 获取特定笔记本
- `PUT /{id}` - 更新笔记本
- `DELETE /{id}` - 删除笔记本

#### 源文件管理 (`/api/sources`)
- `GET /` - 获取所有源文件
- `POST /` - 上传新源文件
- `GET /{id}` - 获取源文件详情
- `DELETE /{id}` - 删除源文件
- `POST /{id}/process` - 处理源文件

#### AI 对话 (`/api/chat`)
- `POST /` - 发送对话消息
- `GET /sessions` - 获取会话列表
- `DELETE /sessions/{id}` - 删除会话

#### 播客生成 (`/api/podcasts`)
- `POST /generate` - 生成播客
- `GET /` - 获取播客列表
- `GET /{id}` - 获取播客详情

## 关键依赖与配置

### 服务层

API 模块包含以下核心服务：

1. **`chat_service.py`** - 处理 AI 对话逻辑
2. **`notebook_service.py`** - 笔记本业务逻辑
3. **`sources_service.py`** - 源文件处理
4. **`embedding_service.py`** - 向量嵌入管理
5. **`podcast_service.py`** - 播客生成服务
6. **`models_service.py`** - AI 模型管理

### 认证中间件

- **`auth.py`** - 实现密码认证中间件
- 支持可选的全局密码保护
- JWT 令牌验证

### 配置依赖

```python
# 主要依赖
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from loguru import logger

# 内部依赖
from open_notebook.database.async_migrate import AsyncMigrationManager
from api.auth import PasswordAuthMiddleware
```

## 数据模型

### 请求/响应模型
- **`models.py`** - 定义所有 Pydantic 模型
- 包含请求验证、序列化等

### 示例模型
```python
class NotebookCreate(BaseModel):
    name: str
    description: Optional[str] = None

class SourceUpload(BaseModel):
    notebook_id: str
    file_type: str
```

## 错误处理

### 异常类型
- **HTTPException** - 标准 HTTP 错误
- **自定义异常** - 业务逻辑错误
- **验证错误** - Pydantic 验证失败

### 错误响应格式
```json
{
  "detail": "错误描述",
  "error_code": "ERROR_CODE",
  "timestamp": "2025-12-09T08:27:02Z"
}
```

## 中间件

### CORS 配置
- 支持跨域请求
- 可配置允许的源

### 认证中间件
- 可选的密码保护
- 会话管理

### 请求日志
- 使用 loguru 记录所有请求
- 包含请求 ID 跟踪

## 测试与质量

### 测试文件
- **API 测试** 位于 `tests/test_models_api.py`
- 测试覆盖率：约 70%

### 测试运行
```bash
pytest tests/test_models_api.py -v
```

### 代码质量工具
- **Ruff** - 代码格式化和 linting
- **MyPy** - 类型检查
- **Pytest** - 单元测试

## 性能优化

### 异步支持
- 全异步路由处理
- 使用 async/await 模式

### 缓存策略
- AI 模型实例缓存
- 向量嵌入缓存

### 请求限制
- 可配置的请求速率限制
- 文件上传大小限制

## 安全措施

1. **输入验证** - 所有输入都经过 Pydantic 验证
2. **SQL 注入防护** - 使用参数化查询
3. **文件上传安全** - 类型检查和病毒扫描
4. **认证授权** - 可选的密码保护

## 部署注意事项

### 环境变量
- `PORT` - API 端口（默认 5055）
- `PASSWORD` - 可选的访问密码
- `CORS_ORIGINS` - 允许的跨域源

### Docker 部署
- API 运行在容器内
- 需要暴露端口 5055
- 建议使用反向代理

## 常见问题 (FAQ)

### Q: 如何添加新的 API 端点？
A: 在 `routers/` 目录下创建新文件，然后在 `main.py` 中注册路由。

### Q: 如何处理异步任务？
A: 使用 surreal-commands 后台任务系统，见 `commands/` 目录。

### Q: 如何调试 API 问题？
A: 查看 `http://localhost:5055/docs` 的 API 文档，检查日志输出。

## 相关文件清单

### 核心文件
- `main.py` - 应用入口
- `auth.py` - 认证逻辑
- `models.py` - 数据模型

### 路由文件（按功能分组）
- **笔记本**: `notebooks.py`
- **源文件**: `sources.py`, `source_chat.py`
- **AI 功能**: `chat.py`, `models.py`, `embedding.py`
- **播客**: `podcasts.py`, `episode_profiles.py`, `speaker_profiles.py`
- **系统**: `settings.py`, `config.py`, `commands.py`

### 服务文件
- 所有 `*_service.py` 文件包含业务逻辑

## 变更记录 (Changelog)

### 2025-12-09 08:27:02
- 📝 创建 API 模块文档
- 🏗️ 整理路由结构
- 📊 添加核心端点说明
- 🔧 补充部署和调试指南

---

*此文档由 AI 自动生成，如需更新请参考项目贡献指南*
=======
# API Module

FastAPI-based REST backend exposing services for notebooks, sources, notes, chat, podcasts, and AI model management.

## Purpose

FastAPI application serving three architectural layers: routes (HTTP endpoints), services (business logic), and models (request/response schemas). Integrates LangGraph workflows (chat, ask, source_chat), SurrealDB persistence, and AI providers via Esperanto.

## Architecture Overview

**Three layers**:
1. **Routes** (`routers/*`): HTTP endpoints mapping to services
2. **Services** (`*_service.py`): Business logic orchestrating domain models, database, graphs, AI providers
3. **Models** (`models.py`): Pydantic request/response schemas with validation

**Startup flow**:
- Load .env environment variables
- Initialize CORS middleware + password auth middleware
- Run database migrations via AsyncMigrationManager on lifespan startup
- Register all routers

**Key services**:
- `chat_service.py`: Invokes chat graph with messages, context
- `podcast_service.py`: Orchestrates outline + transcript generation
- `sources_service.py`: Content ingestion, vectorization, metadata
- `notes_service.py`: Note creation, linking to sources/insights
- `transformations_service.py`: Applies transformations to content
- `models_service.py`: Manages AI provider/model configuration
- `episode_profiles_service.py`: Manages podcast speaker/episode profiles

## Component Catalog

### Main Application
- **main.py**: FastAPI app initialization, CORS setup, auth middleware, lifespan event, router registration
- **Lifespan handler**: Runs AsyncMigrationManager on startup (database schema migration)
- **Auth middleware**: PasswordAuthMiddleware protects endpoints (password-based access control)

### Services (Business Logic)
- **chat_service.py**: Invokes chat.py graph; handles message history via SqliteSaver
- **podcast_service.py**: Generates outline (outline.jinja), then transcript (transcript.jinja) for episodes
- **sources_service.py**: Ingests files/URLs (content_core), extracts text, vectorizes, saves to SurrealDB
- **transformations_service.py**: Applies transformations via transformation.py graph
- **models_service.py**: Manages ModelManager config (AI provider overrides)
- **episode_profiles_service.py**: CRUD for EpisodeProfile and SpeakerProfile models
- **insights_service.py**: Generates and retrieves source insights
- **notes_service.py**: Creates notes linked to sources/insights

### Models (Schemas)
- **models.py**: Pydantic schemas for request/response validation
- Request bodies: ChatRequest, CreateNoteRequest, PodcastGenerationRequest, etc.
- Response bodies: ChatResponse, NoteResponse, PodcastResponse, etc.
- Custom validators for enum fields, file paths, model references

### Routers
- **routers/chat.py**: POST /chat
- **routers/source_chat.py**: POST /source/{source_id}/chat
- **routers/podcasts.py**: POST /podcasts, GET /podcasts/{id}, etc.
- **routers/notes.py**: POST /notes, GET /notes/{id}
- **routers/sources.py**: POST /sources, GET /sources/{id}, DELETE /sources/{id}
- **routers/models.py**: GET /models, POST /models/config
- **routers/transformations.py**: POST /transformations
- **routers/insights.py**: GET /sources/{source_id}/insights
- **routers/auth.py**: POST /auth/password (password-based auth)
- **routers/commands.py**: GET /commands/{command_id} (job status tracking)

## Common Patterns

- **Service injection via FastAPI**: Routers import services directly; no DI framework
- **Async/await throughout**: All DB queries, graph invocations, AI calls are async
- **SurrealDB transactions**: Services use repo_query, repo_create, repo_upsert from database layer
- **Config override pattern**: Models/config override via models_service passed to graph.ainvoke(config=...)
- **Error handling**: Services catch exceptions and return HTTP status codes (400 Bad Request, 404 Not Found, 500 Internal Server Error)
- **Logging**: loguru logger in main.py; services expected to log key operations
- **Response normalization**: All responses follow standard schema (data + metadata structure)

## Key Dependencies

- `fastapi`: FastAPI app, routers, HTTPException
- `pydantic`: Validation models with Field, field_validator
- `open_notebook.graphs`: chat, ask, source_chat, source, transformation graphs
- `open_notebook.database`: SurrealDB repository functions (repo_query, repo_create, repo_upsert)
- `open_notebook.domain`: Notebook, Source, Note, SourceInsight models
- `open_notebook.ai.provision`: provision_langchain_model() factory
- `ai_prompter`: Prompter for template rendering
- `content_core`: extract_content() for file/URL processing
- `esperanto`: AI provider client library (LLM, embeddings, TTS)
- `surreal_commands`: Job queue for async operations (podcast generation)
- `loguru`: Structured logging

## Important Quirks & Gotchas

- **Migration auto-run**: Database schema migrations run on every API startup (via lifespan); no manual migration steps
- **PasswordAuthMiddleware is basic**: Uses simple password check; production deployments should replace with OAuth/JWT
- **No request rate limiting**: No built-in rate limiting; deployment must add via proxy/middleware
- **Service state is stateless**: Services don't cache results; each request re-queries database/AI models
- **Graph invocation is blocking**: chat/podcast workflows may take minutes; no timeout handling in services
- **Command job fire-and-forget**: podcast_service.py submits jobs but doesn't wait (async job queue pattern)
- **Model override scoping**: Model config override via RunnableConfig is per-request only (not persistent)
- **CORS open by default**: main.py CORS settings allow all origins (restrict before production)
- **No OpenAPI security scheme**: API docs available without auth (disable before production)
- **Services don't validate user permission**: All endpoints trust authentication layer; no per-notebook permission checks

## How to Add New Endpoint

1. Create router file in `routers/` (e.g., `routers/new_feature.py`)
2. Import router into `main.py` and register: `app.include_router(new_feature.router, tags=["new_feature"])`
3. Create service in `new_feature_service.py` with business logic
4. Define request/response schemas in `models.py` (or create `new_feature_models.py`)
5. Implement router functions calling service methods
6. Test with `uv run uvicorn api.main:app --host 0.0.0.0 --port 5055`

## Testing Patterns

- **Interactive docs**: http://localhost:5055/docs (Swagger UI)
- **Direct service tests**: Import service, call methods directly with test data
- **Mock graphs**: Replace graph.ainvoke() with mock for testing service logic
- **Database: Use test database** (separate SurrealDB instance or mock repo_query)
>>>>>>> upstream/main
