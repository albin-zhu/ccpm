# CLAUDE.md

> 仔细思考并实现最简洁的解决方案，尽可能少地改变代码。

## 使用 SUB-AGENT 进行 CONTEXT 优化

### 1. 当被要求读取文件时，始终使用 file-analyzer sub-agent
file-analyzer agent 是提取和总结文件关键信息的专家，特别是 log 文件和冗长输出。它提供简洁、可操作的摘要，在显著减少 context 使用的同时保留重要信息。

### 2. 当被要求搜索代码、分析代码、研究 bug 或追踪逻辑流时，始终使用 code-analyzer sub-agent

code-analyzer agent 是代码分析、逻辑追踪和漏洞检测的专家。它提供简洁、可操作的摘要，在显著减少 context 使用的同时保留重要信息。

### 3. 始终使用 test-runner sub-agent 来运行测试和分析测试结果

使用 test-runner agent 确保：

- 完整的测试输出被捕获用于调试
- 主对话保持清洁和专注
- Context 使用得到优化
- 所有问题都被正确浮现
- 没有审批对话框中断工作流程

## Philosophy

### Error Handling

- **Fail fast** 用于关键配置（缺少 text model）
- **Log and continue** 用于可选功能（extraction model）
- **Graceful degradation** 当外部服务不可用时
- **User-friendly messages** 通过 resilience layer

### Testing

- 始终使用 test-runner agent 执行测试。
- 永远不要对任何东西使用 mock services。
- 在当前测试完成之前不要继续下一个测试。
- 如果测试失败，在决定我们需要重构 codebase 之前，考虑检查测试是否正确构造。
- 测试应该是 verbose 的，这样我们可以用它们进行调试。


## Tone and Behavior

- 欢迎批评。当我错误或有误时请告诉我，甚至当你认为我可能错误或有误时也请告诉我。
- 如果有比我正在采用的方法更好的方法，请告诉我。
- 如果有我似乎不了解的相关标准或约定，请告诉我。
- 保持怀疑。
- 保持简洁。
- 简短摘要是可以的，但除非我们正在制定计划的细节，否则不要给出详细分析。
- 不要奉承，除非我特别要求你的判断，否则不要给予赞美。
- 偶尔的客套话是可以的。
- 随时提问。如果你对我的意图有疑问，不要猜测。直接问。

## 绝对规则：

- 禁止部分实现
- 禁止简化：不要 "//这是现在简化的东西，完整实现会blablabla"
- 禁止代码重复：检查现有 codebase 重用 functions 和 constants，在编写新 functions 之前先读取文件。使用常识 function name 来方便查找。
- 禁止死代码：要么使用要么从 codebase 中完全删除
- 为每个 FUNCTION 实现测试
- 禁止作弊测试：测试必须准确，反映真实使用并设计来揭示缺陷。不要无用的测试！设计测试为 verbose 以便我们可以用它们调试。
- 禁止不一致命名 - 阅读现有 codebase 命名模式。
- 禁止过度工程 - 当简单 functions 就能工作时，不要添加不必要的抽象、factory patterns 或 middleware。需要 "working" 时不要想 "enterprise"
- 禁止混合关注点 - 不要将验证逻辑放在 API handlers 内，将 database queries 放在 UI components 内等，而应该适当分离
- 禁止资源泄漏 - 不要忘记关闭 database connections，清除 timeouts，移除 event listeners，或清理 file handles