# 用 Claude Code 做全栈开发：33 天 710 次提交的真实经验

> 471 次会话 · 852 小时 · 710 次提交
> 2026-03-06 ~ 2026-04-08

---

## 项目是什么

数据沙箱管理系统，企业级的数据安全隔离平台。用 Kubernetes 给每个沙箱创建独立 namespace，数据只读挂载进去，用户通过 VNC 桌面操作。

代码分布在 4 个子模块：Go 后端、React 前端、Go 管控服务、SQL 迁移脚本。同时维护 6 个以上的开发测试环境。一个功能改下来，往往要动 DB schema、backend API、manager 服务、前端页面，四个地方全要改。

这个背景决定了我用 AI 的方式跟大多数人不一样——不是让它帮我写代码片段，而是让它跑完整个工程闭环。

---

## 我怎么用它

一句话：**把 Claude 当 DevOps 操作员用**。

一个典型会话是这样的：发现 bug → 追根因（跨 Go 服务、K8s 资源、nginx、DB）→ 改代码 → 构建镜像 → 部署到指定环境 → 验证 pod 状态 → 回归测试 → 提交 + 更新子模块指针。全部在一个会话里完成。

19,677 次 Bash 调用，远超其他任何工具。它在我这里是真正的终端操作员。

---

## 开发流程

### 1. 创建 Monorepo 项目

新项目启动时，先把 AI 的工作环境搭好，后续所有操作才有正确的上下文：

```
1. 初始化根目录 CLAUDE.md
   → 技术栈、禁止项、代码规范
   → 建立 L0/L1/L2 渐进式文档结构（L0 全景 → L1 模块概览 → L2 功能细节）

2. 初始化 MEMORY.md 体系
   → env-info.md：填入所有环境连接信息
   → ops-reference.md：填入运维手册

3. 每个子模块建自己的 CLAUDE.md（L1）
   → 根目录只做导航，不重复细节

4. 配置 Hooks
   → Go 文件变更后自动 go build
```

---

### 2. 需求分析 → Spec 收敛

把模糊需求变成可执行的技术规格，这一步做好，后面编码返工少很多。

**用到的 Skills：**

```
superpowers:brainstorming  ← 先发散，理解需求意图和边界，探索实现方向
         ↓
speckit.clarify            ← 提 5 个关键澄清问题，消除歧义
         ↓
      用户回答
         ↓
speckit.specify            ← 生成 spec.md（功能边界 + API 设计 + DB 变更）
         ↓
plan-eng-review            ← 工程架构审查（多仓库边界、API 设计、租户隔离策略）
         ↓
speckit.tasks              ← 生成 tasks.md（依赖排序的可执行任务列表）
         ↓
speckit.analyze            ← 跨文档一致性检查（spec/plan/tasks 是否对齐）
```

**spec.md 的结构：**

```markdown
## 功能描述
## 影响范围（哪些模块）
## DB 变更（字段、索引、迁移 SQL）
## API 设计（路径、请求/响应结构）
## 业务规则（权限、状态机、边界条件）
## 不在范围内
```

**关键：在 spec 阶段就逼出所有 edge case，而不是编码后才发现。**

```
生成 spec 之前，先列出这个功能所有可能的边界情况：
- 所有涉及的状态码
- 所有权限角色的行为差异
- 并发场景下的幂等性要求
确认后再写 spec。
```

---

### 3. 编码实现

**一次会话只做一件事。** 这是我踩了很多坑之后得出的结论。

同时修 bug + 部署 + QA + 迁移 DB，完成率很低，而且出了问题很难定位是哪一步的锅。拆开来，每个会话一个目标，完成率和质量都明显更高。

**全栈变更的标准顺序：**

```
DB 迁移（MySQL 5.7 兼容 SQL）
  → Backend（GORM model + API handler，go build 验证）
    → Manager（接口调用更新，go build 验证）
      → Frontend（TypeScript interface + 页面实现）
        → 跨栈对齐验证（字段名/类型/枚举值全链路一致）
```

**编码前必做：Edge Case 审计**

```
修这个功能之前，先列出：
- 所有涉及的状态码和状态转换
- 所有读写该字段的代码位置
- 并发场景下的幂等性要求
- 权限边界（哪些角色可以操作）
给我完整矩阵，确认后再写代码。
```

