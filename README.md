# Code Skills

用于长期软件开发项目的 Codex Skills 集合。

## development-orchestrator

`development-orchestrator` 是一个面向真实软件开发需求的编排 skill。它帮助 Codex 在实现前审查目标与技术方案、判断是否值得使用子代理、为子任务划分边界、整合验证结果，并在适当节点核实项目状态和建议交接新会话。

它适用于功能开发、缺陷修复、重构、迁移和接续长期项目；不适用于纯知识问答、翻译或一般讨论。

### 它解决的问题

- 用户提出技术方案时，区分“想达成的结果”和“建议的实现方式”，根据代码、约束、成本和验收标准判断是否应调整方向。
- 小而明确的任务直接完成，不为使用多代理而制造流程。
- 复杂任务先检查共享接口和依赖，再按模块或文件所有权安排调查、实现或独立审查，减少重复探索和写入冲突。
- 主代理负责最终整合和验证；子代理的结论不会自动视为功能完成。
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

### 目录

```text
.agents/skills/development-orchestrator/
├── SKILL.md                         # 主流程：需求审查、分工、验证
├── agents/openai.yaml               # UI 元数据
└── references/
    ├── delegation.md                # 子代理任务约定与配置选择
    └── session-checkpoints.md       # 会话检查与交接规则
```

### 使用边界

该 skill 是工作流程，不是后台监控器，也不保证发现所有模型错误。它不会擅自改变产品目标、兼容承诺或成本边界；这些需要由用户决策。它只在当前工具实际支持的范围内选择代理配置，并且不把未执行的代理工作或测试写成已完成。
