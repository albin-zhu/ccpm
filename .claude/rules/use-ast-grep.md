# AST-Grep Integration Protocol for Cursor Agent

## 何时使用 AST-Grep

在以下情况使用 `ast-grep`（如果已安装）而不是普通 regex 或文本搜索：

- **结构化代码模式**涉及（例如，查找所有 function calls、class definitions 或 method implementations）
- **语言感知重构**需要（例如，重命名变量、更新 function signatures 或更改 imports）
- **复杂代码分析**需要（例如，跨不同语法上下文查找模式的所有用法）
- **跨语言搜索**必要（例如，在 monorepo 中同时使用 Ruby 和 TypeScript）
- **语义代码理解**重要（例如，基于代码结构而非文本查找模式）

## AST-Grep Command 模式

### 基本搜索模板：
```sh
ast-grep --pattern '$PATTERN' --lang $LANGUAGE $PATH
```

这提供了强大的结构化代码搜索和重构能力。