这条规则来自一个真实教训：修引擎销毁逻辑，Claude 只处理了常规状态，漏掉了 `status=7`（外部引擎）。部署后回归测试才发现，多跑了一轮部署。如果事先列矩阵，这个问题在编码阶段就能发现。

---

### 4. Code Review

用 `superpowers:requesting-code-review` 触发，重点检查：

```
1. 是否覆盖了所有 edge case（特别是异常状态码）
2. 跨模块接口字段是否端到端对齐
3. 是否有并发安全问题
4. 数据库操作是否有事务保护
5. 权限检查是否完整（租户隔离、角色边界）
```

**真实踩过的坑，现在都在 review checklist 里：**

| 问题 | 案例 |
|------|------|
| 漏掉状态码 | 引擎销毁逻辑未处理 status=7（外部引擎），ExternalEngineId 没清除 |
| 重复路由注册 | 手动改 goctl 生成文件，CrashLoopBackOff 上线 |
| 字段未传播 | DB 加了字段，API struct 没更新，前端取不到 |
| 权限漏洞 | 跨租户操作未做隔离检查 |
| 叠加 bug | SFTP 入箱失败：source_type 大小写不匹配 + SM2 密码未解密 + SSH 握手无超时，三个问题叠在一起 |

---

### 5. 端到端验证与部署

**标准部署提示词：**

```
读 MEMORY.md 的环境表。
把 sandbox-manager 部署到 dev-140：
1. 从当前分支构建镜像，tag 格式：1.0.0-beta-{commit_hash}
2. push 到镜像仓库
3. kubectl rollout status 等待 pod Running
4. 访问 /healthz 验证服务响应
5. 如果失败，kubectl logs 抓日志后回滚
```

**QA 测试覆盖多角色：**

```bash
# SM2 加密登录（系统不接受明文密码）
PUB=$(curl -s http://HOST/api/v1/auth/public-key | python3 -c "...")
TOKEN=$(# SM2 加密后登录)

# 多角色权限边界
for ROLE in admin smgr req datamgr analyst; do
  # 正常操作 + 越权操作（应被拒绝）
done

# 核心业务流程
# 沙箱创建 → 数据注入 → VNC 访问 → 出箱申请 → 审批 → 下载
```

**验证完成标准：**
- 所有 pod Running，无 CrashLoop
- 核心业务流程（创建/访问/销毁）正常
- 多角色权限边界符合预期
- 跨集群（管控集群 + 数据集群）通信正常

---

## 工具体系

### CLAUDE.md — 给 AI 的工作规范

放在项目根目录，约束它的行为边界。核心思路是：**把踩过的坑写进去，它下次就不会再犯**。

我们项目里最重要的几条：

```markdown
## Go 代码生成规范
- 禁止直接修改 goctl 生成的文件（routes.go 等）
- 每次编辑 Go 文件后运行 go build ./api/... 验证编译

## 数据库迁移规范
- 只用 MySQL 5.7 兼容语法，禁止 IF NOT EXISTS with ADD COLUMN
- 禁止 DELIMITER-based stored procedures（CLI 粘贴不兼容）

## Git 工作流规范
- 禁止 git add . 或 git add -A，必须明确指定文件
- 操作前确认当前在哪个子模块目录
```

为什么要写这些？因为都是真实踩过的坑。比如 goctl 生成文件被手动改了，导致重复路由注册，直接 CrashLoopBackOff 上线。MySQL 5.7 语法问题更是反复出现，Claude 默认会生成 8.0 的语法。

CLAUDE.md 还建立了**渐进式文档披露**的结构——L0 是系统全景，L1 是各子模块概览，L2 是具体功能细节。AI 按需深入，不会一次性把所有文档都读进来。

---

### MEMORY.md — 跨会话的持久记忆

Claude 每次会话都是全新的，不记得上次的任何东西。这是最大的摩擦来源之一。

解决方案是建一套 memory 文件体系，放在 `.claude/projects/<project>/memory/` 下：

```
MEMORY.md          ← 索引，每次会话自动加载
env-info.md        ← 所有环境的 namespace/MySQL/SSH/镜像仓库
deploy-versions.md ← 各环境当前跑的版本号
bugfixes.md        ← Bug 修复历史（Fix 20~57）
ops-reference.md   ← 低频运维：SM2登录、出箱路径、Kaniko、审批流
push-steps.md      ← detached HEAD 场景下的推送流程
feedback_*.md      ← Claude 行为反馈，记录它犯过的错
```

