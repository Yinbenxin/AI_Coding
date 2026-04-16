---
name: project-bootstrap
description: 新项目初始化。建立 CLAUDE.md、MEMORY.md 体系、Hooks 配置、子模块文档结构，让 AI 在每次会话里都能快速进入状态。
---

# Project Bootstrap — 新项目初始化

**Announce at start:** "I'm using the project-bootstrap skill 来初始化项目的 AI 协作体系。"

**目标：** 建立一套让 AI 每次会话都能快速进入状态的上下文体系，而不是从零开始摸索。

---

## Step 0: 了解项目基本情况

先问清楚，再动手：

```
1. 项目是什么？（一句话描述）
2. 技术栈是什么？（语言、框架、数据库、部署方式）
3. 有哪些子模块/服务？
4. 有哪些开发/测试/生产环境？
5. 有哪些已知的禁止项或特殊约束？
6. 功能开发工作流是什么？
   - 有没有代码生成工具？（如 goctl、protoc、openapi-generator、mybatis-generator 等）
   - 新增接口/功能的强制步骤是什么？（如：先改 .api → 生成 → 再实现）
   - 有没有"生成文件禁止手改"的约束？
```

---

## Step 1: 初始化根目录 CLAUDE.md

**先检查是否已有 CLAUDE.md：**

| 情况 | 处理方式 |
|------|---------|
| 不存在 | 按模板新建 |
| 已存在但内容简单（< 30 行） | 在现有内容基础上补充缺失章节，不覆盖 |
| 已存在且内容完整 | 只补充"禁止项"和"构建验证命令"章节（如缺失），其余保留 |

**禁止覆盖已有 CLAUDE.md**——已有内容可能包含重要的项目约束，覆盖会丢失。

放在项目根目录，约束 AI 的行为边界。**把踩过的坑写进去，它下次就不会再犯。**

```markdown
# CLAUDE.md — [项目名]

## 项目概述
[一句话描述项目做什么]

## 目录结构
[列出主要目录和职责]

## 技术栈
[语言、框架、数据库、部署方式]

## 核心规范

### 代码规范
[语言特有的规范，如 Go 禁止修改 goctl 生成文件]

### 数据库规范
[版本兼容要求、禁止的语法]

### Git 工作流
[分支策略、commit 格式、禁止 git add . 等]

### 构建验证命令
[每次修改后必须运行的验证命令]

## 禁止项
[明确列出绝对不能做的事，来自真实踩坑]

## 文档索引
[L0 全景 → L1 模块概览 → L2 功能细节]

## 人工补充约束（请勿删除）

### AI Agent 工作流适配（feature-analysis / feature-implement skill 使用时必读）

#### feature-analysis 产出映射
- `spec.md` → 遵守 [需求文档规范路径，如无则用默认格式]
- `plan.md` → 遵守 [设计文档规范路径，如无则用默认格式]
- `tasks.md` 后端层 task 顺序：
  1. [步骤1，如：修改接口定义文件]
  2. [步骤2，如：执行代码生成命令]
  3. [步骤3，如：实现业务逻辑层]
  4. [步骤4，如：实现领域服务层]

#### feature-implement 后端层强制步骤
Backend Engineer Agent 实现后端时，**必须严格按此顺序**：
1. 先读 [代码规范文件路径]
2. [项目特有步骤，如：修改 .api 文件]
3. [项目特有步骤，如：执行 make goctl-api]
4. [实现业务逻辑]
5. 执行 [构建验证命令] 确认编译通过

#### 禁止项（Agent 必须遵守）
- [生成文件禁止手改，如：禁止修改 _gen.go]
- [其他项目特有禁止项]
```

**禁止项要具体，来自真实踩坑，不要写空泛的规则。**

---

## Step 2: 初始化 MEMORY.md 体系

路径：`.claude/projects/<project>/memory/`（Claude Code 自动按工作目录隔离）

### MEMORY.md（索引文件）

```markdown
# MEMORY.md

- [环境信息](env-info.md) — 所有环境的连接信息
- [运维手册](ops-reference.md) — 低频但重要的运维操作
- [业务特例](business-rules.md) — 不符合通用规则的业务逻辑
```

### env-info.md

```markdown
# 环境信息

## [环境名，如 dev / staging / prod]
- 访问地址：
- 数据库：
- 认证方式：（查 CLAUDE.md 的认证规范）
- 镜像仓库：
- 其他连接信息：
```

### ops-reference.md

```markdown
# 运维手册

## 首次部署检查清单
[环境特有的前置条件，如目录创建、ConfigMap 配置]

## 常见问题
[历史踩过的环境问题和解决方法]
```

### business-rules.md

```markdown
# 业务特例

[不符合通用规则的业务逻辑，如：
- 某接口不需要租户隔离
- 某字段有特殊枚举值含义
]
```

---

## Step 3: 配置 Hooks（自动化守护）

在 `.claude/settings.json` 配置，代码变更后自动触发构建验证：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "<按 CLAUDE.md 的构建验证命令>"
          }
        ]
      }
    ]
  }
}
```

**在部署前就抓住编译错误，而不是等到服务崩溃才发现。**

Hook 命令要：
- 快速（秒级，不要跑完整测试）
- 只验证当前修改的模块
- 输出限制行数（避免刷屏）

---

## Step 4: 子模块 CLAUDE.md（如有子模块）

每个子模块建自己的 CLAUDE.md，根目录只做导航，不重复细节：

```markdown
# CLAUDE.md — [子模块名]

## 职责
[这个子模块负责什么]

## 技术栈
[子模块特有的技术栈]

## 构建验证
[子模块的构建命令]

## 禁止项
[子模块特有的禁止项]

## 关键文件
[最重要的几个文件路径和作用]
```

根目录 CLAUDE.md 加入导航：

```markdown
## 子模块导航
- [模块A](./moduleA/CLAUDE.md) — 职责描述
- [模块B](./moduleB/CLAUDE.md) — 职责描述
```

---

## Step 5: 验证体系是否就绪

用以下问题检验：

```
- [ ] AI 能否从 CLAUDE.md 知道：技术栈、禁止项、构建命令？
- [ ] AI 能否从 MEMORY.md 知道：所有环境的连接信息？
- [ ] AI 能否从 business-rules.md 知道：项目的业务特例？
- [ ] Hooks 是否能在代码修改后自动触发构建验证？
- [ ] 子模块是否各有自己的 CLAUDE.md？
- [ ] CLAUDE.md 的"AI Agent 工作流适配"章节是否已填写？
  - feature-analysis 产出映射是否明确？
  - feature-implement 后端层强制步骤是否完整？
  - 生成文件禁止手改的约束是否已列出？
```

全部通过，体系就绪。

---

## 持续维护原则

- **踩了新坑 → 立即写入 CLAUDE.md 的禁止项**
- **发现业务特例 → 立即写入 business-rules.md**
- **遇到环境问题 → 解决后写入 ops-reference.md**
- **MEMORY.md 是活文档，不是一次性初始化**

---

## 配套 Skills

| 场景 | Skill |
|------|-------|
| 开始新需求 | `feature-analysis` |
| 生成项目文档 | `generate-monorepo-docs` |
| 优化 CLAUDE.md | `claude-memory` |
