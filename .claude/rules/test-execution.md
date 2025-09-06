# Test Execution 规则

所有测试命令的测试运行标准模式。

## 核心原则

1. **始终使用 test-runner agent** 来自 `.claude/agents/test-runner.md`
2. **No mocking** - 使用真实服务获得准确结果
3. **Verbose output** - 捕获所有内容用于调试
4. **先检查测试结构** - 在假设代码 bug 之前

## 执行模式

```markdown
执行测试用于：{target}

要求：
- 用 verbose output 运行
- 无 mock services
- 捕获完整 stack traces
```

遵循这些模式确保测试结果可靠和可调试。