每次会话开头说一句"读 MEMORY.md"，它就能直接知道所有环境细节，不用重新解释。

**feedback 文件是最有价值的**。比如我记录了一条：认证审批接口不需要租户隔离，admin 可以跨租户审批。这是业务特例，不写下来它每次都会按常规逻辑加租户过滤，然后我再纠正，循环往复。

---

### 常用 Skills

高频重复的操作封装成斜杠命令，不用每次重新描述规则。

**需求分析阶段**

| Skill | 用途 |
|-------|------|
| `superpowers:brainstorming` | 发散探索需求意图，理解边界 |
| `speckit.clarify` | 提 5 个关键澄清问题，消除歧义 |
| `speckit.specify` | 生成 spec.md |
| `plan-eng-review` | 工程架构审查（多仓库边界、租户隔离） |
| `speckit.tasks` | 生成依赖排序的 tasks.md |
| `speckit.analyze` | 检查 spec/plan/tasks 三文档一致性 |

**编码阶段**

| Skill | 用途 |
|-------|------|
| `superpowers:systematic-debugging` | 遇到 bug 时系统性追踪，先列所有可能原因 |
| `superpowers:test-driven-development` | 先写测试再实现，本地验证通过再部署 |
| `feature-dev:feature-dev` | 有 spec 后驱动全栈实现 |

**Review 与收尾**

| Skill | 用途 |
|-------|------|
| `superpowers:requesting-code-review` | 完成实现后触发 code review |
| `superpowers:verification-before-completion` | 声称完成前强制跑验证命令，证据先于断言 |
| `/commit` | 安全提交（禁止 git add .，强制确认文件列表） |
| `superpowers:finishing-a-development-branch` | 实现完成后决策：merge / PR / cleanup |

**`/commit` 的具体步骤：**

```
1. 确认目标仓库（哪个子模块）
2. 运行 git status，展示给我看
3. 等我确认文件列表，不自行决定
4. 只用 git add <具体文件>，禁止 git add .
5. 展示 git diff --staged 做最终确认
6. 格式：type(scope): description
7. 推送，detached HEAD 用 git push origin HEAD:<branch>
8. 更新子模块指针
```

为什么要这么严格？monorepo 里一不小心就会把不相关的变更混进去，或者忘记更新子模块指针，导致 CI 拉到旧代码。

---

### Hooks — 自动化守护

在 `.claude/settings.json` 配置，每次代码变更后自动触发：

```json
{
  "hooks": {
    "postToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "cd sandbox-backend && go build ./... 2>&1 | head -20" }]
      }
    ]
  }
}
```

在部署前就抓住编译错误，而不是等到 pod CrashLoop 了才发现。

---

### Git Worktree — 多功能并行开发互不干扰

同时开发多个功能时，最大的问题是会话之间的代码状态相互污染——A 功能改了一半的文件，B 功能的会话读到了，产生错误的上下文。

Git Worktree 解决这个问题：每个功能在独立的目录里工作，共享同一个 git 仓库历史，但工作区完全隔离。

**用法：**

```bash
# 为新功能创建独立 worktree
git worktree add .claude/worktrees/feat-approval-stats -b feat/approval-stats

# 在 Claude Code 里直接说：
# "用 worktree 开发审批状态统计功能"
# Claude 会自动创建 worktree 并切换进去
```

**典型场景：**

```
主工作区（main/release 分支）
  ├── worktree A：feat/approval-stats    ← 会话 1 在这里开发
  ├── worktree B：feat/engine-paging     ← 会话 2 在这里开发
  └── worktree C：fix/vnc-proxy-bug      ← 会话 3 在这里修 bug
```

三个会话同时跑，互不影响。每个 worktree 有自己独立的工作区文件，Claude 在各自的 worktree 里读文件、改代码、跑编译，不会读到其他功能未完成的变更。

**开发完成后：**

```bash
# 功能完成，合并回主分支
git checkout main
git merge feat/approval-stats

# 清理 worktree
git worktree remove .claude/worktrees/feat-approval-stats
```

**在 Claude Code 里用 `superpowers:using-git-worktrees` skill** 可以自动化整个 worktree 的创建、切换和清理流程。

---

### Parallel Agents — 多 repo 并行修复

一个 bug 同时涉及 backend、manager、frontend 三个 repo 时，串行一个个来太慢。可以用并行 Agent：

