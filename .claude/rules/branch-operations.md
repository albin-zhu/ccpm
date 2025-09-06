# Branch Operations

Git branches 通过允许多个开发者在同一 repository 上进行隔离更改的并行开发来实现。

## 创建 Branches

始终从干净的 main branch 创建 branches：
```bash
# 确保 main 是最新的
git checkout main
git pull origin main

# 为 epic 创建 branch
git checkout -b epic/{name}
git push -u origin epic/{name}
```

branch 将被创建并推送到 origin 并设置 upstream tracking。

## 在 Branches 中工作

在 branch 中工作时应该：
- 保持 commits 专注和原子化
- 定期与 main 同步
- 使用清晰的 commit 消息
- 在推送前进行本地测试
