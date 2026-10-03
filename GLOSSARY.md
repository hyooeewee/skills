# Matt Pocock 技能

由 Claude Code 加载的一组 agent 技能（斜杠命令和行为）。技能按桶（buckets）组织，并由 `/setup-matt-pocock-skills` 生成的每个仓库配置使用。

## 语言

**Issue 跟踪器**：
托管仓库问题的工具：GitHub Issues, Linear，本地 `.scratch/` markdown 约定，或类似工具。像 `to-tickets`、`to-spec` 和 `triage` 这样的技能会从中读取和写入。
*避免使用*：backlog manager, backlog backend, issue host

**Issue**：
**Issue tracker** 内的一个单独跟踪的工作单元：由 `to-tickets` 生成的 bug、任务、spec 或 slice。
*避免使用*：ticket（仅在引用将它们称为 tickets 的外部系统时使用，或者对于 **Decision ticket**，请参阅下文）

**决策 ticket**：
一个 `wayfinder` 单元：一个持有 *问题* 的 `wayfinder:map` 的子级 **Issue**，其解决方案是一个决策，而不是要执行的构建片段。**decision** 限定词使其与实施 ticket 区分开来；`wayfinder` 引入了这个术语，然后使用 "ticket"。

**Triage role**：
分诊期间应用于 **Issue** 的规范状态机标签（例如 `needs-triage`、`ready-for-afk`）。每个角色通过 `docs/agents/triage-labels.md` 映射到 **Issue tracker** 中的实际标签字符串。

## 关系

* * 一个 **Issue tracker** 包含许多 **Issue**
* * 一个 **Issue** 一次只承担一个 **Triage role**
* * 一个 **Decision ticket** 是一个 **Issue**（`wayfinder:map` 的子项）

## 已标记的歧义

* "backlog" 之前被用来同时指代托管问题的 *工具* 和其中包含的 *工作体*。已解决：该工具是 **Issue tracker**；"backlog" 不再作为领域术语使用。
* "backlog backend" / "backlog manager"。已解决：合并为 **Issue tracker**。