```
├── Agent A → sandbox-backend（追踪 + 修复 + 编译验证）
├── Agent B → sandbox-manager（追踪 + 修复 + 编译验证）
└── Agent C → sandbox-frontend（追踪 + 修复 + 类型检查）
         ↓
    协调 Agent 验证跨 repo 接口契约一致性
```

---

## 真实场景记录

### 多用户 VNC 隔离

这个功能横跨 5 个模块：backend、manager、sandbox-images（Docker 镜像）、Helm Chart、frontend。

AI 帮我做的事：
- 追踪 slot 分配逻辑（SELECT FOR UPDATE 并发安全）
- 生成 `member_vnc_provision.sh` 脚本（创建 Linux 用户、启动独立 vncserver、创建隔离目录）
- 更新 Helm Chart 的 NodePort Service（slot1~5 对应端口 6902-6906）
- 写 kubectl exec 调用脚本的 manager 侧代码

AI 没帮到的地方：
- 镜像构建失败时，它不知道 registry 的实际状态，需要我手动 `docker pull` 验证
- 跨集群的 NodePort 路由问题，它给的方案在双集群环境下不对，需要我纠正

最后的结果：后端 + manager + 镜像 + Chart 全部实现，前端 UI 是唯一缺口（`UserManagementModal.tsx` 还没调用 `/members/detail` 接口展示 VNC 凭据）。这个缺口就记在 MEMORY.md 里，下次会话直接接着做。

---

### 发布生产环境（publish_to_prod）

审批流涉及多个状态转换：提交 → 审批 → 部署 → running，还有 `deploy_status` 字段的端到端传播。

AI 帮我追踪了整个审批链路，找到 `resolved_image` 为空的根因——Kaniko 未配置导致快照镜像为空，publish 时拿不到镜像。

但环境问题它解决不了：
- NFS 目录不会自动创建，首次部署 production 引擎时 PVC 挂载失败，需要手动 `mkdir`
- ConfigMap 里缺 `PodReadyTimeoutSeconds`，旧镜像的默认值覆盖了新配置

这类**环境特有问题**，AI 能帮你分析日志、定位原因，但最终还是要你去手动操作。我把这些坑都记进了 `ops-reference.md`，下次换环境部署时直接查。

---

### 叠加 Bug 的追踪

SFTP 抽样入箱一直失败，报 `invalid connection`。表面看是网络问题，实际是三个 bug 叠在一起：

1. `source_type` 大小写不匹配：DB 存 `'SFTP'`，代码判断 `== "sftp"`，走错了 MySQL 路径，SSH 握手卡 2 分钟后超时
2. SM2 密码未解密：连接前没有解密加密密码
3. SSH 握手无超时：`ssh.Dial` 的 Timeout 只控制 TCP connect，不控制握手阶段

AI 在这里的价值是**系统性地列出所有可能原因**，而不是只看最表面的那一层。我给它看了日志，它把三个问题都找出来了，然后逐一修复。

如果只靠自己看，很可能修了第一个，发现还有问题，再修第二个，再发现还有问题——来回三轮。AI 一次性把所有问题列出来，效率差很多。

---

### AI 在复杂系统里的边界

用了 33 天之后，对 AI 能做什么有了比较清晰的认知：

**擅长的：**

- 跨多个文件追踪调用链
- 系统性列出所有 edge case（比人脑更不容易漏）
- 生成符合项目规范的样板代码
- 写测试脚本（特别是多角色权限测试）
- 分析日志、定位根因

**不擅长的：**
- 不知道环境的实际状态（需要你告诉它）
- 环境特有的配置问题（NFS、ConfigMap、镜像仓库认证）
- 判断"这个改动会不会影响生产"（它没有生产环境的上下文）

所以我的用法是：**让 AI 做代码层面的事，环境层面的事我来确认**。它追踪代码路径、生成修复方案，我来验证环境状态、确认部署结果。

---

## 最重要的一条经验

> 上下文越精准，AI 越不需要猜，你越不需要纠正。

我有一次调试数据注入流程，Claude 连续走错路——先怀疑清理任务，再在 crictl/nerdctl/ctr 之间反复失败。我忍不住说了一句"能不能高效"。

事后复盘，根本原因是我没给它足够的上下文。它不知道环境的具体配置，只能靠猜。

工具再强，没有好的上下文，它只能靠猜。CLAUDE.md、MEMORY.md、Skills 这套体系，本质上都是在解决同一个问题：**让 AI 在每次会话里都能快速进入状态，而不是从零开始摸索。**

