---
name: feature-implement
description: 新需求实现阶段。一个会话 = 一个层级。同层并行 Agent，跨层串行验证，每层 commit，所有层完成后全量 Code Review + 功能完整度验收。适用于任何技术栈的全栈项目。
---

# Feature Implement — 分层实现

**Announce at start:** "I'm using the feature-implement skill 来实现当前层级。"

**前置条件：** spec.md、plan.md 和 tasks.md 已由 `feature-analysis` 生成。

**核心原则：**
- **一个会话 = 一个层级**（DB层 / 后端层 / 前端层）
- **同层并行，跨层串行**
- **每层完成 → 跨层对齐验证 → commit → 关闭会话**
- **所有层完成 → 全量 Code Review → 功能完整度验收**

**发现业务特例时立即写入 MEMORY.md，不要留在对话里。**

---

## Step 0: 加载上下文（最小化）

```
1. MEMORY.md（环境信息、业务特例、历史踩坑）
2. tasks.md（识别当前层级、未完成 task、完整度检查点）
3. plan.md 和 spec.md 中与当前层相关的章节
```

**识别当前层级：**
```
完整度检查点全部 ✓ → 进入全量 Code Review
DB 层有未完成 task → 当前层 = DB 层
DB 层完成，后端层有未完成 → 当前层 = 后端层
后端层完成，前端层有未完成 → 当前层 = 前端层
```

---

## Step 1: 是否需要 Worktree

并发开发多个功能 → `superpowers:using-git-worktrees`
单功能顺序开发 → 直接在当前分支工作

---

## Step 2: 大文件降级判断

当前层某个文件已有超过 3 个接口方法，或 tasks.md 描述涉及超过 3 个独立业务逻辑时，拆分为子 task 再分配 Agent。

---

## Step 3: 当前层 Agent 并行实现

用 `superpowers:dispatching-parallel-agents` 分配当前层所有 task。
等待所有 Agent 输出结构化摘要（必须包含：构建验证结果 + 字段清单 + edge case 覆盖列表）后进入 Step 4。任何 Agent 失败 → 当前会话修复。

---

### DB Engineer（资深数据库工程师）

**触发时机：** 当前层 = DB 迁移层

```
你是资深数据库工程师，负责实现以下 DB 迁移 task：

目标文件：[tasks.md 里的迁移文件路径]
变更描述：[tasks.md 里的描述]

⚠️ 实现前先输出理解确认（一句话）：
"我理解这个 task 要做的是：[具体表结构变更描述]"
等主会话确认后再开始实现。

必须先读：
- plan.md 的"数据结构"章节（DB 表定义）
- plan.md 的"架构决策"章节（DB 兼容性约束）
- CLAUDE.md 的数据库规范

实现要求：
- 只用 CLAUDE.md 规定的数据库版本兼容语法
- changeSet id 必须全局唯一，格式遵守 CLAUDE.md 规范
- 迁移文件一旦写好不可修改，只能新增
- 必须写 rollback 策略（无法回滚的操作显式标注 no-rollback）
- 字段名、类型、枚举值严格按 plan.md 的表定义

完成后输出结构化摘要：
## DB Agent 摘要
### 变更的表和字段
| 表名 | 字段名 | 类型 | 变更类型 |
### changeSet id 列表
### rollback 策略
### 与 plan.md 对应关系（一致 ✓/✗）
```

---

### Backend Engineer（资深后端工程师）

**触发时机：** 当前层 = 后端层（DB 层已完成）
**同层多个文件时每个 Agent 只负责一个文件。**

```
你是资深后端工程师，负责实现以下 task：

目标文件：[tasks.md 里的具体文件路径]
功能描述：[tasks.md 里的功能描述]
路由：[tasks.md 里的路由信息]

⚠️ 实现前先输出 CoT 推理（主会话确认后再写代码）：

## 实现推理
### 核心逻辑
[这个接口/函数的核心业务逻辑是什么，一句话]

### 关键决策
1. 分层：这个逻辑放在哪一层？为什么？
2. 事务边界：哪些操作需要在同一事务里？
3. 并发风险：有没有竞态条件？如何处理？
4. 权限检查：在哪一层检查？有没有业务特例（查 MEMORY.md）？

### Edge Case 处理计划
| Edge Case | 处理方式 |
（来自 spec.md，逐条说明怎么处理）

等主会话确认推理正确后再开始实现。

必须先读：
- plan.md 的"技术架构""数据结构""接口清单""核心逻辑设计"章节
- spec.md 的"Edge Cases""业务规则"章节
- CLAUDE.md 的后端规范（分层规则、错误处理、日志规范、禁止项）
- MEMORY.md（业务特例）

实现要求：
- 严格遵守 CLAUDE.md 的所有后端规范和禁止项
- 覆盖 spec.md 的所有 edge case，不只实现 happy path
- 每次修改后按 CLAUDE.md 的构建验证命令确认编译通过

完成后输出结构化摘要：
## Backend Agent 摘要 — [文件名]
### 实现的接口
| 路由 | 方法 | 状态 |
### 构建验证（通过/失败）
### 字段清单（供跨层对齐）
| 字段名 | 类型 | 枚举值 |
### Edge Case 覆盖
| Edge Case | 处理方式 | 对应检查点 |
### 业务特例（无/或描述）
```

