# Requirement Orchestrator（需求编排器）

把一个软件请求拆成**可独立验证**的工作，同时始终由**一个控制 agent** 对范围、依赖、证据与集成负责。它是一个编排器，不是提示词生成器。

> 规范定义在 [`SKILL.md`](SKILL.md)；本 README 是给人读的导览。二者冲突时以 `SKILL.md` 为准。

## 解决什么痛点

在真实代码库里做多步骤 agent 工作,翻车往往不是逻辑错,而是:范围悄悄膨胀、上下文丢失/过时、并行写静默冲突、"完成"未经证据验证、状态没有唯一负责人——跨会话/跨平台(Codex↔Claude)尤甚。本 skill 用**单一控制 agent + ledger(账本)+ 冻结契约 + 证据门验收**,把"分解 → 委派 → 集成"钉成可复现、可审计的流程。

## 环境与依赖

- 纯**方法论 skill(只有 Markdown)**:无运行时、无 `pip`/`npm` 依赖、不装任何包。
- 需要一个能读 skill / `SKILL.md` 的 **agent 宿主**:Claude Code(omp)、Codex,或任何你让它读 `SKILL.md` 的 agent。
- 可选集成:[skills-radar](https://github.com/tomtdhzz/skills-radar) 作为"起点能力库"(见 `references/knowledge-base.md`)。

## 安装

```bash
# 方式一:skills CLI(装到 ~/.claude/skills 或对应 agent 目录)
npx skills add tomtdhzz/requirement-orchestrator -g -y

# 方式二:手动 clone 后软链到你的 agent skills 目录
git clone https://github.com/tomtdhzz/requirement-orchestrator.git
ln -s "$PWD/requirement-orchestrator" ~/.claude/skills/requirement-orchestrator
```
或直接让你的 agent 读仓库里的 `SKILL.md`。

## 使用

- **触发**:对 agent 说"用 requirement-orchestrator 分析/编排这个需求……";当请求需要分解+委派+验证时它也会自动启用。
- **选模式**:`analyze`(默认,只读蓝图)/ `diagnose` / `execute`(需你单独授权改代码)/ `challenge`。
- **过程**:agent 先读 `SKILL.md`,按需加载 `references/*`;维护分阶段 TODO 与 ledger;`execute` 完成后按证据门验收。
- **回灌(可选)**:任务完成后,项目教训自动记入 `.ai-work/lessons.md`;是否把**通用经验回灌进本 skill** 是一个 opt-in 选项(默认关、需你同意、单独提交)——见 `references/experience.md`。
- 想看完整走一遍:`references/examples/feature-example.md`、`bug-example.md`。

## 何时用

需求、功能、缺陷、服务变更或领域变更需要被拆分、委派给子 agent、并经验证与集成来管控时。典型触发：

- 一个请求包含多个候选任务，或跨服务/领域边界；
- 需要调度决策（谁先谁后、能否并行）；
- 构建/部署失败且仓库状态可能是诱因；
- 需要跨 Codex / Claude 协作或交接控制权。

单一根因、无需委派的小任务：直接走控制循环即可，不必拉起全套机制。

## 四种语义模式

模式是**语义**层的，不等于平台的原生 Plan Mode，也不强制 `EnterPlanMode` 或 Explore/Plan agent。

| 模式 | 做什么 | 是否改代码 |
|---|---|---|
| `analyze` | 调查请求与代码、维护 ledger、产出执行蓝图 | 否（默认模式） |
| `diagnose` | 复现并解释故障，分清确证根因/证据/未知/修复路径 | 否 |
| `execute` | 派发有界工作、评审回传结果、完成集成 | 是（**需用户单独授权**） |
| `challenge` | 检验既有需求/分解/设计的遗漏与风险，不改其已确认目标 | 否 |

`analyze` 与 `diagnose` 是**只读**的：平台权限或原生 plan 批准都不构成改代码的授权，必须另有用户指令进入 `execute`。

## 控制循环

1. 确立目标、范围、验收标准、约束与已核实的代码事实；能查的自己查，只在实质影响范围/验收/方向时才问用户。
2. 缺陷/故障先做等价成本的本地复现再下根因；未复现前根因只作假设并标注未核实项。
3. 选**一条**主分解轴，建有界任务，显式记依赖与跨任务契约；把任务组镜像成**分阶段 TODO**（见下）。
4. 只保留一个控制 agent：它独占 ledger、派发、评审状态、重规划与最终集成。
5. 只派发契约完整的任务；对本地工作树以外的写入，派发前先跑**目标系统预检**。
6. 结果先入 `review`，校验证据与边界后才置 `completed` 或带具体意见退回；批量/外部改动用**独立回读 + 增量校验**，不信工具自报的成功数。
7. 出现发现、失败或范围变化后重算受影响依赖，只返工最小分支。
8. 任务级检查、跨任务契约、端到端验收全过，才宣告完成。

## 进度面（Progress surface）

维护一份与 ledger 同步的**可见分阶段 TODO**，让用户看到阶段与步级进度，而不只是散文。ledger 是详细真相层（依赖/范围/契约/证据），分阶段 TODO 是它的进度视图。

- 每个分解组一个 phase，末尾加一个验收 phase（缺陷流：`对照`/`修复`/`验证`；功能流：各功能组 + `全量验收`）。每项是一个有界任务或验证步，5–10 词的结果式描述。
- 用平台原生任务列表驱动，绝不手搓树。Claude/omp 用 `todo`（`init` 传 `list:[{phase,items}]`，再逐项 `start`/`done`）；Codex 用其 plan/update-plan。
- 进度由真实推进驱动：任务派发时标 in-progress，只有 ledger 记 `completed` 才标 done。阶段计数 `N/M` 反映的是已验收，不是已派发。

## 目标系统预检（Target-system preflight）

向本地工作树**以外**的系统（wiki、工单、数据库、远程 API）批量写入**前**，先探明：

- 写权限与操作所需的确切 scope（若计划含删除/移动也要）；
- 频控，以及失败是否静默；据此定安全并发与重试/退避；
- 结果能否回读验证。

缺能力要在这一步发现，而不是批量到一半才撞上。所需权限/scope 不可得时当阻塞项：说清缺什么、最小授权是什么，并**停在任何部分写入之前**。

## 不可逾越的边界

- 不把任务树位置当执行依赖。
- 不让 worker 悄悄扩写范围。
- 依赖未解、写范围重叠、共享契约未冻结、校验不能独立跑——四者不全满足，不并行跑写 agent。
- worker 只能提交评审，唯有控制 agent 能置 `completed`。
- 改既有工件时，凡本任务未产出且不可再生的内容不清空重写——就地插入或补丁，带可重跑标记，并独立回读验证。
- 派生值（分类/标签/摘要/风险评级/结论）不得超出其已核实来源；未核实的派生一律标记，绝不编造填槽。
- 不因为存在 `.trellis/` 就启用 Trellis。

## 目录结构

```
requirement-orchestrator/
├── SKILL.md                      # 规范：模式、控制循环、进度面、预检、边界（权威）
├── README.md                     # 本文件：中文导览
├── agents/
│   └── openai.yaml               # 对外展示名与默认提示
├── docs/
│   └── experience-loop.md        # 经验闭环的设计依据（为何最小化）
└── references/                   # 按需加载，非一次性全读
    ├── spec-driven.md            # 规格模型：Requirements / Acceptance Scenarios / Contracts
    ├── tech-design.md            # 技术设计文档（how）：架构/详设/备选权衡/横切关注点，业界 TDD/RFC 结构
    ├── deliverables.md           # 交付物布局：可发布 docs(prd/tech-design) vs 内部 .ai-work(ledger/plan/lessons)；可发布项目 README/LICENSE/.gitignore
    ├── context-grounding.md      # 分解前接地：架构/依赖/约定/影响面 → 记为事实
    ├── decomposition.md          # 分解与调度：选主轴、缺陷三分诊、依赖、并行门、重规划
    ├── verification.md           # 按类型的验收证据标准 + 控制方评审 + 防幻觉
    ├── challenge.md              # 压力测试：质量维度对抗 + 遗漏/风险清单
    ├── ledger.md                 # 需求 ledger：机器状态 YAML、状态机、出处纪律、交接快照
    ├── agent-contract.md         # 子 agent 契约：派发必填字段、worker 义务、控制方评审
    ├── mutation.md               # 安全批量改动：非破坏式、幂等、试点→批量、增量+回读校验
    ├── knowledge-base.md         # 起点先查能力库（可选集成，如 skills-radar）
    ├── experience.md             # 经验闭环：项目教训自动 append；skill 自身改进=opt-in（默认关）
    ├── examples/                 # 端到端范例：feature-example.md、bug-example.md
    ├── codex-adapter.md          # Codex 平台适配
    ├── claude-adapter.md         # Claude 平台适配
    └── trellis-adapter.md        # Trellis 适配（可选）
```

### 按需读哪份

- 塑造规格（需求/验收场景/契约）→ `spec-driven.md`；在既有代码库里分解前接地 → `context-grounding.md`
- 非平凡构建的技术设计（架构/详设/备选/横切）→ `tech-design.md`；交付物布局与可发布项目 → `deliverables.md`
- 多个候选任务 / 服务·领域边界 / 调度决策 / 疑似仓库状态诱发的构建失败 → `decomposition.md`
- 验收证据标准 → `verification.md`；压力测试既有需求/设计 → `challenge.md`
- 多任务、多 agent、跨会话或跨平台 → `ledger.md`；首次派发前 → `agent-contract.md`
- 改写/删除/跨多个既有工件铺开变更，或写外部系统 → `mutation.md`
- 起点先查已有能力（可选)→ `knowledge-base.md`；沉淀项目教训、或把经验回灌进 skill（opt-in，默认关）→ `experience.md`
- 完整走一遍的范例 → `examples/`；具体平台 → 对应 `*-adapter.md`

## 核心概念：ledger（账本）

跨 agent / 会话 / 平台的**共享真相源**。存决策与证据，不存隐藏推理或完整聊天记录。

- 小的单会话任务可留在对话里；多 agent / 跨会话 / 跨平台则落盘持久化。
- 用可读 Markdown 包一个受控 YAML 块记机器状态：`request` / `analysis` / `tasks` / `integration`。
- 状态机：`pending → in_progress → review → completed`，另有 `blocked` / `failed`；worker 只报 `review`/`blocked`/`failed`，`completed` 只由控制方在验证后记录。
- 切换控制平台前，先写并确认**交接快照**（当前模式与阶段、决策与开口、在跑任务与归属、阻塞与未验证结果、写范围与冻结契约、下一步）；新控制方确认后旧控制方才停止派发。

## 平台路由与交接

- Codex / Claude 各读对应 adapter；主 agent/会话默认保持控制权，除非交接快照显式移交。
- 两者互转控制权时，旧控制方停止派发**之前**先写并确认交接快照。
- Trellis 可选：仅在其升级信号适用或用户要求时才读 `trellis-adapter.md`，不因目录存在而自动启用。
