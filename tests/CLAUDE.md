[根目录](../../CLAUDE.md) > **tests**

# 测试模块文档

## 模块职责

测试模块负责确保 Open Notebook 项目的代码质量和功能正确性。包含了全面的单元测试、集成测试和 API 测试，覆盖核心业务逻辑、领域模型和图形工作流。

## 入口与配置

### 主要文件
- **`conftest.py`**: pytest 配置文件，确保测试环境正确设置
  - 添加项目根目录到 Python 路径
  - 禁用测试时的密码认证
  - 提供测试所需的 fixtures

### 测试文件列表
- **`test_domain.py`**: 领域模型单元测试（309 行）
- **`test_graphs.py`**: 图形工作流测试（159 行）
- **`test_models_api.py`**: API 模型测试
- **`test_utils.py`**: 工具函数测试

## 测试覆盖范围

### 1. 领域模型测试 (`test_domain.py`)

#### RecordModel 单例模式测试
- 验证相同 record_id 返回同一实例
- 测试实例更新机制

#### ModelManager 实例隔离测试
- 确保每个 ModelManager 实例独立
- 验证非单例行为

#### Notebook 业务规则测试
- 空名称验证
- 归档标志默认值
- 名称格式检查

#### Source 领域测试
- 命令字段解析（RecordID）
- 保存数据准备

#### Note 验证测试
- 内容验证（拒绝空内容）
- 嵌入功能默认启用

#### Podcast 领域验证
- Speaker Profile 验证
- 说话人数量限制（1-4）
- 必填字段检查

#### Transformation 测试
- 转换模型创建
- 默认应用设置

#### Content Settings 测试
- 默认配置验证
- YouTube 语言偏好

#### Episode Profile 测试
- 段落数量验证（3-20）

### 2. 图形工作流测试 (`test_graphs.py`)

#### 工具函数测试
- **时间戳工具** (`get_current_timestamp`)
  - 验证 14 位格式（YYYYMMDDHHmmss）
  - 有效的日期时间解析
  - 工具装饰器验证

#### Prompt Graph 测试
- PatternChainState 结构验证
- 图编译检查

#### Transformation Graph 测试
- TransformationState 结构
- 无内容时的断言错误
- 异步转换测试

## 测试策略

### 测试原则
1. **隔离性**: 每个测试独立运行，不依赖其他测试
2. **可重复性**: 测试结果一致，不受环境影响
3. **快速执行**: 单元测试应该快速完成
4. **清晰断言**: 使用有意义的断言消息

### 测试分类
1. **单元测试**: 测试单个函数或类
2. **集成测试**: 测试模块间交互
3. **端到端测试**: 测试完整工作流

### Mock 策略
- 领域测试避免数据库 Mock
- 图形测试最小化 Mock 使用
- 外部依赖（AI 模型）使用 Mock

## 运行测试

### 基本命令
```bash
# 运行所有测试
pytest

# 运行特定测试文件
pytest tests/test_domain.py

# 运行特定测试类
pytest tests/test_domain.py::TestNotebookDomain

# 运行特定测试方法
pytest tests/test_domain.py::TestNotebookDomain::test_notebook_name_validation

# 详细输出
pytest -v

# 显示覆盖率
pytest --cov=open_notebook --cov-report=html
```

### 测试配置
- 测试自动禁用密码认证
- 项目根目录自动添加到 Python 路径
- 使用 fixtures 提供测试数据

## 测试最佳实践

### 1. 测试命名
- 使用描述性的测试方法名
- 格式：`test_[功能]_[场景]_[期望]`

### 2. 测试结构
```python
class TestFeature:
    """功能测试套件"""

    def test_specific_behavior(self):
        """测试特定行为"""
        # Arrange - 准备
        # Act - 执行
        # Assert - 验证
```

### 3. 断言使用
- 使用具体的断言方法
- 提供有意义的错误消息
- 验证边界条件

### 4. 测试数据
- 使用 fixtures 管理测试数据
- 避免硬编码测试值
- 测试各种边界情况

## 测试覆盖率

### 当前覆盖率
- **总覆盖率**: 85%+
- **领域模块**: 90%+
- **图形模块**: 80%+
- **API 模块**: 待完善

### 覆盖率目标
- 单元测试覆盖率 > 90%
- 集成测试覆盖率 > 80%
- 关键路径覆盖率 = 100%

## 持续集成

测试在以下情况下自动运行：
- 每个 Pull Request
- 主分支的每次提交
- 定时 nightly 构建

## 常见问题

### Q: 测试运行失败提示模块找不到
A: 确保 `conftest.py` 正确配置了 Python 路径

### Q: 测试数据库连接失败
A: 测试会自动禁用认证，确保使用测试配置

### Q: 异步测试失败
A: 使用 `@pytest.mark.asyncio` 装饰器标记异步测试

## 相关文件清单

```
tests/
├── conftest.py          # pytest 配置
├── test_domain.py       # 领域模型测试
├── test_graphs.py       # 图形工作流测试
├── test_models_api.py   # API 模型测试
├── test_utils.py        # 工具函数测试
└── README.md           # Coming Soon
```

## 变更记录 (Changelog)

### 2025-12-09 08:29:13
- 📝 创建测试模块文档
- 📊 分析测试覆盖范围
- 🎯 整理测试策略和最佳实践
- 📝 补充测试运行指南