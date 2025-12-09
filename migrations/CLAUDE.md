[根目录](../../CLAUDE.md) > **migrations**

# 数据库迁移模块

## 模块职责

数据库迁移模块负责管理 SurrealDB 的数据库 schema 演进。通过版本化的迁移文件，确保数据库结构的一致性和可追踪性，支持系统的平滑升级和数据迁移。

## 入口与结构

### 目录结构
```
migrations/
├── 1.surrealql          ->  1_down.surrealql    # 初始化 schema
├── 2.surrealql          ->  2_down.surrealql    # 扩展功能
├── 3.surrealQL          ->  3_down.surrealQL    # 索引优化
├── 4.surrealQL          ->  4_down.surrealQL    # 功能增强
├── 5.surrealQL          ->  5_down.surrealQL    # 性能优化
├── 6.surrealQL          ->  6_down.surrealQL    # 数据结构调整
├── 7.surrealQL          ->  7_down.surrealQL    # 安全增强
├── 8.surrealQL          ->  8_down.surrealQL    # 功能扩展
└── 9.surrealQL          ->  9_down.surrealQL    # 搜索优化
```

### 命名规范
- `N.surrealql` - 向上迁移脚本
- `N_down.surrealql` - 向下回滚脚本
- `N` 为递增的版本号

## 迁移文件详解

### 1. 初始化 Schema (v1)

#### 主要内容
- 定义核心表结构
- 设置基础字段
- 创建关系表
- 配置搜索索引

#### 核心表定义

##### source 表
```sql
DEFINE TABLE IF NOT EXISTS source SCHEMAFULL;
DEFINE FIELD IF NOT EXISTS title ON TABLE source TYPE option<string>;
DEFINE FIELD IF NOT EXISTS topics ON TABLE source TYPE option<array<string>>;
DEFINE FIELD IF NOT EXISTS full_text ON TABLE source TYPE option<string>;
```

##### source_embedding 表
```sql
DEFINE TABLE IF NOT EXISTS source_embedding SCHEMAFULL;
DEFINE FIELD IF NOT EXISTS source ON TABLE source_embedding TYPE record<source>;
DEFINE FIELD IF NOT EXISTS content ON TABLE source_embedding TYPE string;
DEFINE FIELD IF NOT EXISTS embedding ON TABLE source_embedding TYPE array<float>;
```

##### note 表
```sql
DEFINE TABLE IF NOT EXISTS note SCHEMAFULL;
DEFINE FIELD IF NOT EXISTS title ON TABLE note TYPE option<string>;
DEFINE FIELD IF NOT EXISTS content ON TABLE note TYPE option<string>;
DEFINE FIELD IF NOT EXISTS embedding ON TABLE note TYPE array<float>;
```

##### notebook 表
```sql
DEFINE TABLE IF NOT EXISTS notebook SCHEMAFULL;
DEFINE FIELD IF NOT EXISTS name ON TABLE notebook TYPE option<string>;
DEFINE FIELD IF NOT EXISTS description ON TABLE notebook TYPE option<string>;
DEFINE FIELD IF NOT EXISTS archived ON TABLE notebook TYPE option<bool> DEFAULT False;
```

#### 关系定义
```sql
-- source 到 notebook 的引用关系
DEFINE TABLE IF NOT EXISTS reference
TYPE RELATION FROM source TO notebook;

-- note 到 notebook 的归属关系
DEFINE TABLE IF NOT EXISTS artifact
TYPE RELATION FROM note TO notebook;
```

#### 搜索索引
```sql
DEFINE ANALYZER IF NOT EXISTS my_analyzer
TOKENIZERS blank,class,camel,punct
FILTERS snowball(english), lowercase;

DEFINE INDEX IF NOT EXISTS idx_source_title ON TABLE source
COLUMNS title SEARCH ANALYZER my_analyzer BM25 HIGHLIGHTS;
```

### 2. 函数定义 (v1)

#### 文本搜索函数
```sql
DEFINE FUNCTION IF NOT EXISTS fn::text_search(
    $query_text: string,
    $match_count: int,
    $sources:bool,
    $show_notes:bool
) {
    -- 实现多表文本搜索
    -- 支持来源和笔记搜索
    -- 返回相关性排序结果
};
```

