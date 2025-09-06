# Worktree Operations

Git worktrees 通过允许同一 repository 的多个工作目录来实现并行开发。

## 创建 Worktrees

始终从干净的 main branch 创建 worktrees：
```bash
# 确保 main 是最新的
git checkout main
git pull origin main

# 为 epic 创建 worktree
git worktree add ../epic-{name} -b epic/{name}
```

worktree 将作为兄弟目录创建以保持清洁分离。

## 在 Worktrees 中工作

每个 worktree 都是独立的工作环境：
- 可以有不同的 branches 被 checkout
- 独立的暂存区和工作目录
- 共享相同的 git history 和 remotes

使用 worktrees 进行大型功能开发和并行工作流程。
