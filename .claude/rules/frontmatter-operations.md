# Frontmatter Operations 规则

在 markdown 文件中使用 YAML frontmatter 的标准模式。

## 读取 Frontmatter

从任何 markdown 文件提取 frontmatter：
1. 在文件开始处寻找 `---` 标记之间的内容
2. 解析为 YAML
3. 如果无效或缺失，使用合理默认值

## 更新 Frontmatter

更新现有文件时：
1. 保留所有现有字段
2. 只更新指定字段
3. 始终用当前 datetime 更新 `updated` 字段（参见 `/rules/datetime.md`）

## 标准字段

### 所有文件
```yaml
---
name: {identifier}
created: {ISO datetime}      # 创建后永不更改
updated: {ISO datetime}      # 任何修改时更新
---
```

### Status 值
- PRDs：`backlog`、`in-progress`、`complete`
- Epics：`backlog`、`in-progress`、`completed`  
- Tasks：`open`、`in-progress`、`closed`

### Progress 跟踪
```yaml
progress: {0-100}%           # 用于 epics
completion: {0-100}%         # 用于 progress 文件
```

## 创建新文件

创建 markdown 文件时始终包含 frontmatter：
```yaml
---
name: {from_arguments_or_context}
status: {initial_status}
created: {current_datetime}
updated: {current_datetime}
---
```

## 重要说明

- 初始创建后永不修改 `created` 字段
- 始终使用系统的真实 datetime（参见 `/rules/datetime.md`）
- 尝试解析前验证 frontmatter 存在
- 在所有文件中使用一致的字段名称