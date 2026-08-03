# Matt Pocock 技能

由 Claude Code 加载的代理技能（斜杠命令和行为）集合。技能被组织成分类，并使用由 `/setup-matt-pocock-skills` 生成的每个仓库配置进行消费。

## 语言

**Issue tracker**：托管仓库问题的工具 — GitHub Issues、Linear、本地 .scratch/ markdown 规范或类似工具。诸如 `to-tickets`、`to-spec`、`triage` 和 `qa` 之类的技能会从中读取和写入。
*避免*：待办事项管理器、待办事项后端、问题托管工具

**Issue**：Issue tracker 内部的一个单一跟踪工作单元 — 由 `to-tickets` 产生的 bug、任务、规范或切片。
*避免*：ticket（仅用于引用称其为 ticket 的外部系统时，或用于 **Decision ticket** — 见下文）

**Decision ticket**：一个 `wayfinder` 单位 — 一个持有问题的 `wayfinder:map` 的子 Issue，其解决结果是一个决策，而不是要执行的构建切片。`decision` 限定符是将其与实现 ticket 区分开来的关键；`wayfinder` 引入了这个术语，然后使用 "ticket"。

**Triage role**：在分类过程中应用于 Issue 的规范状态机标签（例如 `needs-triage`、`ready-for-afk`）。每个角色通过 `docs/agents/triage-labels.md` 映射到 Issue tracker 中的真实标签字符串。

## 关系

* 一个 Issue tracker 持有多个 Issues
* 一个 Issue 同时只能携带一个 Triage role
* 一个 Decision ticket 是一个 Issue（`wayfinder:map` 的子项）

## 标记的歧义

* "backlog" 以前被用来指代托管问题的工具以及其中的工作内容 —— 已解决：该工具是 Issue tracker；"backlog" 不再作为领域术语使用。
* "backlog backend" / "backlog manager" — 已解决：合并为 Issue tracker。
