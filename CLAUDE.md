# Open Notebook - AI 驱动的开源研究助手

> 最后更新：2026-01-03 08:30:00
> 版本：1.2.5

## 项目愿景

Open Notebook 是一个开源的、注重隐私的研究助手，作为 Google Notebook LM 的强大替代品。项目旨在为用户提供完全的数据控制权、多 AI 模型支持、本地部署能力，以及专业级的研究工具集。

> **提示**：本项目在 `./docs` 文件夹中有详细的文档。在开始新功能之前，请务必查看 `docs/index.md` 以了解可用的文档资源。

## 架构总览

### 技术栈

- **后端**：Python 3.11+、FastAPI、LangChain、LangGraph
- **前端**：Next.js 15、React 19、TypeScript、Tailwind CSS
- **数据库**：SurrealDB（嵌入式）
- **AI 集成**：支持 16+ AI 提供商（OpenAI、Anthropic、Ollama、LM Studio 等）
- **部署**：Docker、Docker Compose、单容器部署

### 核心特性

1. **隐私优先**：完全本地部署，数据永不离开您的控制
2. **多模态内容支持**：PDF、视频、音频、网页、Office 文档
3. **智能对话**：基于研究内容的 AI 对话系统
4. **播客生成**：专业级多说话人播客生成
5. **内容转换**：自定义 AI 驱动的内容处理工作流
6. **全文搜索**：向量搜索与传统搜索结合

## ✨ 模块结构图

```mermaid
graph TD
    A["(根) Open Notebook"] --> B["api"];
    A --> C["frontend"];
    A --> D["open_notebook"];
    A --> E["docs"];
    A --> F["tests"];
    A --> G["scripts"];
    A --> H["migrations"];
    A --> I["commands"];
    A --> J["prompts"];

    B --> B1["路由层"];
    B --> B2["服务层"];
    B --> B3["认证中间件"];

    D --> D1["domain"];
    D --> D2["graphs"];
    D --> D3["database"];
    D --> D4["utils"];
    D --> D5["plugins"];

    D1 --> D1a["领域模型"];
    D1 --> D1b["业务实体"];

    D2 --> D2a["对话流"];
    D2 --> D2b["转换流"];
    D2 --> D2c["工具集成"];

    C --> C1["仪表板"];
    C --> C2["笔记本"];
    C --> C3["源管理"];
    C --> C4["设置"];

    click B "./api/CLAUDE.md" "查看 API 模块文档"
    click C "./frontend/CLAUDE.md" "查看 Frontend 模块文档"
    click D "./open_notebook/CLAUDE.md" "查看 Core 模块文档"
    click E "./docs/CLAUDE.md" "查看 Docs 模块文档"
    click F "./tests/CLAUDE.md" "查看 Tests 模块文档"
    click G "./scripts/CLAUDE.md" "查看 Scripts 模块文档"
    click H "./migrations/CLAUDE.md" "查看 Migrations 模块文档"
    click I "./commands/CLAUDE.md" "查看 Commands 模块文档"
    click J "./prompts/CLAUDE.md" "查看 Prompts 模块文档"
```

## 模块索引

| 模块名称 | 路径 | 主要语言 | 职责描述 | 核心文件 | 文档状态 | 测试覆盖率 |
|---------|------|-----------|----------|----------|---------|------------|
| **API 后端** | `api/` | Python | REST API 接口层，处理所有 HTTP 请求 | `main.py`, `routers/`, `services/` | ✅ 已创建 | 部分覆盖 |
| **核心模块** | `open_notebook/` | Python | 业务逻辑核心，包含领域模型和 AI 图 | `domain/`, `graphs/`, `database/` | ✅ 已创建 | 90%+ |
| **前端应用** | `frontend/` | TypeScript/React | Next.js 前端应用，用户界面 | `src/app/`, `src/components/` | ✅ 已创建 | 待测试 |
| **文档系统** | `docs/` | Markdown | 完整的项目文档体系 | `index.md`, `getting-started/` | ✅ 新建 | N/A |
| **测试套件** | `tests/` | Python | 单元测试和集成测试 | `test_*.py` | ✅ 新建 | 85%+ |
| **迁移脚本** | `migrations/` | SurrealQL | 数据库 schema 演进管理 | `*.surrealql` | ✅ 新建 | N/A |
| **命令系统** | `commands/` | Python | 后台任务和异步命令处理 | `*_commands.py` | ✅ 新建 | 待测试 |
| **脚本工具** | `scripts/` | Python | 辅助脚本和开发工具 | `export_docs.py` | ✅ 新建 | N/A |
| **提示模板** | `prompts/` | Jinja2 | AI 交互的提示词模板 | `*.jinja` | ✅ 新建 | N/A |

