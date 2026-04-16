# Skills

自定义 AI 编程技能（Skill）的模板与经验，适用于 Claude Code 等支持 Skill 的工具。

## 技能列表

| 目录 | 名称 | 功能描述 |
|------|------|---------|
| `09-feishu-expert/` | 飞书专家 | 飞书全链路解决方案，涵盖文档/Wiki 读写、多维表格自动化、机器人交互，基于 lark-mcp 实现 |
| `brainstorming/` | 头脑风暴 | 创意工作前必用，探索用户意图、需求和设计，避免直接跳入实现 |
| `feature-analysis/` | 需求分析 | 新需求分析阶段，产出 spec.md + plan.md + tasks.md，适用于任何技术栈的全栈项目 |
| `feature-implement/` | 需求实现 | 新需求实现阶段，同层并行 Agent、跨层串行验证，每层 commit，最终 Code Review |
| `feature-migration/` | 功能迁移 | 从旧系统读取并理解功能逻辑，在新系统按新架构实现，不是简单复制 |
| `feature-refactor/` | 功能改造 | 改造前做影响分析（调用链、接口契约、数据依赖），确认范围后实现，最后回归验证 |
| `frontend-design/` | 前端设计 | 生成高质量、有设计感的前端界面，避免千篇一律的 AI 风格 |
| `frontend-slides/` | 前端幻灯片 | 从零创建或将 PPT 转换为动画丰富的 HTML 演示文稿 |
| `project-bootstrap/` | 项目初始化 | 建立 CLAUDE.md、MEMORY.md 体系、Hooks 配置，让 AI 每次会话快速进入状态 |

## 使用方式

在 Claude Code 中通过 Skill 工具调用，例如：

```
/brainstorming
/feature-analysis
/project-bootstrap
```

或在对话中直接描述需求，Claude Code 会自动匹配合适的 Skill。
