# 工程

我日常用于代码工作的技能。

## 用户调用

仅在输入时可用（Claude Code: `disable-model-invocation: true`; Codex: `agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

* **[ask-matt](./ask-matt/SKILL.md)** — 询问哪种技能或流程适合您的情况。本仓库中用户调用技能的路由。
* **[grill-with-docs](./grill-with-docs/SKILL.md)** — 深度审查会，同时也会构建您的项目领域模型，完善术语并内联更新 `CONTEXT.md` 和 ADR。
* **[triage](./triage/SKILL.md)** — 通过分类角色的状态机流转问题。
* **[improve-codebase-architecture](./improve-codebase-architecture/SKILL.md)** — 扫描代码库以寻找改进机会，将其呈现为视觉 HTML 报告，然后通过您选择的任何一个进行深入审查。
* **[setup-matt-pocock-skills](./setup-matt-pocock-skills/SKILL.md)** — 为工程技能配置此仓库（问题跟踪器、分类标签、领域文档布局）。每个仓库运行一次。
* **[to-spec](./to-spec/SKILL.md)** — 将当前对话转换为规范（spec）并发布到问题跟踪器。
* **[to-tickets](./to-tickets/SKILL.md)** — 将任何计划、规范或对话分解为一组追踪型工单（tracer-bullet tickets），每个工单声明其阻塞依赖关系——本地文件中的文本，或真实跟踪器上的原生阻塞链接。
* **[implement](./implement/SKILL.md)** — 构建规范或一组票据所描述的工作，在预定的接合点（seams）驱动 `/tdd`，并在提交前使用 `/code-review` 结束。
* **[wayfinder](./wayfinder/SKILL.md)** — 将大量工作（超过一个代理会话所能容纳）规划为问题跟踪器上的一张共享决策票据地图，逐一解决，直到通往目的地的路径清晰为止。

## 模型调用

模型或用户可达（丰富的触发短语，以便模型能够调用它们）。

* **[prototype](./prototype/SKILL.md)** — 构建一个一次性原型来回答设计问题：可运行的终端应用用于状态/逻辑，或几个可切换的 UI 变体。

* **[diagnosing-bugs](./diagnosing-bugs/SKILL.md)** — 针对顽固 Bug 和性能回退的严谨诊断循环：重现 → 最小化 → 假设 → 添加观测（instrument）→ 修复 → 回归测试。

* **[research](./research/SKILL.md)** — 针对高可信度的一手来源调查问题，并将发现作为引用的 Markdown 文件保存在仓库中，作为后台代理运行。

* **[tdd](./tdd/SKILL.md)** — 具有红-绿-重构循环的测试驱动开发（TDD）。一次构建一个功能或修复一个 Bug 的垂直切片。

* **[domain-modeling](./domain-modeling/SKILL.md)** — 主动构建和完善项目的领域模型——挑战术语，用场景进行压力测试，内联更新 `CONTEXT.md` 和 ADR。

* **[codebase-design](./codebase-design/SKILL.md)** — 设计深度模块的共享规范和词汇：小接口、干净的接合点、通过接口可测试。

* **[code-review](./code-review/SKILL.md)** — 对固定点以来差异的双轴审查：**标准**（是否遵循仓库的编码标准，以及 Fowler 味道基线？）和**规范**（是否忠实实现原始问题/PRD？），作为并行子代理运行。

* **[resolving-merge-conflicts](./resolving-merge-conflicts/SKILL.md)** — 逐一解决进行中的 git 合并或变基冲突块，通过追溯到每一方主要来源的意图来解决，然后完成操作——绝不 `--abort`。