---

### Frontend Engineer（资深前端工程师）

**触发时机：** 当前层 = 前端层（后端层已完成且跨层验证通过）
**同层多个文件时每个 Agent 只负责一个文件。**

```
你是资深前端工程师，负责实现以下 task：

目标文件：[tasks.md 里的具体文件路径]
功能描述：[tasks.md 里的功能描述]

⚠️ 实现前先输出理解确认（一句话）：
"我理解这个 task 要做的是：[具体 UI 交互和数据展示描述]"
等主会话确认后再开始实现。

必须先读：
- plan.md 的"技术架构""数据结构""接口清单"章节
- spec.md 的"用户故事""Edge Cases"章节
- CLAUDE.md 的前端规范（页面结构、API 调用规范、类型命名、禁止项）
- MEMORY.md（业务特例）

实现要求：
- 严格遵守 CLAUDE.md 的所有前端规范和禁止项
- 字段名和类型严格按 plan.md 的数据结构定义
- 每次修改后按 CLAUDE.md 的构建验证命令确认通过

完成后输出结构化摘要：
## Frontend Agent 摘要 — [文件名]
### 实现的页面/组件
### 构建验证（通过/失败）
### Interface 字段清单（供跨层对齐）
| 字段名 | 类型 | 对应 API 字段 |
### Edge Case 覆盖
| Edge Case | 处理方式 | 对应检查点 |
### 业务特例（无/或描述）
```

---

## Step 4: 跨层对齐验证（汇合点）

**收到所有 Agent 摘要后，用 Grep 工具实际验证关键字段，不靠记忆比对。**

### DB 层完成后
```
- [ ] plan.md 表定义和迁移 SQL 字段名/类型一致
- [ ] changeSet id 无重复（Grep 确认）
- [ ] rollback 策略完整
```

### 后端层完成后
```
用 Grep 验证（按 CLAUDE.md 的模块路径）：
grep -r "字段名" [后端模块路径] --include="*.go" | head -10

- [ ] 字段名一致（Grep 确认大小写）
- [ ] 字段类型一致（对照 Agent 摘要）
- [ ] 枚举值一致（Grep 确认大小写完全匹配）
- [ ] 可空字段处理一致
- [ ] 后端层所有 edge case 已覆盖
- [ ] plan.md 接口清单全部实现
```

### 前端层完成后
```
用 Grep 验证（按 CLAUDE.md 的模块路径）：
grep -r "字段名" [前端模块路径] --include="*.ts" --include="*.tsx" | head -10

- [ ] interface 字段名和 API 返回字段名一致
- [ ] 枚举值和后端定义一致（Grep 确认）
- [ ] 可空字段有 undefined/null 处理
- [ ] 前端层所有 edge case 已覆盖
- [ ] spec.md 所有用户故事有对应 UI 实现
```

**发现不一致时：** 当前会话修复，修复后重新 Grep 验证。

---

## Step 5: commit 当前层

```
1. 确认目标仓库（哪个子模块）
2. git status 展示变更文件，等用户确认
3. 只用 git add <具体文件>，禁止 git add .
4. git diff --staged 最终确认
5. 格式：type(scope): description
6. 推送（monorepo 场景同步更新子模块指针）
```

tasks.md 标记当前层所有 task 完成，**同时更新顶部进度状态**：

```markdown
- [x] T1 xxx（完成于 <commit hash>，跨层验证 ✓）

## 当前进度（更新）
- DB 层：✓ 完成（commit: <hash>）
- 后端层：🔄 进行中
- 前端层：⬜ 未开始
```

---

## Step 6: 判断是否还有下一层

有未完成层 → 告知用户下一层，关闭本会话，新会话继续
所有层完成 → 进入 Step 7

---

## Step 7: 全量 Code Review

用 `superpowers:requesting-code-review` 触发。

| 检查项 | 说明 |
|--------|------|
| 功能完整度 | tasks.md 完整度检查点全部 ✓ |
| Edge case 覆盖 | 对照 spec.md 矩阵逐一确认 |
| 并发安全 | 涉及共享资源是否有锁/事务保护 |
| 权限完整性 | 查 MEMORY.md 业务特例 |
| 接口契约 | 是否破坏现有调用方依赖 |

---

## Step 8: 功能完整度验收

用 `superpowers:verification-before-completion` 验证。

```
- [ ] 每个用户故事有对应实现
- [ ] 每个 edge case 有处理逻辑
- [ ] 每个接口已实现
- [ ] 每个 DB 表变更已迁移
- [ ] 所有模块构建无错误
```

如需部署 → `deploy-verify`，如需端到端验证 → `qa-e2e`

---

## 配套 Skills

| 场景 | Skill |
|------|-------|
| 隔离工作区 | `superpowers:using-git-worktrees` |
| 同层多文件并行 | `superpowers:dispatching-parallel-agents` |
| 遇到 bug | `bug-fix` |
| Code Review | `superpowers:requesting-code-review` |
| 完整度验收 | `superpowers:verification-before-completion` |
| 提交代码 | `/commit` |
| 端到端验证 | `qa-e2e` |
| 部署 | `deploy-verify` |
| 完成分支 | `superpowers:finishing-a-development-branch` |
