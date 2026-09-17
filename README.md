# Code Skills

用于长期软件开发项目的 Skills 合集，同时覆盖 Codex 与 ZCode 客户端。

- `development-orchestrator`：Codex 版编排 skill，派发子代理时按次显式传入模型与推理等级。
- `zcode-dev-orchestrator`：ZCode 版编排 skill，将子任务路由到预置机队卡片（`.zcode/agents/`），模型档位固化在卡片上。

两者共享同一套编排思想（需求审查、最小任务图、批次并行、证据验收、会话交接），只在“执行配置如何表达”这一层按客户端机制分叉。

## development-orchestrator

`development-orchestrator` 是一个面向真实软件开发需求的编排 skill。它帮助 Codex 在实现前审查目标与技术方案、判断是否值得使用子代理、为子任务划分边界、整合验证结果，并在适当节点核实项目状态和建议交接新会话。

它适用于功能开发、缺陷修复、重构、迁移和接续长期项目；不适用于纯知识问答、翻译或一般讨论。

### 它解决的问题

- 用户提出技术方案时，区分“想达成的结果”和“建议的实现方式”，根据代码、约束、成本和验收标准判断是否应调整方向。
- 小而明确的任务直接完成，不为使用多代理而制造流程。
- 复杂任务先检查共享接口和依赖，再按模块或文件所有权安排调查、实现或独立审查，减少重复探索和写入冲突。
- 主代理负责最终整合和验证；子代理的结论不会自动视为功能完成。
- 一旦决定创建子代理，主代理必须在实际启动时显式传入适合该任务的模型和推理等级；不会因缺少项目默认值而静默继承父代理设置。
- 在任务开始、重要接口变更、任务完成、集成子代理结果或发现矛盾时，核对目标、代码、约束、验证和下一步。
- 在阶段完成或同一会话完成三个已验收任务单元后，整理项目状态并建议下一项开发在新会话开始。该周期是预防性维护，不意味着模型一定已经失效。

### 典型使用方式

```text
请使用 $development-orchestrator 完成退款流程迁移，保持旧 API 兼容，并补充必要测试。
```

如果希望它在所有实际开发任务中优先执行，应在个人 Codex 说明或项目 `AGENTS.md` 中加入类似规则：

```text
实际开发任务优先使用 development-orchestrator。由 Codex 判断是否需要子代理，负责任务拆解、结果整合、验证和会话交接检查。
```

### 安装

将目录复制到个人全局技能目录：

```text
~/.agents/skills/development-orchestrator/
```

或将其保留在项目中：

```text
<repo>/.agents/skills/development-orchestrator/
```

新启动的 Codex 会话会发现该 skill。可使用 `$development-orchestrator` 显式调用；描述与需求匹配时也可被自动选择。

### 使用边界

该 skill 是工作流程，不是后台监控器，也不保证发现所有模型错误。它不会擅自改变产品目标、兼容承诺或成本边界；这些需要由用户决策。它只在当前工具实际支持的范围内选择代理配置，并且不把未执行的代理工作或测试写成已完成。

对于项目状态汇报、资料核实和局部只读审阅，主代理通常直接完成；广泛审阅则先拆出独立调查路径，并按照当前可用并发槽位和可安全整合的结果数分批派发代理。若当前工具无法设置单个子代理的模型或推理等级，skill 会明确报告这个限制与实际回退配置。

## zcode-dev-orchestrator

`zcode-dev-orchestrator` 是 development-orchestrator 的 ZCode 适配版。ZCode 的 Agent 派发调用没有模型与推理等级参数，模型档位配置在子代理定义（机队卡片）上。因此该版本将“按任务性质选配置”改为“按任务性质选执行者”：skill 将子任务路由到三张预置机队卡片，档位差异由卡片固化。

### 预置机队

| 卡片 | 职责 | 档位 | 工具 |
|---|---|---|---|
| `orch-scout` | 只读调查、状态核实、代码地图 | GLM-5.3-Flash | Read, Bash |
| `orch-builder` | 限定范围内的实现 | 继承会话模型 | 全部工具 |
| `orch-auditor` | 独立证据审查 | GLM-5.3 | Read, Bash |

机队按档位划分角色，不按业务领域划分；项目特异性由每次派发的任务消息承载，不为单个项目新造卡片。卡片缺失时按 skill 内置规格重建。

无匹配机队角色时回退到 ZCode 内置类型：`Explore`（只读调查）、`general-purpose`（需要写入或多步执行）、`judge`（渲染交付物的视觉验收）。

### 安装

skill 复制到全局技能目录（与 Codex 共用 `~/.agents/skills/`，其描述限定“在 ZCode 客户端中”，不会被 Codex 误触发）：

```text
~/.agents/skills/zcode-dev-orchestrator/
```

机队卡片复制到 ZCode 用户级子代理目录：

```text
~/.zcode/agents/
```

或将本仓库直接作为 ZCode 工作区打开：仓库内的 `.zcode/agents/` 会作为项目级机队被自动发现。

### 生效时机

ZCode 在会话启动时扫描子代理名单和技能，会话中新建的文件下个会话生效。首次安装后需要新开一个会话。

### 使用方式

```text
请使用 $zcode-dev-orchestrator 完成退款流程迁移，保持旧 API 兼容，并补充必要测试。
```

编排行为（是否分工、任务图、批次、验收、交接检查）与 Codex 版一致；差异只在执行配置层，详见其 `references/delegation.md` 中的机队路由表。

## 目录

```text
.agents/skills/
├── development-orchestrator/        # Codex 版：派发时按次传模型与推理等级
│   ├── SKILL.md
│   ├── agents/openai.yaml           # UI 元数据
│   └── references/
│       ├── delegation.md
│       └── session-checkpoints.md
└── zcode-dev-orchestrator/          # ZCode 版：路由到预置机队卡片
    ├── SKILL.md
    └── references/
        ├── delegation.md            # 含机队路由表与机队规格
        └── session-checkpoints.md

.zcode/agents/                       # ZCode 机队卡片（项目级）
├── orch-scout.md
├── orch-builder.md
└── orch-auditor.md
```