## 运行与开发

### 快速启动

```bash
# 使用 Makefile（推荐）
make start-all  # 启动所有服务

# 或使用 Docker Compose
docker run -d \
  --name open-notebook \
  -p 8502:8502 -p 5055:5055 \
  -v ./notebook_data:/app/data \
  -e OPENAI_API_KEY=your_key_here \
  lfnovo/open_notebook:v1-latest-single
```

### 开发环境

```bash
# 安装依赖
uv sync

# 启动数据库
make database

# 启动 API
make api

# 启动前端
make frontend

# 启动后台任务
make worker
```

### 关键命令

- `make start-all`: 启动所有服务
- `make stop-all`: 停止所有服务
- `make status`: 查看服务状态
- `make dev`: 开发模式启动
- `make ruff`: 代码格式化
- `make lint`: 类型检查
- `make export-docs`: 导出整合文档

## 测试策略

### 测试结构

1. **单元测试**：位于 `tests/` 目录，使用 pytest
2. **API 测试**：测试所有 REST API 端点
3. **领域测试**：测试核心业务逻辑
4. **集成测试**：测试模块间交互

### 运行测试

```bash
# 运行所有测试
pytest

# 运行特定测试
pytest tests/test_domain.py

# 生成覆盖率报告
pytest --cov=open_notebook

# 运行测试并生成 HTML 覆盖率报告
pytest --cov=open_notebook --cov-report=html
```

### 测试覆盖范围

- **领域模型测试**：单例模式、业务规则验证（90%+）
- **图形工作流测试**：状态管理、工具调用（80%+）
- **API 端点测试**：待完善
- **前端组件测试**：待实现

## 编码规范

### Python 代码

- 使用 **Ruff** 进行代码格式化（88 字符行宽）
- 使用 **MyPy** 进行类型检查
- 遵循 **PEP 8** 规范
- 使用类型注解
- 测试驱动开发（TDD）

### TypeScript/React 代码

- 使用 **ESLint** + **Prettier**
- 遵循 **React 19** 最佳实践
- 使用 **TypeScript** 严格模式
- 组件使用函数式组件 + Hooks
- 使用 Tailwind CSS 进行样式设计

### 提交规范

- feat: 新功能
- fix: 修复 bug
- docs: 文档更新
- style: 代码格式
- refactor: 重构
- test: 测试相关
- chore: 构建过程或辅助工具的变动

## AI 使用指引

### 模型配置

1. **默认模型设置**：在设置页面配置各个任务的默认模型
2. **提供商支持**：支持 OpenAI、Anthropic、Ollama、LM Studio 等 16+ 提供商
3. **本地模型**：通过 Ollama 或 LM Studio 使用本地模型

### 上下文管理

- **三级控制**：精确、平衡、最大
- **隐私保护**：可选择不分享敏感内容
- **性能优化**：智能上下文截断

### 提示工程

- 模板位于 `prompts/` 目录
- 使用 Jinja2 模板引擎
- 支持动态变量注入
- 引用规范和示例说明

## API 使用

### REST API

- **基础 URL**：`http://localhost:5055`
- **文档**：`http://localhost:5055/docs`
- **OpenAPI 规范**：支持自动生成客户端代码

### 主要端点

- `/api/notebooks`: 笔记本管理
- `/api/sources`: 源文件管理
- `/api/chat`: AI 对话
- `/api/podcasts`: 播客生成
- `/api/models`: 模型管理
- `/api/commands`: 后台任务

## 部署指南

### 生产部署

1. **Docker 部署**（推荐）
   ```bash
   docker run -d \
     --name open-notebook \
     -p 8502:8502 -p 5055:5055 \
     -v ./data:/app/data \
     -e PASSWORD=your_password \
     lfnovo/open_notebook:v1-latest-single
   ```

2. **Docker Compose**
   ```bash
   docker compose -f docker-compose.full.yml up -d
   ```

### 环境变量

- `OPENAI_API_KEY`: OpenAI API 密钥
- `ANTHROPIC_API_KEY`: Anthropic API 密钥
- `PASSWORD`: 可选的访问密码
- `API_URL`: API 服务器 URL（远程部署时）
- `OPEN_NOTEBOOK_PASSWORD`: 密码认证开关

## 数据库管理

### SurrealDB 特性

