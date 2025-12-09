[根目录](../../CLAUDE.md) > **commands**

# 命令系统模块

## 模块职责

命令系统模块实现了与 Surreal Commands 的集成，提供后台任务处理能力。这些命令用于处理耗时的异步操作，如源文件处理、嵌入生成、播客生成等，确保 API 响应的快速性和系统的可扩展性。

## 入口与结构

### 模块入口
- **`__init__.py`**: 导出所有可用命令
  - 集中管理命令注册
  - 提供统一的命令接口

### 命令文件列表
```
commands/
├── __init__.py              # 命令模块入口
├── source_commands.py       # 源文件处理命令
├── embedding_commands.py    # 嵌入相关命令
├── podcast_commands.py      # 播客生成命令
└── example_commands.py      # 示例命令
```

## 命令详解

### 1. source_commands.py - 源文件处理命令

#### process_source_command
处理单个源文件的核心命令。

**功能特性**：
- 支持多种文件格式
- 自动内容提取和预处理
- 生成 AI 洞察
- 创建嵌入向量
- 更新搜索索引

**输入参数**：
```python
class SourceProcessingInput(CommandInput):
    source_id: str                    # 源文件 ID
    content_state: Dict[str, Any]     # 内容处理状态
    notebook_ids: List[str]           # 关联的笔记本 ID
    transformations: List[str]        # 应用的转换
    embed: bool                       # 是否生成嵌入
```

**输出结果**：
```python
class SourceProcessingOutput(CommandOutput):
    success: bool                     # 处理是否成功
    source_id: str                    # 源文件 ID
    embedded_chunks: int = 0          # 嵌入的块数
    insights_created: int = 0         # 创建的洞察数
    processing_time: float            # 处理耗时
    error_message: Optional[str]      # 错误信息
```

**重试配置**：
- 最大重试次数：3 次
- 指数退避策略
- 可配置的重试延迟

### 2. embedding_commands.py - 嵌入命令

#### embed_single_item_command
为单个内容项生成嵌入向量。

**用途**：
- 源文件内容嵌入
- 笔记内容嵌入
- 洞察内容嵌入

**特性**：
- 支持多种嵌入模型
- 批量处理优化
- 错误恢复机制

#### rebuild_embeddings_command
重建整个系统的嵌入向量。

**使用场景**：
- 更换嵌入模型
- 修复损坏的嵌入
- 批量更新嵌入

**注意事项**：
- 资源密集型操作
- 建议在低峰期执行
- 支持进度跟踪

### 3. podcast_commands.py - 播客生成命令

#### generate_podcast_command
生成多说话人播客。

**处理流程**：
1. 分析源内容
2. 生成播客大纲
3. 创建脚本
4. 生成音频
5. 后期处理

**配置选项**：
- 说话人数量（1-4）
- 声音特性配置
- 播客时长控制
- 输出格式选择

### 4. example_commands.py - 示例命令

提供命令开发的参考实现：
- `analyze_data_command`: 数据分析示例
- `process_text_command`: 文本处理示例

## 命令开发指南

### 1. 创建新命令

#### 步骤 1：定义输入输出
```python
from pydantic import BaseModel
from surreal_commands import CommandInput, CommandOutput

class MyCommandInput(CommandInput):
    required_field: str
    optional_field: Optional[str] = None

class MyCommandOutput(CommandOutput):
    result: str
    success_count: int
```

#### 步骤 2：实现命令函数
```python
from surreal_commands import command, CommandInput, CommandOutput
from loguru import logger

@command(
    "my_command",
    app="open_notebook",
    retry={"max_attempts": 3, "backoff": "exponential"},
)
async def my_command(input_data: MyCommandInput) -> MyCommandOutput:
    """
    命令功能描述

    Args:
        input_data: 命令输入参数

    Returns:
        命令执行结果
    """
    try:
        logger.info(f"执行命令: {input_data.required_field}")

        # 实现命令逻辑
        result = process_data(input_data.required_field)

        return MyCommandOutput(
            success=True,
            result=result,
            success_count=1
        )
    except Exception as e:
        logger.error(f"命令执行失败: {e}")
        return MyCommandOutput(
            success=False,
            result="",
            success_count=0,
            error_message=str(e)
        )
```

#### 步骤 3：注册命令
在 `__init__.py` 中导入和导出：
```python
from .my_command import my_command

__all__ = [
    "my_command",
    # ... 其他命令
]
```

### 2. 命令装饰器选项

```python
@command(
    name="command_name",           # 命令名称
    app="open_notebook",          # 应用名称
    retry={                        # 重试配置
        "max_attempts": 3,         # 最大重试次数
        "backoff": "exponential",  # 退避策略
        "initial_delay": 1.0,      # 初始延迟
    },
    timeout=300,                   # 超时时间（秒）
    queue="default",               # 队列名称
)
```

### 3. 最佳实践

#### 错误处理
- 使用 try-catch 捕获异常
- 记录详细的错误日志
- 返回有意义的错误信息
- 确保资源清理

#### 日志记录
```python
from loguru import logger

# 开始日志
logger.info(f"开始处理: {source_id}")

# 进度日志
logger.info(f"处理进度: {current}/{total}")

# 错误日志
logger.error(f"处理失败: {error}")

# 完成日志
logger.info(f"处理完成: 耗时 {duration} 秒")
```

#### 性能优化
- 使用批量操作
- 实现进度跟踪
- 优化数据库查询
- 管理内存使用

## 命令执行

### 1. 通过 API 触发
```python
# 在 API 端点中触发命令
from open_notebook.commands import process_source_command

command_id = await process_source_command.delay(
    source_id="source:123",
    content_state={},
    notebook_ids=["notebook:456"],
    transformations=["summarize"],
    embed=True
)
```

### 2. 查看命令状态
```python
# 获取命令状态
from surreal_commands import get_command_status

status = await get_command_status(command_id)
```

### 3. 命令结果处理
```python
# 处理命令完成事件
@command_handler("process_source", "completed")
async def handle_source_completed(result):
    # 更新数据库
    # 发送通知
    # 清理资源
    pass
```

## 监控和调试

### 1. 命令队列监控
- 队列长度监控
- 处理时间统计
- 失败率追踪

### 2. 性能指标
- 平均执行时间
- 资源使用情况
- 并发处理能力

### 3. 调试技巧
- 查看命令日志
- 检查输入参数
- 验证输出结果
- 分析错误堆栈

## 相关文件清单

```
commands/
├── __init__.py              # 命令导出
├── source_commands.py       # 源文件处理
├── embedding_commands.py    # 嵌入操作
├── podcast_commands.py      # 播客生成
└── example_commands.py      # 示例实现
```

## 变更记录 (Changelog)

### 2025-12-09 08:29:13
- 📝 创建命令系统模块文档
- 🎯 详细说明各命令功能
- 📚 提供命令开发指南
- 🔧 补充调试和监控方法