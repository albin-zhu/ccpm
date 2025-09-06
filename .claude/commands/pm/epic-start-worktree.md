---
allowed-tools: Bash, Read, Write, LS, Task
---

# Epic Start

启动并行 agents 在共享 worktree 中处理 epic tasks。

## 使用方法
```
/pm:epic-start <epic_name>
```

## 快速检查

1. **验证 epic 存在：**
   ```bash
   test -f .claude/epics/$ARGUMENTS/epic.md || echo "❌ Epic 未找到。运行：/pm:prd-parse $ARGUMENTS"
   ```

2. **检查 GitHub 同步：**
   在 epic frontmatter 中查找 `github:` 字段。
   如果缺失："❌ Epic 未同步。首先运行：/pm:epic-sync $ARGUMENTS"

3. **检查 worktree：**
   ```bash
   git worktree list | grep "epic-$ARGUMENTS"
   ```

## 指令

### 1. 创建或进入 Worktree

遵循 `/rules/worktree-operations.md`：

```bash
# 如果 worktree 不存在，创建它
if ! git worktree list | grep -q "epic-$ARGUMENTS"; then
  git checkout main
  git pull origin main
  git worktree add ../epic-$ARGUMENTS -b epic/$ARGUMENTS
  echo "✅ 创建 worktree：../epic-$ARGUMENTS"
else
  echo "✅ 使用现有 worktree：../epic-$ARGUMENTS"
fi
```

### 2. 识别就绪 Issues

读取 `.claude/epics/$ARGUMENTS/` 中的所有 task 文件：
- 解析 frontmatter 中的 `status`、`depends_on`、`parallel` 字段
- 如需要检查 GitHub issue 状态
- 构建依赖图

分类 issues：
- **Ready**：无未满足依赖，未开始
- **Blocked**：有未满足依赖
- **In Progress**：已在进行中
- **Complete**：已完成

### 3. 分析就绪 Issues

对于每个没有分析的就绪 issue：
```bash
# 检查分析
if ! test -f .claude/epics/$ARGUMENTS/{issue}-analysis.md; then
  echo "分析 issue #{issue}..."
  # 运行分析（内联或通过 Task tool）
fi
```

### 4. 启动并行 Agents

对于每个有分析的就绪 issue：

```markdown
## 开始 Issue #{issue}：{title}

读取分析...
发现 {count} 个并行 streams：
  - Stream A：{description}（Agent-{id}）
  - Stream B：{description}（Agent-{id}）

在 worktree 中启动 agents：../epic-$ARGUMENTS/
```

使用 Task tool 启动每个 stream：
```yaml
Task:
  description: "Issue #{issue} Stream {X}"
  subagent_type: "{agent_type}"
  prompt: |
    在 worktree 中工作：../epic-$ARGUMENTS/
    Issue：#{issue} - {title}
    Stream：{stream_name}

    你的范围：
    - 文件：{file_patterns}
    - 工作：{stream_description}

    从以下读取完整需求：
    - .claude/epics/$ARGUMENTS/{task_file}
    - .claude/epics/$ARGUMENTS/{issue}-analysis.md

    遵循 /rules/agent-coordination.md 中的协调规则

    使用消息格式频繁提交：
    "Issue #{issue}: {specific change}"

    在以下更新进度：
    .claude/epics/$ARGUMENTS/updates/{issue}/stream-{X}.md
```

### 5. 跟踪活跃 Agents

创建/更新 `.claude/epics/$ARGUMENTS/execution-status.md`：

```markdown
---
started: {datetime}
worktree: ../epic-$ARGUMENTS
branch: epic/$ARGUMENTS
---

# 执行状态

## 活跃 Agents
- Agent-1：Issue #1234 Stream A（Database）- 开始 {time}
- Agent-2：Issue #1234 Stream B（API）- 开始 {time}
- Agent-3：Issue #1235 Stream A（UI）- 开始 {time}

## 队列中的 Issues
- Issue #1236 - 等待 #1234
- Issue #1237 - 等待 #1235

## 已完成
- {暂无}
```

### 6. 监控和协调

设置监控：
```bash
echo "
Agents 启动成功！

监控进度：
  /pm:epic-status $ARGUMENTS

查看 worktree 更改：
  cd ../epic-$ARGUMENTS && git status

停止所有 agents：
  /pm:epic-stop $ARGUMENTS

完成时合并：
  /pm:epic-merge $ARGUMENTS
"
```

### 7. 处理依赖

当 agents 完成 streams：
- 检查是否有被阻塞的 issues 现在就绪
- 为新就绪的工作启动新 agents
- 更新 execution-status.md

## 输出格式

```
🚀 Epic 执行已开始：$ARGUMENTS

Worktree：../epic-$ARGUMENTS
Branch：epic/$ARGUMENTS

在 {issue_count} 个 issues 中启动 {total} 个 agents：

Issue #1234：Database Schema
  ├─ Stream A：Schema creation（Agent-1）✓ 已开始
  └─ Stream B：Migrations（Agent-2）✓ 已开始

Issue #1235：API Endpoints
  ├─ Stream A：User endpoints（Agent-3）✓ 已开始
  ├─ Stream B：Post endpoints（Agent-4）✓ 已开始
  └─ Stream C：Tests（Agent-5）⏸ 等待 A & B

被阻塞的 Issues（2）：
  - #1236：UI Components（依赖 #1234）
  - #1237：Integration（依赖 #1235、#1236）

监控命令：/pm:epic-status $ARGUMENTS
```

## 错误处理

如果 agent 启动失败：
```
❌ 启动 Agent-{id} 失败
  Issue：#{issue}
  Stream：{stream}
  错误：{reason}

继续其他 agents？（yes/no）
```

如果 worktree 创建失败：
```
❌ 无法创建 worktree
  {git error message}

尝试：git worktree prune
或：使用以下检查现有 worktrees：git worktree list
```

## 重要说明

- 遵循 `/rules/worktree-operations.md` 进行 git 操作
- 遵循 `/rules/agent-coordination.md` 进行并行工作
- Agents 在相同 worktree 中工作（不是分离的）
- 最大并行 agents 应该合理（例如 5-10）
- 如果启动很多 agents 要监控系统资源