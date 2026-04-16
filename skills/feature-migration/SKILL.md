---
name: feature-migration
description: 功能迁移流程。从旧系统读取并理解功能逻辑，在新系统按新架构实现，不是简单复制。适用于 gaia→glite 等跨系统迁移场景。
---

# Feature Migration — 功能迁移

**Announce at start:** "I'm using the feature-migration skill 来执行功能迁移。"

**核心原则：理解旧逻辑，用新架构实现，不是复制粘贴。**

**发现业务特例时立即写入 MEMORY.md。**

---

## Step 0: 加载上下文

```
1. 读 CLAUDE.md（新系统架构、禁止项、reference/ 只读规范）
2. 读 MEMORY.md（业务特例、历史踩坑）
3. 确认迁移的功能名称和旧系统路径
```

---

## Step 1: 旧系统代码分析（Codebase Explorer）

**角色：资深代码分析师**
**目标：提取旧系统的功能逻辑，不修改任何文件。**

```
你是资深代码分析师，分析旧系统的功能实现。

旧系统路径：[reference/ 下的模块路径]
功能名称：[要迁移的功能]

分析以下内容（只读，不修改）：
1. 入口接口：API 路径、请求/响应结构
2. 核心业务逻辑：关键函数、状态机、业务规则
3. 数据结构：DB 表、字段、枚举值
4. 权限控制：角色、隔离策略
5. 依赖关系：调用了哪些外部服务/模块

输出分析报告（保存到 docs/specs/NNN-migration-name/old-system-analysis.md）：

## 接口清单
| 方法 | 路径 | 说明 |

## 核心业务逻辑
[关键函数和逻辑描述，含状态机]

## 数据结构
[DB 表和字段，含枚举值]

## 权限控制
[角色和隔离策略]

## 与新系统的架构差异
[新旧系统在分层、命名、规范上的差异]

## 迁移风险
[可能的兼容性问题、数据差异、逻辑差异]
```

**Agent 完成后，主会话确认：**
- 接口清单是否完整（对照旧系统路由文件）
- 架构差异是否识别清楚
- 迁移风险是否有遗漏

**分析报告质量不足时的处理：**

| 问题 | 处理方式 |
|------|---------|
| 接口清单不完整（遗漏了某些路由） | 主会话用 Grep 补充：`grep -r "router\|handler\|@Get\|@Post" [旧系统路径] -l`，手动补充到报告 |
| 架构差异描述模糊（只说"不同"没说怎么不同） | 主会话直接修改 old-system-analysis.md，不重跑 Agent |
| 迁移风险为空（Agent 没有识别到风险） | 标注"需人工确认"，继续流程，但在 spec.md 的"已知限制"章节记录 |
| 报告格式错误（缺少某个章节） | 主会话补充缺失章节，不重跑 Agent |

**禁止因报告不完整而重跑 Agent**——重跑成本高且结果不一定更好，直接修正报告更高效。

---

## Step 2: 制定迁移 Spec

基于 old-system-analysis.md，用 `feature-analysis` skill 生成迁移的 spec.md + plan.md + tasks.md。

**传递给 feature-analysis 的额外输入：**
- 将 old-system-analysis.md 的路径告知 Agent A（Product Manager），让其读取业务规则和接口清单
- 将 old-system-analysis.md 的路径告知 Agent B（Software Architect），让其读取架构差异和数据结构

**Agent A prompt 补充（在 feature-analysis 的 Agent A prompt 基础上追加）：**
```
额外输入：
- 旧系统分析报告：docs/specs/NNN-migration-name/old-system-analysis.md
  （读取业务规则、接口清单、权限控制章节，作为用户故事和业务规则的来源）

注意：
- 用户故事必须覆盖旧系统的所有接口（对照接口清单逐条转化）
- 业务规则必须包含旧系统的所有状态机和权限控制
- "背景"章节注明：这是迁移，原始逻辑来自 [旧系统路径]
```

**Agent B prompt 补充（在 feature-analysis 的 Agent B prompt 基础上追加）：**
```
额外输入：
- 旧系统分析报告：docs/specs/NNN-migration-name/old-system-analysis.md
  （读取架构差异、数据结构、迁移风险章节）

注意：
- 技术架构必须按新系统规范设计，不能照搬旧系统目录结构
- 数据结构必须说明新旧字段的映射关系（如枚举值大小写差异）
- "架构决策"章节必须说明：新旧架构差异如何处理
```

**迁移场景的特殊要求：**
- tasks.md 的完整度检查点必须覆盖旧系统的所有接口（对照 old-system-analysis.md 接口清单）

**Edge Case 审计时额外检查：**
- 旧系统有哪些特殊状态码在新系统没有对应
- 旧系统的权限逻辑在新系统是否有差异
- 旧系统的数据格式（枚举值、字段名）和新系统规范是否一致

---

## Step 3: 实现

用 `feature-implement` skill 按层级实现。

**迁移场景的额外验证（每层完成后）：**
- 对照 old-system-analysis.md 的接口清单，确认每个接口都已实现
- 对照旧系统的枚举值，确认新系统的枚举值完全一致（或有明确的映射关系）
- 对照旧系统的业务规则，确认新系统的实现没有遗漏

---

## Step 4: 迁移验证

用 `qa-e2e` skill 验证，额外增加：

```
迁移专项验证：
- [ ] 旧系统的所有接口在新系统都有对应实现
- [ ] 关键业务流程的行为和旧系统一致
- [ ] 数据格式（枚举值、字段名）和旧系统兼容或有明确映射
- [ ] 权限控制行为和旧系统一致
```

---

## 常见踩坑

| 问题 | 预防措施 |
|------|----------|
| 直接复制旧代码，不适配新架构 | Step 1 先分析架构差异，Step 2 制定适配方案 |
| 漏掉旧系统的特殊状态码 | Step 1 的分析报告必须列出所有枚举值 |
| 枚举值大小写不一致 | Step 3 跨层验证时 Grep 确认枚举值 |
| 修改了 reference/ 目录 | CLAUDE.md 禁止项：reference/ 只读 |

---

## 配套 Skills

| 场景 | Skill |
|------|-------|
| 制定迁移 spec | `feature-analysis` |
| 分层实现 | `feature-implement` |
| 迁移验证 | `qa-e2e` |
| 部署 | `deploy-verify` |