#### 向量搜索函数（v1 初始版本）
```sql
DEFINE FUNCTION IF NOT EXISTS fn::vector_search(
    $query: array<float>,
    $match_count: int,
    $sources:bool,
    $show_notes:bool
) {
    -- 基础向量相似度搜索
    -- 支持来源嵌入和笔记嵌入
};
```

### 3. 关键演进 (v9 - 向量搜索优化)

#### 改进的向量搜索函数
v9 版本对向量搜索进行了重要优化：

```sql
REMOVE FUNCTION IF EXISTS fn::vector_search;

DEFINE FUNCTION IF NOT EXISTS fn::vector_search(
    $query: array<float>,
    $match_count: int,
    $sources: bool,
    $show_notes: bool,
    $min_similarity: float  -- 新增：最小相似度阈值
) {
    -- 优化内容：
    -- 1. 添加相似度阈值过滤
    -- 2. 改进结果格式（包含标题和父ID）
    -- 3. 优化性能（提前过滤无效嵌入）
    -- 4. 更好的结果聚合
};
```

#### 主要改进
1. **相似度阈值**：避免返回低相关结果
2. **结构化输出**：返回更丰富的元数据
3. **性能优化**：减少不必要的计算
4. **错误处理**：验证嵌入维度

## 迁移执行

### 1. 自动迁移
系统启动时会自动检查并执行待处理的迁移。

### 2. 手动迁移
```sql
-- 执行特定迁移
-- 将迁移文件内容直接在 SurrealDB 中执行
```

### 3. 迁移状态跟踪
```sql
-- 查看当前数据库版本
SELECT * FROM migrations;

-- 或检查特定表是否存在
INFO FOR TABLE source;
```

## 迁移最佳实践

### 1. 向前兼容
- 新增字段使用 `IF NOT EXISTS`
- 修改字段前创建备份
- 保留旧字段一段时间

### 2. 向后回滚
每个迁移都应有对应的回滚脚本：
```sql
-- 示例：回滚 v9
REMOVE FUNCTION fn::vector_search;

-- 恢复旧版本函数
DEFINE FUNCTION fn::vector_search(...) {
    -- v8 版本实现
};
```

### 3. 数据安全
- 执行前备份数据
- 使用事务确保一致性
- 验证迁移结果

### 4. 性能考虑
- 大数据量迁移分批执行
- 避免高峰期执行
- 监控迁移进度

## 迁移内容总结

### 表结构演进
1. **v1**: 基础表结构（source, note, notebook）
2. **v2**: 扩展字段和关系
3. **v3**: 索引优化
4. **v4**: 功能增强
5. **v5**: 性能优化
6. **v6**: 数据结构调整
7. **v7**: 安全增强
8. **v8**: 功能扩展
9. **v9**: 搜索优化（向量搜索改进）

### 索引演进
- BM25 全文搜索索引
- 向量搜索优化
- 多语言分析器
- 高亮显示支持

### 函数演进
- 文本搜索功能完善
- 向量搜索性能优化
- 结果格式标准化
- 相似度阈值控制

## 故障处理

### 常见问题

#### 1. 迁移失败
- 检查 SQL 语法
- 验证权限设置
- 查看错误日志

#### 2. 性能问题
- 分析慢查询
- 优化索引
- 考虑分批处理

#### 3. 数据不一致
- 运行一致性检查
- 修复脚本
- 必要时回滚

## 相关文件清单

```
migrations/
├── 1.surrealql          -> 1_down.surrealql
├── 2.surrealql          -> 2_down.surrealql
├── 3.surrealQL          -> 3_down.surrealQL
├── 4.surrealQL          -> 4_down.surrealQL
├── 5.surrealQL          -> 5_down.surrealQL
├── 6.surrealQL          -> 6_down.surrealQL
├── 7.surrealQL          -> 7_down.surrealQL
├── 8.surrealQL          -> 8_down.surrealQL
└── 9.surrealQL          -> 9_down.surrealQL
```

## 变更记录 (Changelog)

### 2025-12-09 08:29:13
- 📝 创建数据库迁移模块文档
- 📚 分析 9 个版本的迁移内容
- 🎯 重点说明 v9 向量搜索优化
- 🔧 补充迁移最佳实践