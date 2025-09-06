# Strip Frontmatter

在发送内容到 GitHub 前移除 YAML frontmatter 的标准方法。

## 问题

YAML frontmatter 包含不应出现在 GitHub issues 中的内部元数据：
- status、created、updated 字段
- 内部引用和 IDs
- 本地文件路径

## 解决方案

使用 sed 从任何 markdown 文件中剥离 frontmatter：

```bash
# 剥离 frontmatter（前两个 --- 行之间的所有内容）
sed '1,/^---$/d; 1,/^---$/d' input.md > output.md
```

这确保只有纯内容被发送到 GitHub。
