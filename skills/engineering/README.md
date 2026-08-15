# 工程

我日常处理代码工作所用的技能。

## 用户调用

仅在您输入它们时才可访问（Claude Code：`disable-model-invocation: true`；Codex：`agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

* **[ask-matt](./ask-matt/SKILL.md)** — 询问哪种技能或流程适合你的情况。这是对本仓库中用户调用技能的路由器。
* **[grill-with-docs](./grill-with-docs/SKILL.md)** — 一种拷问式研讨，同时构建项目的领域模型，锐化术语，并内联更新 `CONTEXT.md` 和 ADR。
* **[triage](./triage/SKILL.md)** — 将议题在分流角色的状态机中流转。
* **[improve-codebase-architecture](./improve-codebase-architecture/SKILL.md)** — 扫描代码库，寻找可深化的机会，将其呈现为可视化 HTML 报告，然后对你选择的任何一个进行拷问式研讨。
* **[setup-matt-pocock-skills](./setup-matt-pocock-skills/SKILL.md)** — 为本仓库配置工程技能（议题追踪器、分流标签、领域文档布局）。每个仓库运行一次。
* **[to-spec](./to-spec/SKILL.md)** — 将当前对话转化为一份规格说明，并发布到议题追踪器。
* **[to-tickets](./to-tickets/SKILL.md)** — 将任何计划、规格或对话分解为一组曳光弹式票据，每张票据都声明其阻塞边界——可以是本地文件中的文本，也可以是真实追踪器上的原生阻塞链接。
* **[implement](./implement/SKILL.md)** — 构建由规格或一组票据所描述的工作，在预先约定的接缝处驱动 `/tdd`，并在提交前以 `/code-review` 收尾。
* **[wayfinder](./wayfinder/SKILL.md)** — 将一大块工作——超过一个代理会话所能承载的量——规划为议题追踪器上的一张共享决策票据地图，逐一解决，直到通往目的地的路径清晰为止。

## 模型调用

模型或用户均可访问（采用丰富的触发措辞，便于模型主动调用）。

* **[prototype](./prototype/SKILL.md)** — 构建一个一次性原型来回答设计问题：一个可共享的 HTML 文件用于状态/逻辑，或几个可切换的 UI 变体。

* **[diagnosing-bugs](./diagnosing-bugs/SKILL.md)** — 针对疑难 bug 和性能回归的纪律性诊断循环：构建一个反馈循环，让此 bug 变红 → 最小化 → 假设 → 插桩 → 修复 → 回归测试。

* **[research](./research/SKILL.md)** — 针对高可信度的一手来源调查一个问题，并将发现以带引用的 Markdown 文件捕获到仓库中，作为后台代理运行。

* **[tdd](./tdd/SKILL.md)** — 采用红-绿-重构循环的测试驱动开发。一次一个垂直切片地构建功能或修复 bug。

* **[domain-modeling](./domain-modeling/SKILL.md)** — 主动构建并打磨项目的领域模型——挑战术语、用场景进行压力测试、内联更新 `CONTEXT.md` 和 ADR。

* **[codebase-design](./codebase-design/SKILL.md)** — 设计深模块时共享的纪律和词汇：小接口、干净的接缝、可通过接口测试。

* **[code-review](./code-review/SKILL.md)** — 自某个固定点以来，对 diff 进行双轴审查：**标准**（是否符合仓库的编码标准，外加 Fowler 的坏味道基线？）和 **规格**（是否忠实地实现了原始议题/规格？），以并行子代理方式运行。

* **[resolving-merge-conflicts](./resolving-merge-conflicts/SKILL.md)** — 逐块处理进行中的 git merge 或 rebase 冲突，根据追溯到双方一手来源的意图来解决，然后完成操作——绝不使用 `--abort`。

* **[wizard](./wizard/SKILL.md)** — 生成一个交互式 bash 向导，引导人类完成只有他们才能执行的步骤：配置基础设施、设置凭据或 CI 机密、熟悉陌生的第三方仪表盘，或运行一次性迁移或切换。
