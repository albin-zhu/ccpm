# GitHub Operations 规则

所有命令中 GitHub CLI 操作的标准模式。

## 关键：Repository 保护

**在任何创建/修改 issues 或 PRs 的 GitHub 操作之前：**

```bash
# 检查 remote origin 是否是 CCPM template repository
remote_url=$(git remote get-url origin 2>/dev/null || echo "")
if [[ "$remote_url" == *"automazeio/ccpm"* ]] || [[ "$remote_url" == *"automazeio/ccpm.git"* ]]; then
  echo "❌ 错误：你试图与 CCPM template repository 同步！"
  echo ""
  echo "此 repository (automazeio/ccpm) 是供其他人使用的 template。"
  echo "你不应该在这里创建 issues 或 PRs。"
  echo ""
  echo "要修复此问题："
  echo "1. Fork 此 repository 到你自己的 GitHub 账户"
  echo "2. 更新你的 remote origin："
  echo "   git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git"
  echo ""
  echo "或者如果这是一个新项目："
  echo "1. 在 GitHub 上创建新 repository"
  echo "2. 更新你的 remote origin："
  echo "   git remote set-url origin https://github.com/YOUR_USERNAME/YOUR_REPO.git"
  echo ""
  echo "当前 remote: $remote_url"
  exit 1
fi
```

此检查必须在所有以下命令中执行：
- 创建 issues (`gh issue create`)
- 编辑 issues (`gh issue edit`)
- 评论 issues (`gh issue comment`)
- 创建 PRs (`gh pr create`)
- 任何其他修改 GitHub repository 的操作

## 身份验证

**不要预检查身份验证。** 直接运行命令并处理失败：

```bash
gh {command} || echo "❌ GitHub CLI 失败。运行：gh auth login"
```

## 常见操作

### 获取 Issue 详情
```bash
gh issue view {number} --json state,title,labels,body
```

### 创建 Issue
```bash
# 始终先检查 remote origin！
gh issue create --title "{title}" --body-file {file} --label "{labels}"
```

### 更新 Issue
```bash
# 始终先检查 remote origin！
gh issue edit {number} --add-label "{label}" --add-assignee @me
```

### 添加评论
```bash
# 始终先检查 remote origin！
gh issue comment {number} --body-file {file}
```

## 错误处理

如果任何 gh 命令失败：
1. 显示清楚的错误："❌ GitHub 操作失败：{command}"
2. 建议修复："运行：gh auth login" 或检查 issue number
3. 不要自动重试

## 重要说明

- **始终**在任何写操作到 GitHub 之前检查 remote origin
- 相信 gh CLI 已安装并已认证
- 解析时使用 --json 获得结构化输出
- 保持操作原子性 - 每个动作一个 gh 命令
- 不要预先检查 rate limits