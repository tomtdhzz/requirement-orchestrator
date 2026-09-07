# Requirement Orchestrator

> **让 agent 说的「做完了」变成可核对的事实。**
> 零安装 · 不要求 spec · 并行安全看**编译/测试目标**，完成与否看 **`base_commit..HEAD` 的 diff**。

**中文** · [English](README.en.md) · [MIT](LICENSE) · 纯 Markdown，零依赖

## TL;DR

给 AI coding agent 用的**委派与验收控制层**。它只回答两个问题：**扇出前必须冻结什么**、**凭什么算做完**。

- 零前置：不装包、不要 spec、不改仓库结构。
- 判据而非印象：并行安全看**编译/测试目标**，完成与否看 **`base_commit..HEAD` 的 diff**。
- 给一个人用。一人多 agent 时，瓶颈是验收，不是产能。

## 特性

**它解决什么**

| 场景 | 没有它 | 有它 |
|---|---|---|
| 两个 agent 改同一 package 的不同文件 | 双方 `go test` 互相打红，半小时才定位 | 并行门按编译/测试目标判定，且必须留下扫描表 |
| worker 报「done, tests pass」 | 信报告，三天后发现改动不在 diff 里 | 控制方读 `base_commit..HEAD`、复跑验证命令后才记 `completed` |
| 需求只有一句口头描述 | 直接开写，做完了才发现验收标准没定 | 现场立最小规格：一条需求 + 一个验收场景 + 冻结契约 |
| 跨会话续跑 | `in_progress` 被当成「做了一半」继续推进 | 一律降级为未验证，核对工件后再决定重派或作废 |

**核心机制**

- 单一控制 agent 持有 ledger、派发、评审与集成；worker 只能提交 `review`。
- 契约在扇出前冻结，用 `<frozen-after-approval>` 标出边界，可 grep。
- 派发契约写 `run` / `expect` 命令，不写意图；空的 `verification` 视为不完整契约。
- 复评只对既有 finding 定 verdict，五轮封顶，只在上限处裁决。
- 驳回也要出证据：`false` 附反证，`unverified` 附待查项。

**不做**

- 任务库、看板、仪表盘、进度同步。
- 生成 spec 本身（仓库已有 spec 就用它，绝不造第二份）。
- 任务排序调度算法、云端后端、绑定单一宿主的运行时。

**限制**

- **无强制力**：规则是可观察的检查点，不是阻止。真正的阻止要靠宿主 hooks / CI。
- 并行门是判据不是锁，工作树隔离取决于你的 git 习惯。
- ledger 是文件不是数据库，损坏或过期时没有仲裁者。

## 快速开始

```bash
npx skills add tomtdhzz/requirement-orchestrator -g -y
```

```text
用 requirement-orchestrator 分析这个需求：给订单接口加个限流。
```

它产出规格、验收场景、任务切分与并行判定后**停下等你授权**——`analyze` 只读，改代码需要另一句明确指令。

## 核心配置

**四种语义模式**

| 模式 | 用途 | 改代码 |
|---|---|---|
| `analyze` | 调查 + 产出执行蓝图（默认；尚无故障时） | 否 |
| `diagnose` | 复现并解释故障（**请求提到已发生的故障即走这里**） | 否 |
| `execute` | 派发、评审、集成 | 是，需单独授权 |
| `challenge` | 压力测试既有需求／设计／实现（代码与 PR 评审走这里） | 否 |

**落盘位置**

| 路径 | 内容 | 是否入库 |
|---|---|---|
| `docs/prd/`、`docs/tech-design/` | 规格与技术设计 | 是 |
| `.ai-work/ledger.md` | 任务状态机、证据、决策 | 否（gitignore） |
| `.ai-work/plan.md`、`lessons.md` | 分阶段计划、项目教训 | 否 |

**依赖**

| 项 | 要求 |
|---|---|
| 运行时 | 无 |
| 宿主 | 任何能读 `SKILL.md` 的 agent（Claude Code / omp / Codex） |
| 可选 | [skills-radar](https://github.com/tomtdhzz/skills-radar) 作起点能力库 |

规范见 [`SKILL.md`](SKILL.md)（90 行，常驻）；细节见 [`references/`](references)（15 篇，按需加载）。冲突时以 `SKILL.md` 为准。

## 贡献指南

见 [CONTRIBUTING.md](CONTRIBUTING.md)。提规则改动必须写明四件事：

| 项 | 说明 |
|---|---|
| 缺口 | 没有它会出什么错，附真实运行或明确失败模式 |
| 后果类型 | `不可逆` / `返工一轮` / `噪音` |
| 代价落点 | 常驻上下文 / agent 往返 / 人工注意力 |
| 撤回条件 | 什么信号出现就删掉它 |

- 规则须能从它生效的控制环步骤一跳可达，否则等于不存在。
- `噪音` 级且占常驻上下文的规则一律拒绝。
- `CHANGELOG.md` 只记用户会注意到的改动；内部措辞修复靠 commit body。
