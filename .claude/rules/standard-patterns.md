# Commands 标准模式

此文件定义所有命令应遵循的通用模式以保持一致性和简单性。

## 核心原则

1. **Fail Fast** - 检查关键先决条件，然后继续
2. **Trust the System** - 不要过度验证很少失败的事情
3. **Clear Errors** - 当某事失败时，准确说明什么和如何修复
4. **Minimal Output** - 显示重要内容，跳过装饰

## 标准验证

### 最小预检
只检查绝对必要的：
```markdown
## 快速检查
1. 如果命令需要特定 directory/file：
   - 检查它存在：`test -f {file} || echo "❌ {file} not found"`
   - 如果缺失，告诉用户修复它的确切命令
```

所有命令都应遵循这些模式以确保一致的用户体验。
