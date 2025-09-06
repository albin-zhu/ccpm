# Agents

专门的 agents 执行繁重工作并返回简洁摘要以保存 context。

## 核心理念

> "不要将 subagents 拟人化。使用它们来组织你的 prompts 并省略 context。Subagents 在能够做大量工作但只向主对话线程提供少量信息时效果最好。"
>
> – Adam Wolff, Anthropic

## 可用 Agents

### 🔍 `code-analyzer`
- **目的**：在不污染主 context 的情况下跨多个文件寻找 bugs
- **模式**：搜索多个文件 → 分析代码 → 返回 bug 报告
- **使用场景**：当你需要追踪逻辑流、找 bugs 或验证更改时
- **返回**：仅包含关键发现的简洁 bug 报告

### 📄 `file-analyzer`
- **目的**：读取和总结冗长文件（logs、outputs、configs）
- **模式**：读取文件 → 提取见解 → 返回摘要
- **使用场景**：当你需要理解 log 文件或分析冗长输出时
- **返回**：关键发现和可操作见解（80-90% 大小减少）

### 🧪 `test-runner`
- **目的**：执行测试而不将输出转储到主线程
- **模式**：运行测试 → 捕获到 log → 分析结果 → 返回摘要
- **使用场景**：当你需要运行测试并了解失败时
- **返回**：带有失败分析的测试结果摘要

### 🔀 `parallel-worker`
- **目的**：为一个 issue 协调多个并行工作流
- **模式**：读取分析 → spawn sub-agents → 合并结果 → 返回摘要
- **使用场景**：在 worktree 中执行并行工作流时
- **返回**：所有并行工作的合并状态

## 为什么使用 Agents？

Agents 是**context 防火墙**，保护主对话免受信息过载：

```
没有 Agent：
主线程读取10个文件 → Context 爆炸 → 失去连贯性

有 Agent：
Agent 读取10个文件 → 主线程获得1个摘要 → Context 保持
```

## Agents 如何保存 Context

1. **繁重工作** - Agents 做杂乱工作（读取文件、运行测试、实现功能）
2. **Context 隔离** - 实现细节保留在 agent 中，不在主线程
3. **简洁返回** - 只有重要信息返回到主对话
4. **并行执行** - 多个 agents 可以同时工作而不会 context 冲突

## 使用示例

```bash
# 分析代码查找 bugs
Task: "搜索代码库中的内存泄漏"
Agent: code-analyzer
Returns: "发现3个潜在泄漏：[简洁列表]"
主线程永远看不到：检查的数百个文件

# 运行测试
Task: "运行认证测试"
Agent: test-runner
Returns: "2/10 测试失败：[失败摘要]"
主线程永远看不到：冗长的测试输出和 logs

# 并行实现
Task: "使用并行 streams 实现 issue #1234"
Agent: parallel-worker
Returns: "完成 4/4 streams，修改15个文件"
主线程永远看不到：个别实现细节
```

## 创建新 Agents

新 agents 应遵循这些原则：

1. **单一目的** - 每个 agent 有一个明确工作
2. **Context 减少** - 返回处理内容的 10-20%
3. **无角色扮演** - Agents 不是"专家"，它们是任务执行者
4. **清晰模式** - 定义输入 → 处理 → 输出模式
5. **错误处理** - 优雅处理失败并清楚报告

## 要避免的反模式

❌ **创建"专家" agents**（database-expert、api-expert）
   Agents 没有不同知识 - 它们都是同一个 model

❌ **返回冗长输出**
   违背了 context 保存的目的

❌ **让 agents 互相通信**
   使用协调器 agent 代替（如 parallel-worker）

❌ **对简单任务使用 agents**
   只有当 context 减少有价值时才使用 agents

## 与 PM System 集成

Agents 与 PM 命令系统无缝集成：

- `/pm:issue-analyze` → 识别工作流
- `/pm:issue-start` → spawns parallel-worker agent
- parallel-worker → spawns 多个 sub-agents
- Sub-agents → 在 worktree 中并行工作
- 结果 → 合并回主线程

这创建了一个层次结构，在每个级别最大化并行性的同时保存 context。