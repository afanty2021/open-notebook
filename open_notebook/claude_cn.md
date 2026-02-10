[根目录](../CLAUDE.md) > **open_notebook**

# Open Notebook 核心模块

> 最后更新：2025-12-09 08:27:02

## 模块职责

`open_notebook` 是整个项目的核心 Python 包，包含所有业务逻辑、领域模型、AI 图集成、数据库操作和工具函数。它被设计为独立的可重用包，为 API 层提供核心功能支持。

## 入口与启动

### 包初始化
- **`__init__.py`** - 包初始化，导出主要类和函数
- **`config.py`** - 全局配置管理

### 配置项
```python
# 数据文件夹
DATA_FOLDER = "./data"
UPLOADS_FOLDER = f"{DATA_FOLDER}/uploads"
LANGGRAPH_CHECKPOINT_FILE = f"{DATA_FOLDER}/sqlite-db/checkpoints.sqlite"
TIKTOKEN_CACHE_DIR = f"{DATA_FOLDER}/tiktoken-cache"
```

## 对外接口

### 主要导出
```python
from open_notebook.domain import Notebook, Source, Note
from open_notebook.graphs import ChatGraph, TransformationGraph
from open_notebook.database import repository
from open_notebook.utils import context_builder
```

## 关键依赖与配置

### 外部依赖
- **SurrealDB** - 主数据库
- **LangChain** - AI 框架
- **LangGraph** - AI 工作流
- **Esperanto** - AI 模型抽象层
- **Content Core** - 内容处理
- **Podcast Creator** - 播客生成

### 内部结构
```
open_notebook/
├── database/         # 数据库层
├── domain/          # 领域模型
├── graphs/          # AI 工作流
├── utils/           # 工具函数
├── plugins/         # 插件系统
├── config.py        # 配置
└── exceptions.py    # 异常定义
```

## 数据模型

### 领域模型 (`domain/`)

1. **`base.py`** - 基础模型类
   - `ObjectModel` - 基础对象模型
   - `RecordModel` - 记录模型（带缓存）

2. **`models.py`** - AI 模型管理
   - `Model` - AI 模型定义
   - `ModelManager` - 模型管理器
   - `DefaultModels` - 默认模型配置

3. **`notebook.py`** - 笔记本模型
   - `Notebook` - 笔记本实体
   - 笔记本与源的关联关系

4. **`podcast.py`** - 播客相关模型
   - `Podcast` - 播客实体
   - `SpeakerProfile` - 说话人配置
   - `EpisodeProfile` - 节目配置

5. **`transformation.py`** - 内容转换模型
   - `Transformation` - 转换规则
   - 转换模板和配置

### 示例模型定义
```python
class Notebook(ObjectModel):
    table_name: ClassVar[str] = "notebook"
    name: str
    description: Optional[str] = None
    created_at: datetime = Field(default_factory=datetime.utcnow)
```

## AI 图系统 (`graphs/`)

LangGraph 实现的 AI 工作流：

1. **`chat.py`** - 对话图
   - 处理用户对话
   - 上下文管理
   - 响应生成

2. **`ask.py`** - 问答图
   - 基于内容的问答
   - 引用生成

3. **`transformation.py`** - 内容转换图
   - 自定义内容处理
   - 批量转换支持

4. **`source.py`** - 源处理图
   - 文档解析
   - 内容提取

5. **`tools.py`** - AI 工具集成
   - 搜索工具
   - 计算工具
   - 外部 API 集成

### 图使用示例
```python
from open_notebook.graphs.chat import ChatGraph

# 创建对话图
chat_graph = ChatGraph(
    model_id="gpt-4",
    notebook_id="notebook_123"
)

# 执行对话
response = await chat_graph.ainvoke({
    "messages": [("user", "解释这个概念")]
})
```

## 数据库层 (`database/`)

1. **`repository.py`** - 数据访问层
   - `repository` - 主数据库实例
   - `repo_query` - 查询函数
   - CRUD 操作封装

