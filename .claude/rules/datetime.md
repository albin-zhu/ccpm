# DateTime 规则

## 获取当前日期和时间

当任何命令需要当前日期/时间（用于 frontmatter、timestamps 或 logs）时，你必须从系统获取真实的当前日期/时间，而不是估计或使用占位符值。

### 如何获取当前 DateTime

使用 `date` 命令获取当前 ISO 8601 格式的 datetime：

```bash
# 获取 ISO 8601 格式的当前 datetime（适用于 Linux/Mac）
date -u +"%Y-%m-%dT%H:%M:%SZ"

# 支持的系统的替代方法
date --iso-8601=seconds

# Windows（如果使用 PowerShell）
Get-Date -Format "yyyy-MM-ddTHH:mm:ssZ"
```

### 必需格式

frontmatter 中的所有日期必须使用 UTC timezone 的 ISO 8601 格式：
- 格式：`YYYY-MM-DDTHH:MM:SSZ`
- 示例：`2024-01-15T14:30:45Z`

### 在 Frontmatter 中的使用

在任何文件（PRD、Epic、Task、Progress）中创建或更新 frontmatter 时，始终使用真实的当前 datetime：

```yaml
---
name: feature-name
created: 2024-01-15T14:30:45Z  # 使用 date 命令的实际输出
updated: 2024-01-15T14:30:45Z  # 使用 date 命令的实际输出
---
```

### 实现指令

1. **在编写任何带有 frontmatter 的文件之前：**
   - 运行：`date -u +"%Y-%m-%dT%H:%M:%SZ"`
   - 存储输出
   - 在 frontmatter 中使用这个确切值

2. **对于创建文件的命令：**
   - PRD 创建：为 `created` 字段使用真实日期
   - Epic 创建：为 `created` 字段使用真实日期
   - Task 创建：为 `created` 和 `updated` 字段都使用真实日期
   - Progress 跟踪：为 `started` 和 `last_sync` 字段使用真实日期

3. **对于更新文件的命令：**
   - 始终用当前真实 datetime 更新 `updated` 字段
   - 保留原始 `created` 字段
   - 对于 sync 操作，用真实 datetime 更新 `last_sync`

### 示例

**创建新 PRD：**
```bash
# 首先，获取当前 datetime
CURRENT_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
# 输出：2024-01-15T14:30:45Z

# 然后在 frontmatter 中使用：
---
name: user-authentication
description: User authentication and authorization system
status: backlog
created: 2024-01-15T14:30:45Z  # 使用实际的 $CURRENT_DATE 值
---
```

**更新现有 task：**
```bash
# 获取用于更新的当前 datetime
UPDATE_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

# 只更新 'updated' 字段：
---
name: implement-login-api
status: in-progress
created: 2024-01-10T09:15:30Z  # 保持原始
updated: 2024-01-15T14:30:45Z  # 使用新的 $UPDATE_DATE 值
---
```

### 重要说明

- **永远不要使用占位符日期**，如 `[Current ISO date/time]` 或 `YYYY-MM-DD`
- **永远不要估计日期** - 始终获取实际系统时间
- **始终使用 UTC**（`Z` 后缀）以保持跨时区一致性
- **保持时区一致性** - 系统中的所有日期都使用 UTC

### 跨平台兼容性

如果需要确保不同系统的兼容性：

```bash
# 首先尝试主要方法
date -u +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null || \
# 没有 -u 标志的系统的回退
date +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null || \
# 最后手段：如果可用使用 Python
python3 -c "from datetime import datetime; print(datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ'))" 2>/dev/null || \
python -c "from datetime import datetime; print(datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ'))" 2>/dev/null
```

## 规则优先级

此规则具有**最高优先级**，必须被所有以下命令遵循：
- 创建带有 frontmatter 的新文件
- 更新带有 frontmatter 的现有文件
- 跟踪 timestamps 或 progress
- 记录任何基于时间的信息

受影响的命令：prd-new、prd-parse、epic-decompose、epic-sync、issue-start、issue-sync，以及任何其他写入 timestamps 的命令。