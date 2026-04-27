# 全局 AI Coding 偏好设置

## 项目 CLAUDE.md 引用规范

在任何项目中创建或更新 `CLAUDE.md` 时，在文件开头加入：

```markdown
@~/.claude/CLAUDE.md
```

项目级配置只写项目特有的内容（技术栈、目录结构、业务规范等），通用规则不重复。

## Memory

- [关于我](skills/memory-skills/user_profile.md) — 用户基本信息、语言偏好和技术栈
- [编码偏好](skills/memory-skills/coding_preferences.md) — 代码风格、提交信息、实现复杂度等偏好
- [安全规范](skills/memory-skills/security.md) — 提交代码时的安全底线
- [工作流](skills/memory-skills/workflow.md) — Git 分支策略和修改代码前的基本原则
- [文件读写方式](skills/memory-skills/feedback_file_io.md) — 按文件大小和结构性选择读写策略，小文件直接读写，大文件分段或多步执行
- [工具调用失败处理](skills/memory-skills/feedback_tool_failure.md) — 同一操作失败3次后停止重试，输出原因和解决方案，等用户确认