2. **`migrate.py`** - 数据库迁移
   - 迁移管理
   - 版本控制

3. **`async_migrate.py`** - 异步迁移
   - 异步迁移支持

### 数据库操作示例
```python
from open_notebook.database.repository import repo_query

# 查询笔记本
notebooks = await repo_query(
    "SELECT * FROM notebook ORDER BY created_at DESC"
)

# 创建记录
await Notebook.create({
    "name": "新笔记本",
    "description": "描述"
})
```

## 工具函数 (`utils/`)

1. **`context_builder.py`** - 上下文构建
   - 动态上下文组装
   - 隐私级别控制
   - Token 管理

2. **`text_utils.py`** - 文本处理
   - 文本清理
   - 格式化

3. **`token_utils.py`** - Token 管理
   - Token 计数
   - 截断策略

4. **`version_utils.py`** - 版本管理
   - 版本检查
   - 更新提示

## 插件系统 (`plugins/`)

### 播客插件
- **`podcasts.py`** - 播客生成插件
  - 多说话人支持
  - 自定义模板
  - 音频处理集成

## 异常处理 (`exceptions.py`)

自定义异常类：
- `OpenNotebookError` - 基础异常
- `ModelNotFoundError` - 模型未找到
- `InsufficientContextError` - 上下文不足
- `ProcessingError` - 处理错误

## 测试与质量

### 测试覆盖
- **单元测试** - 各模块独立测试
- **集成测试** - 模块间交互测试
- **覆盖率** - 目标 90%+

### 测试运行
```bash
# 运行核心模块测试
pytest tests/test_domain.py tests/test_graphs.py -v

# 生成覆盖率报告
pytest --cov=open_notebook
```

## 性能优化

### 缓存策略
1. **模型缓存** - AI 模型实例复用
2. **查询缓存** - 数据库查询结果缓存
3. **向量缓存** - 嵌入向量缓存

### 异步处理
- 全异步 API 设计
- 后台任务队列
- 批量操作支持

## 扩展指南

### 添加新的领域模型
1. 在 `domain/` 创建新文件
2. 继承 `ObjectModel` 或 `RecordModel`
3. 定义 `table_name` 和字段

### 创建新的 AI 图
1. 在 `graphs/` 创建新文件
2. 使用 LangGraph 构建工作流
3. 定义节点和边

### 添加工具函数
1. 在 `utils/` 添加模块
2. 编写纯函数
3. 添加类型注解

## 常见问题 (FAQ)

### Q: 如何自定义 AI 模型？
A: 通过 `ModelManager` 配置，支持 16+ 提供商。

### Q: 如何扩展内容转换？
A: 在 `domain/transformation.py` 添加新类型，或创建自定义图。

### Q: 数据库迁移如何工作？
A: 使用 SurrealDB 的迁移系统，文件在 `migrations/` 目录。

### Q: 如何优化性能？
A: 使用缓存、调整上下文大小、选择合适的模型。

## 相关文件清单

### 核心模块
- `config.py` - 配置管理
- `exceptions.py` - 异常定义

### 领域层
- `domain/base.py` - 基础模型
- `domain/models.py` - AI 模型
- `domain/notebook.py` - 笔记本
- `domain/podcast.py` - 播客
- `domain/transformation.py` - 转换

### AI 图
- `graphs/chat.py` - 对话图
- `graphs/ask.py` - 问答图
- `graphs/transformation.py` - 转换图
- `graphs/tools.py` - 工具集

### 数据层
- `database/repository.py` - 数据访问
- `database/migrate.py` - 迁移

### 工具
- `utils/context_builder.py` - 上下文构建
- `utils/token_utils.py` - Token 管理

## 变更记录 (Changelog)

### 2025-12-09 08:27:02
- 📝 创建核心模块文档
- 🏗️ 整理领域模型结构
- 📊 添加 AI 图说明
- 🔧 补充扩展指南

---

*此文档由 AI 自动生成，如需更新请参考项目贡献指南*
