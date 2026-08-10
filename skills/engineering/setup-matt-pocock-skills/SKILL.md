---
name: setup-matt-pocock-skills
description: 为此工程技能配置此仓库 — 设置其问题跟踪器、分类标签词汇表和领域文档布局。在首次使用其他工程技能之前运行一次。
disable-model-invocation: true

---

# 设置 Matt Pocock 的技能

构建工程技能假设的每仓库配置：

* **问题跟踪器** — 问题所在位置（默认为 GitHub；本地 Markdown 也开箱即用支持）
* **分类标签** — 用于五个规范分类角色的字符串
* **领域文档** — `CONTEXT.md` 和 ADR 所在位置，以及阅读它们的消费规则

这是一个基于提示的技能，而非确定性脚本。探索、展示你发现的内容、与用户确认，然后编写。

## 流程

### 1. 探索

查看当前仓库以了解其初始状态。读取任何存在的内容；不要假设：

* `git remote -v` 和 `.git/config` — 这是 GitHub 仓库吗？是哪一个？
* 仓库根目录下的 `AGENTS.md` 和 `CLAUDE.md` — 两者都存在吗？其中是否有 `## Agent skills` 部分？
* 仓库根目录下的 `CONTEXT.md` 和 `CONTEXT-MAP.md`
* `docs/adr/` 和任何 `src/*/docs/adr/` 目录
* `docs/agents/` — 此技能的先前输出是否已经存在？
* `.scratch/` — 本地 Markdown 问题跟踪器约定已在使用中的标志
* 是否安装了 `triage` 技能？（此技能旁边的 `triage` 技能文件夹，或你可用技能中的 `triage`）。这决定了 Section B 是否会运行。
* Monorepo 信号 — `pnpm-workspace.yaml`，`package.json` 中的 `workspaces` 字段，或带有其自身 `src/` 的已填充 `packages/*`。仅出现在真正的大型多包仓库中；它们的缺失意味着单上下文，这几乎适用于每个仓库。

### 2. 展示发现并提问

总结存在的内容和缺失的内容。然后按顺序处理各个部分 — 一个部分、一个答案，然后是下一个。

在每个部分开头加上推荐的答案，以便用户可以用一个词接受。只有当选择真正分叉时才给出一行解释；当探索已经确定时，完全跳过该部分（当未安装 `triage` 时的 Section B，当没有 monorepo 时的 Section C）。

**Section A — 问题跟踪器。**

> 解释器："问题跟踪器"是此仓库问题的所在位置。像 `to-tickets`、`triage` 和 `to-spec` 这样的技能从中读取和写入——它们需要知道是调用 `gh issue create`、在 `.scratch/` 下写入 markdown 文件，还是遵循你描述的其他工作流。选择你实际跟踪此仓库工作的位置。

默认姿态：这些技能是为 GitHub 设计的。如果 `git remote` 指向 GitHub，则建议使用它。如果 `git remote` 指向 GitLab（`gitlab.com` 或自托管主机），则建议使用 GitLab。否则（或如果用户更喜欢），提供：

* **GitHub** — 问题存在于仓库的 GitHub Issues 中（使用 `gh` CLI）
* **GitLab** — 问题存在于仓库的 GitLab Issues 中（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
* **本地 Markdown** — 问题作为此仓库 `.scratch/<feature>/` 下的文件存在（适合单人项目或没有远程的仓库）
* **其他**（Jira、Linear 等） — 请用户用一段话描述工作流；该技能将其记录为自由形式散文

在 `docs/agents/issue-tracker.md` 中记录选择。GitHub 和 GitLab 模板带有“PR 作为请求表面”标志，默认为 **关闭** — 保持关闭，不要提出；想要在分类队列中有外部 PR 的用户可以在文件中稍后翻转该标志。

**Section B — 分类标签词汇表。** 如果 `triage` 技能未安装（探索结果已告知你），请完全跳过此部分 — 未安装的技能不需要标签。

如果已安装，只问一个问题：

> 你想保留默认的分类标签吗？（推荐：**是**）

默认值是五个规范角色，每个标签字符串等于其名称：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。在 **是** 上，按原样写入它们。只有当用户说 **不** 时（通常是因为他们的跟踪器已经使用其他名称，例如 `needs-triage` 用 `bug:triage`） — 收集覆盖项，以便 `triage` 应用现有标签而不是创建重复项。

**Section C — 领域文档。** 默认为 **单上下文** — 仓库根目录下的一个 `CONTEXT.md` + `docs/adr/`。这几乎适用于每个仓库；不问就写。

提供 **多上下文** — 根 `CONTEXT-MAP.md` 指向每个上下文的 `CONTEXT.md` 文件 — 仅当探索发现 monorepo 信号时。然后确认他们想要哪个布局。

### 3. 确认并编辑

向用户展示草稿：

* 要添加到正在编辑的 `CLAUDE.md` / `AGENTS.md` 中的 `## Agent skills` 块（请参阅步骤 4 的选择规则）
* `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md` 的内容（后者仅在安装 `triage` 时）

在写入前让他们编辑。

### 4. 写入

**选择要编辑的文件：**

* 如果 `CLAUDE.md` 存在，则编辑它。
* 否则如果 `AGENTS.md` 存在，则编辑它。
* 如果都不存在，询问用户要创建哪一个 — 不要为他们选择。

永远不要在 `CLAUDE.md` 已存在时创建 `AGENTS.md`（反之亦然） — 始终编辑已经存在的那个。

如果所选文件中已存在 `## Agent skills` 块，则原地更新其内容，而不是追加重复项。不要覆盖对周围部分的用户编辑。

块：

```markdown
## Agent skills

### Issue tracker

[one-line summary of where issues are tracked]. See `docs/agents/issue-tracker.md`.

### Triage labels

[one-line summary of the label vocabulary]. See `docs/agents/triage-labels.md`.

### Domain docs

[one-line summary of layout — "single-context" or "multi-context"]. See `docs/agents/domain.md`.
```

包含 `### Triage labels` 子块，并写入 `docs/agents/triage-labels.md`，仅当安装了 `triage` 且运行了 Section B 时。当它未安装时，两者都被省略。

然后使用此技能文件夹中的种子模板作为起点写入文档文件：

* [issue-tracker-github.md](./issue-tracker-github.md) — GitHub 问题跟踪器
* [issue-tracker-gitlab.md](./issue-tracker-gitlab.md) — GitLab 问题跟踪器
* [issue-tracker-local.md](./issue-tracker-local.md) — 本地 Markdown 问题跟踪器
* [triage-labels.md](./triage-labels.md) — 标签映射（仅当安装 `triage` 时）
* [domain.md](./domain.md) — 领域文档消费规则 + 布局

对于“其他”问题跟踪器，使用用户的描述从头开始编写 `docs/agents/issue-tracker.md`。

### 5. 完成

告诉用户设置已完成，以及现在将从哪些文件读取哪些工程技能。提到他们可以稍后直接编辑 `docs/agents/*.md` — 重新运行此技能仅在想要切换问题跟踪器或从头开始重置时才有必要。
