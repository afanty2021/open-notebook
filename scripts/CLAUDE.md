[根目录](../../CLAUDE.md) > **scripts**

# 脚本工具模块

## 模块职责

脚本工具模块包含各种辅助脚本和工具，用于简化开发、部署和维护任务。这些脚本帮助开发者自动化常见操作，提高工作效率。

## 入口与使用

### 目录结构
```
scripts/
├── README.md           # 脚本文档说明
└── export_docs.py      # 文档导出脚本
```

## 脚本详解

### 1. export_docs.py - 文档导出工具

#### 功能描述
将 `docs/` 目录中的 Markdown 文件整合导出，用于 ChatGPT 或其他有文件上传限制的平台。该脚本会扫描所有子目录，为每个子目录创建一个整合的 Markdown 文件。

#### 主要功能
- 扫描 `docs/` 文件夹的所有子目录
- 合并每个子目录中的所有 `.md` 文件（排除 `index.md`）
- 为每个子目录创建一个整合的 Markdown 文件
- 将所有导出文件保存到项目根目录的 `doc_exports/` 文件夹

#### 使用方法
```bash
# 使用 Makefile（推荐）
make export-docs

# 或使用 uv 直接运行
uv run python scripts/export_docs.py

# 或使用标准 Python
python scripts/export_docs.py
```

#### 输出结构
脚本会在 `doc_exports/` 目录中创建以下文件：
- `getting-started.md` - 所有入门指南文档
- `user-guide.md` - 所有用户指南内容
- `features.md` - 所有功能文档
- `development.md` - 所有开发文档
- `deployment.md` - 所有部署文档
- `troubleshooting.md` - 所有故障排除文档

#### 输出格式示例
```markdown
# Getting Started

This document consolidates all content from the getting-started documentation folder.

---

## Installation

*Source: installation.md*

[installation.md 的完整内容]

---

## Quick Start

*Source: quick-start.md*

[quick-start.md 的完整内容]

---
```

#### 技术实现
- 使用 `pathlib` 进行路径操作
- 递归扫描子目录
- 自动排除 `index.md` 文件
- 按字母顺序排序文件
- 添加分隔符和源文件引用

## 脚本开发指南

### 1. 命名规范
- 使用小写字母和下划线
- 描述性名称，清晰表达功能
- 添加 `.py` 扩展名

### 2. 文档要求
- 每个脚本都应有详细的文档字符串
- 包含使用示例
- 说明输入输出格式
- 列出依赖项

### 3. 错误处理
- 使用 logging 模块记录日志
- 提供有意义的错误消息
- 优雅处理异常情况
- 验证输入参数

### 4. 代码示例模板
```python
#!/usr/bin/env python3
"""
脚本功能描述

详细说明脚本的作用、使用场景和主要功能。

使用方法:
    python script_name.py [参数]

示例:
    python script_name.py --input file.txt --output result.txt

依赖:
    - 标准库模块
    - 外部依赖（如果有）
"""

import logging
from pathlib import Path
from typing import List, Optional

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format="%(levelname)s: %(message)s"
)
logger = logging.getLogger(__name__)

def main():
    """主函数"""
    logger.info("脚本开始执行...")
    # 实现逻辑
    logger.info("脚本执行完成")

if __name__ == "__main__":
    main()
```

## 最佳实践

### 1. 脚本组织
- 按功能分类组织脚本
- 使用子目录存放相关脚本
- 保持脚本独立和可重用

### 2. 参数处理
- 使用 `argparse` 处理命令行参数
- 提供帮助信息
- 支持常见的参数格式

### 3. 路径处理
- 使用 `pathlib` 而非 `os.path`
- 支持相对和绝对路径
- 处理跨平台路径差异

### 4. 配置管理
- 从环境变量读取配置
- 支持配置文件
- 提供默认值

## 集成到 Makefile

所有脚本都应集成到项目的 Makefile 中，提供便捷的命令：

```makefile
# 导出文档
export-docs:
	@echo "导出文档..."
	uv run python scripts/export_docs.py
	@echo "文档导出完成: doc_exports/"

# 数据库备份
backup-db:
	@echo "备份数据库..."
	uv run python scripts/backup_database.py
	@echo "备份完成"

# 清理临时文件
clean:
	@echo "清理临时文件..."
	uv run python scripts/cleanup.py
	@echo "清理完成"
```

## 未来可能的脚本

### 1. 数据库管理
- `backup_database.py` - 数据库备份
- `restore_database.py` - 数据库恢复
- `migrate_data.py` - 数据迁移
- `cleanup_db.py` - 清理旧数据

### 2. 部署工具
- `deploy.py` - 自动化部署
- `update_version.py` - 版本更新
- `build_images.py` - Docker 镜像构建
- `health_check.py` - 健康检查

### 3. 开发工具
- `generate_models.py` - 生成模型代码
- `update_deps.py` - 更新依赖
- `run_tests.py` - 运行测试套件
- `format_code.py` - 代码格式化

### 4. 监控脚本
- `check_resources.py` - 资源使用监控
- `log_analyzer.py` - 日志分析
- `performance_test.py` - 性能测试

## 相关文件清单

```
scripts/
├── README.md           # 脚本使用说明
├── export_docs.py      # 文档导出工具
└── [future scripts]    # 未来可能添加的脚本
```

## 变更记录 (Changelog)

### 2025-12-09 08:29:13
- 📝 创建脚本工具模块文档
- 📚 详细说明 export_docs.py 功能
- 🎯 提供脚本开发指南
- 📋 列出未来可能的脚本