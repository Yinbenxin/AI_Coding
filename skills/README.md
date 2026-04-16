# Skills

自定义 AI 编程技能（Skill）的模板与经验，适用于 Claude Code 等支持 Skill 的工具。

## 技能列表

### 开发流程

| 目录 | 名称 | 功能描述 |
|------|------|---------|
| `brainstorming/` | 头脑风暴 | 创意工作前必用，探索用户意图、需求和设计，避免直接跳入实现 |
| `planning-with-files/` | 文件规划 | 把规划、进度和知识写入 Markdown 文件持久化，解决上下文压缩导致 Claude 忘记任务的问题 |
| `feature-analysis/` | 需求分析 | 新需求分析阶段，产出 spec.md + plan.md + tasks.md，适用于任何技术栈的全栈项目 |
| `feature-implement/` | 需求实现 | 新需求实现阶段，同层并行 Agent、跨层串行验证，每层 commit，最终 Code Review |
| `feature-migration/` | 功能迁移 | 从旧系统读取并理解功能逻辑，在新系统按新架构实现，不是简单复制 |
| `feature-refactor/` | 功能改造 | 改造前做影响分析（调用链、接口契约、数据依赖），确认范围后实现，最后回归验证 |

### 代码质量

| 目录 | 名称 | 功能描述 |
|------|------|---------|
| `webapp-testing/` | Web 应用测试 | 自动用 Playwright 写测试脚本、启动浏览器、跑测试、截屏，有问题自动调试 |
| `mcp-builder/` | MCP 构建器 | 分四阶段引导构建 MCP Server：理解 API、设计工具接口、实现、测试，自动处理边界情况 |
| `skill-creator/` | Skill 创建器 | Anthropic 官方出品，帮你创建新 Skill，内置 eval 测试框架，支持 A/B 对比验证效果 |

### 前端设计

| 目录 | 名称 | 功能描述 |
|------|------|---------|
| `frontend-design/` | 前端设计 | 生成高质量、有设计感的前端界面，避免千篇一律的 AI 风格 |
| `frontend-slides/` | 前端幻灯片 | 从零创建或将 PPT 转换为动画丰富的 HTML 演示文稿 |
| `ui-ux-pro-max/` | UI UX Pro Max | 内置 67 种 UI 风格和 161 套行业配色方案，根据项目类型自动推荐设计系统，支持 React/Vue/Svelte/SwiftUI/Flutter |
| `pptx/` | PPTX 生成 | 直接生成 .pptx 文件，支持母版、图表、动画，快速生成 PPT 初稿 |

### 项目管理

| 目录 | 名称 | 功能描述 |
|------|------|---------|
| `project-bootstrap/` | 项目初始化 | 建立 CLAUDE.md、MEMORY.md 体系、Hooks 配置，让 AI 每次会话快速进入状态 |

### 专业领域

| 目录 | 名称 | 功能描述 |
|------|------|---------|
| `09-feishu-expert/` | 飞书专家 | 飞书全链路解决方案，涵盖文档/Wiki 读写、多维表格自动化、机器人交互，基于 lark-mcp 实现 |

## 仅 Plugin 形式（无 SKILL.md，需通过 claude plugin install 安装）

| Plugin | 安装命令 | 功能描述 |
|--------|---------|---------|
| code-review | `claude plugin install code-review` | 多 Agent 并行审查 PR，按置信度过滤假阳性，有效减少无用反馈 |
| code-simplifier | `claude plugin install code-simplifier` | 检查最近修改的代码，合并重复逻辑、简化条件分支，不改功能只做精简 |
| ralph-loop | `claude plugin install ralph-loop` | 通过 Stop Hook 拦截 Claude 退出，强制循环直到满足完成条件，解决 Claude 提前收工问题 |

## 使用方式

在 Claude Code 中通过 Skill 工具调用，例如：

```
/brainstorming
/feature-analysis
/project-bootstrap
```

安装 Plugin：

```bash
claude plugin install code-review
claude plugin install code-simplifier
claude plugin install ralph-loop
claude plugin install planning-with-files
claude plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

## 参考资源

- [Anthropic 官方 Skills 仓库](https://github.com/anthropics/skills)
- [Anthropic 官方 Plugins 仓库](https://github.com/anthropics/claude-plugins-official)
- [Awesome Claude Skills 社区列表](https://github.com/travisvn/awesome-claude-skills)
- [Skills 市场](https://skillsmp.com/)