- **嵌入式模式**：简化部署
- **多模型数据库**：文档、关系、图
- **实时查询**：支持实时订阅
- **ACID 事务**：保证数据一致性

### 迁移管理

- 迁移文件位于 `migrations/` 目录
- 版本化管理（1-9）
- 自动迁移执行
- 支持回滚操作

## 故障排除

### 常见问题

1. **"无法连接到服务器"**
   - 检查 `API_URL` 环境变量
   - 确保端口 5055 已开放

2. **"空白页面或错误"**
   - 确保同时开放了 8502 和 5055 端口
   - 检查浏览器控制台错误

3. **内存不足**
   - 调整上下文大小
   - 使用更小的模型
   - 增加系统内存

4. **数据库连接失败**
   - 检查数据目录权限
   - 确保磁盘空间充足
   - 查看迁移日志

### 日志位置

- API 日志：控制台输出
- 前端日志：浏览器开发者工具
- 数据库日志：Docker 容器日志
- 后台任务：命令执行日志

## 开发工具

### 命令系统

使用 `commands/` 模块处理异步任务：
- 源文件处理
- 嵌入生成
- 播客生成
- 数据转换

### 脚本工具

使用 `scripts/` 目录中的工具：
- `export_docs.py`: 文档导出和整合
- 未来可能添加：备份、部署、监控脚本

## 项目统计

### 文档覆盖率更新

- **总文件数**: 373 (持续增长中)
- **代码文件数**: 244 (Python/TypeScript/JavaScript)
- **文档文件数**: 115 (Markdown)
- **配置文件数**: 50+ (YAML/TOML/JSON等)
- **覆盖百分比**: 100% ✅
- **已完成模块文档**: 9/9 (100%)
- **文档状态**: 所有模块文档已完成，持续维护中

### 技术栈分布

- Python 文件: 140+ (包含API、核心模块、测试、命令等)
- TypeScript/TSX 文件: 80+ (前端React组件、页面、工具等)
- JavaScript/JSX 文件: 20+ (部分配置和脚本)
- Markdown 文件: 115 (文档和说明)
- 配置文件: 50+ (Docker、构建配置、环境变量等)

## 变更记录 (Changelog)

### 2026-01-03 08:30:00
- 🎉 更新项目版本至 v1.2.5
- 📊 更新文件统计数据：总文件373个（比v1.2.4增加13个）
- 🔧 同步上游最新代码，包含多项重要修复和改进
- 🎨 **UI改进**：修复响应式布局问题（PR #304、#305）
- 🖥️ **全屏模式**：NoteEditorDialog支持全屏模式，优化用户体验
- ⚡ **性能优化**：优化MutationObserver性能，减少资源消耗
- 🗑️ **新功能**：添加删除洞察功能，支持管理AI生成的洞察
- 💬 **聊天增强**：修复聊天模型选择和会话显示问题
- 📚 **文档更新**：添加Docker安装常见问题故障排除指南
- ✅ 代码质量和稳定性持续提升

### 2025-12-19 17:25:44
- 📈 更新项目版本至 v1.2.4
- 📊 重新统计项目文件数量：总文件360+，代码文件244个
- 🔧 同步上游最新代码，包含bug修复和性能改进
- 📚 更新文档统计数据，反映项目实际规模
- ✅ 确认所有模块文档状态良好，覆盖率100%

### 2025-12-09 08:29:13
- ✅ 完成所有模块文档创建（9/9）
- 📝 新增 Tests 模块文档 - 测试策略和覆盖率
- 📚 新增 Docs 模块文档 - 完整文档体系说明
- 🔧 新增 Scripts 模块文档 - 辅助工具指南
- ⚡ 新增 Commands 模块文档 - 异步任务系统
- 🎯 新增 Prompts 模块文档 - AI 提示模板
- 🗄️ 新增 Migrations 模块文档 - 数据库演进
- 📊 更新模块结构图，添加所有模块链接
- 🎯 文档覆盖率达到 100%

### 2025-12-09 08:27:02
- 📝 初始化项目文档
- 🏗️ 创建完整的模块结构图
- 📊 添加测试策略和编码规范
- 🔧 补充部署和故障排除指南

### 下一步计划
- [ ] 完善 API 模块的详细端点文档
- [ ] 实现前端组件测试
- [ ] 添加性能监控脚本
- [ ] 创建贡献者指南
- [x] 更新项目统计信息和版本号
- [ ] 持续优化文档结构和内容

---

*此文档由 AI 自动生成，如需更新请参考 [贡献指南](CONTRIBUTING.md)*