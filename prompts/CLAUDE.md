[根目录](../../CLAUDE.md) > **prompts**

# 提示模板模块

## 模块职责

提示模板模块负责管理所有 AI 交互的提示词模板。这些模板使用 Jinja2 模板引擎，支持动态变量注入，为不同的 AI 交互场景提供结构化、一致的提示词，确保 AI 响应的质量和一致性。

## 入口与结构

### 目录结构
```
prompts/
├── chat.jinja           # 通用聊天提示模板
└── source_chat.jinja    # 源文件专用聊天模板
```

## 提示模板详解

### 1. chat.jinja - 通用聊天提示模板

#### 设计理念
为认知学习助手提供系统角色定义和操作指南，帮助用户与研究文档进行有意义的对话。

#### 核心组件

##### 系统角色定义
```
You are a cognitive study assistant that helps users research and learn
by engaging in focused discussions about documents in their workspace.
```

##### 能力说明
- 访问项目信息和选定文档
- 保持学术严谨性的自然对话
- 基于上下文提供准确回答

##### 操作方法
1. **识别查询上下文**：理解用户意图
2. **分析上下文信息**：利用提供的文档
3. **遵循引用规范**：正确引用来源

##### 动态内容注入
```jinja2
{% if notebook %}
# PROJECT INFORMATION
{{notebook}}
{% endif %}

{% if context %}
# CONTEXT
The user has selected this context to help you with your response:
{{context}}
{% endif %}
```

##### 引用规范
- 使用 `[document_id]` 格式引用文档
- ID 格式：`type:randomstring`（如 `note:xyz`, `source:abc`）
- 不得编造文档 ID
- 必须使用完整的 ID 包括类型前缀

##### 示例说明
提供清晰的引用示例：
```
User: Can you tell me more about "Deep Learning"?
Assistant: Deep learning is a subset of machine learning... [note:iuiodadalknda]
```

### 2. source_chat.jinja - 源文件专用聊天模板

#### 设计目标
专门针对单个源文件的深度分析和讨论，提供更聚焦的对话体验。

#### 特色功能

##### 专业化角色
```
You are a specialized research assistant focused on helping users
deeply understand and analyze a specific source document.
```

##### 源文件信息展示
```jinja2
{% if source %}
**Source ID:** {{ source.id }}
**Title:** {{ source.title or "No title" }}

{% if source.topics %}
**Topics:** {{ source.topics | join(", ") }}
{% endif %}
{% endif %}
```

##### 上下文感知
- 显示源文件基本信息
- 展示主题标签
- 提供源文件专属上下文

##### 引用格式优化
- 源内容引用：`[source:id]`
- 洞察引用：`[insight:id]`
- 强调使用真实 ID

##### 对话焦点
专注于：
- 理解复杂概念
- 建立内容关联
- 探索深层含义
- 提出跟进问题

## 模板开发指南

### 1. 创建新模板

#### 基本结构
```jinja2
# SYSTEM ROLE
[系统角色定义]

# CAPABILITIES
[能力列表]

# YOUR OPERATING METHOD
[操作指南]

{% if dynamic_content %}
# DYNAMIC SECTION
{{ dynamic_content }}
{% endif %}

# SPECIFIC INSTRUCTIONS
[具体指令]
```

#### 变量使用
```jinja2
{% if variable %}
# SECTION TITLE
{{ variable }}
{% endif %}

{% for item in list %}
- {{ item }}
{% endfor %}

{{ variable | filter }}
```

### 2. 最佳实践

##### 清晰的结构
- 使用标题分层
- 逻辑清晰的组织
- 明确的指令说明

##### 条件渲染
```jinja2
{% if condition %}
<!-- 条件满足时显示 -->
{% else %}
<!-- 条件不满足时显示 -->
{% endif %}
```

##### 循环渲染
```jinja2
{% for item in items %}
{{ loop.index }}. {{ item }}
{% endfor %}
```

##### 过滤器使用
```jinja2
{{ text | upper }}
{{ list | join(", ") }}
{{ date | strftime("%Y-%m-%d") }}
```

### 3. 变量命名规范
- 使用小写字母和下划线
- 描述性名称
- 避免冲突

```python
# 好的例子
user_query
selected_context
source_document
conversation_history

# 避免使用
content
data
info
```

## 模板使用

### 1. 在代码中使用

```python
from jinja2 import Environment, FileSystemLoader

# 加载模板环境
env = Environment(loader=FileSystemLoader("prompts/"))
template = env.get_template("chat.jinja")

# 渲染模板
prompt = template.render(
    notebook=notebook_info,
    context=selected_context,
    user_query=query_text
)
```

### 2. 动态变量传递

```python
# 传递复杂对象
template.render(
    source={
        "id": "source:123",
        "title": "Research Paper",
        "topics": ["AI", "ML", "NLP"]
    },
    context=search_results,
    user_message="Explain the methodology"
)
```

### 3. 模板继承

创建基础模板：
```jinja2
{# base.jinja #}
# SYSTEM ROLE
You are a helpful AI assistant.

{% block content %}
{% endblock %}

# ADDITIONAL INSTRUCTIONS
{% block instructions %}
{% endblock %}
```

继承使用：
```jinja2
{# specialized.jinja #}
{% extends "base.jinja" %}

{% block content %}
# SPECIALIZED CAPABILITIES
- Capability 1
- Capability 2
{% endblock %}
```

## 模板测试

### 1. 单元测试
```python
def test_chat_template():
    template = env.get_template("chat.jinja")

    # 测试基本渲染
    output = template.render(
        notebook="Test Notebook",
        context="Test Context"
    )

    assert "Test Notebook" in output
    assert "Test Context" in output
```

### 2. 边界条件测试
- 空变量处理
- None 值处理
- 大型数据渲染
- 特殊字符转义

## 性能优化

### 1. 模板缓存
```python
# 启用自动缓存
env = Environment(
    loader=FileSystemLoader("prompts/"),
    autoescape=True,
    cache_size=100
)
```

### 2. 预编译模板
```python
# 预编译常用模板
compiled_templates = {}
for name in ["chat", "source_chat"]:
    template = env.get_template(f"{name}.jinja")
    compiled_templates[name] = template
```

## 安全考虑

### 1. 自动转义
```python
# 启用 HTML 自动转义
env = Environment(
    loader=FileSystemLoader("prompts/"),
    autoescape=True
)
```

### 2. 输入验证
- 验证变量类型
- 限制变量长度
- 过滤敏感内容

## 未来扩展

### 可能的新模板
1. **transformation.jinja** - 内容转换提示
2. **summarization.jinja** - 摘要生成提示
3. **question_generation.jinja** - 问题生成提示
4. **citation.jinja** - 引用格式化提示
5. **podcast_script.jinja** - 播客脚本生成提示

### 模板功能增强
1. **多语言支持** - 国际化模板
2. **个性化调整** - 基于用户偏好的模板
3. **A/B 测试** - 不同版本的效果对比
4. **模板版本控制** - 跟踪模板变更

## 相关文件清单

```
prompts/
├── chat.jinja           # 通用聊天模板
└── source_chat.jinja    # 源文件聊天模板
```

## 变更记录 (Changelog)

### 2025-12-09 08:29:13
- 📝 创建提示模板模块文档
- 📚 详细分析现有模板结构
- 🎯 提供模板开发指南
- 🔧 补充测试和安全建议