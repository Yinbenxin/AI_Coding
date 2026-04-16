# Tools

AI 编程工具使用经验，如 Claude Code、Cursor、Copilot 等。

## 工具列表

### [Superset](https://github.com/superset-sh/superset)

专为 AI Agent 设计的代码编辑器/编排工具，核心能力是在隔离的 git worktree 中并行运行多个 CLI 编码 Agent。

**主要特性：**
- 同时运行 10+ 个 Agent，无需频繁切换上下文
- 每个任务独立在自己的 git worktree 和分支中，Agent 互不干扰
- 内置 diff 查看器，快速 review 和编辑 Agent 产出的变更
- 支持所有主流 CLI Agent：Claude Code、Codex、Cursor、Gemini CLI、Copilot 等
- 一键将任意工作区在编辑器或终端中打开

**适用场景：** 需要同时推进多个独立任务，或想让多个 Agent 并行工作提升效率时。

**平台要求：** macOS，依赖 Bun、Git 2.20+、gh CLI。
