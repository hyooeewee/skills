# Matt Pocock 技能

由 Claude Code 加载的一组 agent 技能（斜杠命令和行为）。技能按桶（buckets）组织，并由 `/setup-matt-pocock-skills` 生成的每个仓库配置使用。

## 语言

**Issue tracker**：
托管仓库问题（issues）的工具——GitHub Issues、Linear、本地 `.scratch/` Markdown 约定或类似工具。`to-tickets`、`to-spec` 和 `triage` 等技能会读写该工具。
*避免*：backlog manager、backlog backend、issue host

**Issue**：
**Issue tracker** 中的一个被跟踪的工作单元——由 `to-tickets` 产生的 bug、任务、规格或切片。
*避免*：ticket（仅在引用外部系统称其为 ticket 时使用，或用于 **Decision ticket** ——见下文）

**Decision ticket**：
一个 `wayfinder` 单元——`wayfinder:map` 的子 **Issue**，包含一个*问题*，其解决结果是一个决策，而不是要执行的构建切片。**decision** 限定词使其区别于实现 ticket；`wayfinder` 引入了该术语，然后使用 "ticket"。

**Triage role**：
分诊期间应用于 **Issue** 的规范状态机标签（例如 `needs-triage`、`ready-for-afk`）。每个角色通过 `docs/agents/triage-labels.md` 映射到 **Issue tracker** 中的实际标签字符串。

## 关系

* * 一个 **Issue tracker** 包含许多 **Issue**
* * 一个 **Issue** 一次只承担一个 **Triage role**
* * 一个 **Decision ticket** 是一个 **Issue**（`wayfinder:map` 的子项）

## 已标记的歧义

* * "backlog" 此前既用于指托管问题的*工具*，也用于指其中的*工作主体*——已解决：该工具是 **Issue tracker**；"backlog" 不再作为领域术语使用。
* * "backlog backend" / "backlog manager" ——已解决：合并到 **Issue tracker** 中。
