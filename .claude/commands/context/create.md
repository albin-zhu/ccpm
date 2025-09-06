---
allowed-tools: Bash, Read, Write, LS
---

# 创建初始上下文

此命令通过分析当前项目状态并建立全面的基线文档，在 `.claude/context/` 中创建初始项目上下文文档。

## 必需规则

**重要：** 在执行此命令之前，阅读并遵循：
- `.claude/rules/datetime.md` - 获取真实当前日期/时间

## 预检查清单

在继续之前，完成这些验证步骤。
不要用预检查进度打扰用户（"我不会..."）。直接执行并继续。

### 1. 上下文目录检查
- 运行：`ls -la .claude/context/ 2>/dev/null`
- 如果目录存在且有文件：
  - 统计现有文件：`ls -1 .claude/context/*.md 2>/dev/null | wc -l`
  - 询问用户："⚠️ 发现 {count} 个现有上下文文件。覆盖所有上下文？(yes/no)"
  - 只有明确确认'yes'才继续
  - 如果用户说no，建议："使用 /context:update 刷新现有上下文"

### 2. 项目类型检测
- 检查项目指示器：
  - Node.js：`test -f package.json && echo "Node.js project detected"`
  - Python：`test -f requirements.txt || test -f pyproject.toml && echo "Python project detected"`
  - Rust：`test -f Cargo.toml && echo "Rust project detected"`
  - Go：`test -f go.mod && echo "Go project detected"`
- 运行：`git status 2>/dev/null` 确认这是git仓库
- 如果不是git仓库，询问："⚠️ 不是git仓库。仍然继续？(yes/no)"

### 3. 目录创建
- 如果 `.claude/` 不存在，创建它：`mkdir -p .claude/context/`
- 验证写权限：`touch .claude/context/.test && rm .claude/context/.test`
- 如果权限被拒绝，告诉用户："❌ 无法创建上下文目录。检查权限。"

### 4. 获取当前日期时间
- 运行：`date -u +"%Y-%m-%dT%H:%M:%SZ"`
- 存储此值用于所有上下文文件前置数据

## 指令

### 1. 预分析验证
- 确认项目根目录正确（存在.git、package.json等）
- 检查可以提供上下文信息的现有文档（README.md、docs/）
- 如果README.md不存在，询问用户项目描述

### 2. 系统化项目分析
按此顺序收集信息：

**项目检测：**
- 运行：`find . -maxdepth 2 -name 'package.json' -o -name 'requirements.txt' -o -name 'Cargo.toml' -o -name 'go.mod' 2>/dev/null`
- 运行：`git remote -v 2>/dev/null` 获取仓库信息
- 运行：`git branch --show-current 2>/dev/null` 获取当前分支

**代码库分析：**
- 运行：`find . -type f -name '*.js' -o -name '*.py' -o -name '*.rs' -o -name '*.go' 2>/dev/null | head -20`
- 运行：`ls -la` 查看根目录结构
- 如果存在，读取README.md

### 3. 使用前置数据创建上下文文件

每个上下文文件必须包含真实日期时间的前置数据：

```yaml
---
created: [使用date命令的真实日期时间]
last_updated: [使用date命令的真实日期时间]
version: 1.0
author: Claude Code PM System
---
```

生成以下初始上下文文件：
  - `progress.md` - 记录当前项目状态、已完成工作和即将进行的步骤
    - 包括：当前分支、最近提交、未完成更改
  - `project-structure.md` - 映射目录结构和文件组织
    - 包括：关键目录、文件命名模式、模块组织
  - `tech-context.md` - 编录当前依赖项、技术和开发工具
    - 包括：语言版本、框架版本、开发依赖
  - `system-patterns.md` - 识别现有架构模式和设计决策
    - 包括：观察到的设计模式、架构风格、数据流
  - `product-context.md` - 定义产品需求、目标用户和核心功能
    - 包括：用户画像、核心功能、用例
  - `project-brief.md` - 建立项目范围、目标和关键目的
    - 包括：功能、存在原因、成功标准
  - `project-overview.md` - 提供功能和能力的高级摘要
    - 包括：功能列表、当前状态、集成点
  - `project-vision.md` - 阐述长期愿景和战略方向
    - 包括：未来目标、潜在扩展、战略优先级
  - `project-style-guide.md` - 记录编码标准、约定和样式偏好
    - 包括：命名约定、文件结构模式、注释风格

### 4. 质量验证

创建每个文件后：
- 验证文件创建成功
- 检查文件不为空（最少10行内容）
- 确保前置数据存在且有效
- 验证markdown格式正确

### 5. 错误处理

**常见问题：**
- **无写权限：** "❌ 无法写入 .claude/context/。检查权限。"
- **磁盘空间：** "❌ 上下文文件磁盘空间不足。"
- **文件创建失败：** "❌ 创建 {filename} 失败。错误：{error}"

如果任何文件创建失败：
- 报告成功创建了哪些文件
- 提供继续使用部分上下文的选项
- 永远不要留下损坏或不完整的文件

### 6. 创建后摘要

提供全面摘要：
```
📋 上下文创建完成

📁 在此创建上下文：.claude/context/
✅ 已创建文件：{count}/9

📊 上下文摘要：
  - 项目类型：{detected_type}
  - 语言：{primary_language}
  - Git状态：{clean/changes}
  - 依赖：{count} 个包

📝 文件详情：
  ✅ progress.md ({lines} 行) - 当前状态和最近工作
  ✅ project-structure.md ({lines} 行) - 目录组织
  [... 列出所有文件及行数和简要描述 ...]

⏰ 创建时间：{timestamp}
🔄 下一步：在新会话中使用 /context:prime 加载上下文
💡 提示：定期运行 /context:update 保持上下文最新
```

## 上下文收集命令

使用这些命令获取项目信息：
- 目标目录：`.claude/context/`（如需要则创建）
- 当前git状态：`git status --short`
- 最近提交：`git log --oneline -10`
- 项目README：如存在则读取 `README.md`
- 包文件：检查 `package.json`、`requirements.txt`、`Cargo.toml`、`go.mod` 等
- 文档扫描：`find . -type f -name '*.md' -path '*/docs/*' 2>/dev/null | head -10`
- 测试检测：`find . -type d \( -name 'test' -o -name 'tests' -o -name '__tests__' -o -name 'spec' \) 2>/dev/null | head -5`

## 重要说明

- **始终使用真实日期时间**从系统时钟，不要占位符
- **覆盖现有上下文前询问确认**
- **验证每个文件**创建成功
- **提供详细摘要**创建了什么
- **优雅处理错误**并提供具体指导

$ARGUMENTS