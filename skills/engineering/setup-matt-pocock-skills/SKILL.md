---
name: setup-matt-pocock-skills
description: 为工程技能配置此仓库：设置其 issue 跟踪器、分流标签词汇和领域文档布局。在其他工程技能首次使用前运行一次。
disable-model-invocation: true

---

# 设置 Matt Pocock 的技能

搭建工程技能所依赖的每仓库配置：

* Issue tracker：问题存放的位置（默认为 GitHub；本地 Markdown 也开箱即支持）
* Triage labels：用于五个标准分流角色的字符串
* 领域文档：`CONTEXT.md` 和 ADRs 存放的位置，以及阅读它们的消费规则

这是一个提示驱动的技能，不是确定性脚本。先探索，展示发现，与用户确认，然后写入。

## 流程

### 1. 探索

查看当前仓库以了解其初始状态。读取所有已存在的内容，不要臆测：

* `git remote -v` 和 `.git/config`：这是 GitHub 仓库吗？是哪一个？
* 仓库根目录下的 `AGENTS.md` 和 `CLAUDE.md`：两者是否存在？其中是否已经有 `## Agent skills` 部分？
* 仓库根目录下的 `CONTEXT.md` 和 `CONTEXT-MAP.md`
* `docs/adr/` 以及任何 `src/*/docs/adr/` 目录
* `docs/agents/`：此技能的先前输出是否已经存在？
* `.scratch/`：表明已经使用本地 Markdown issue 跟踪器约定的迹象
* 是否安装了 `triage` 技能？（与此技能并列的 `triage` 技能文件夹，或你的可用技能中的 `triage`。）这决定了 B 部分是否要运行。
* Monorepo 信号：`pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 字段，或一个包含自身 `src/` 的已填充 `packages/*`。这些仅存在于真正的大型多包仓库中；其缺失意味着单上下文，这几乎是每个仓库。

### 2. 展示发现并询问

概述现有的内容以及缺失的内容。然后按顺序处理各部分。一次一个部分，一个答案，接着下一个。

每个部分都以推荐答案开头，以便用户能用一句话接受。只有当选择确实有分支时才给出单行解释；当探索已经确定答案时，完全跳过该部分（未安装 `triage` 时跳过 B 部分，没有 monorepo 时跳过 C 部分）。

**部分 A：Issue tracker。**

> 解释器："Issue tracker" 是此仓库问题存放的位置。像 `to-tickets`、`triage` 和 `to-spec` 这样的技能会从中读写。它们需要知道是调用 `gh issue create`、在 `.scratch/` 下写入 markdown 文件，还是遵循你描述的其他工作流。选择你实际上为此仓库跟踪工作的位置。

默认姿态：这些技能是为 GitHub 设计的。如果 `git remote` 指向 GitHub，则推荐 GitHub。如果 `git remote` 指向 GitLab（`gitlab.com` 或自托管主机），则推荐 GitLab。否则（或用户偏好），提供：

* **GitHub**：问题存放在仓库的 GitHub Issues 中（使用 `gh` CLI）
* **GitLab**：问题存放在仓库的 GitLab Issues 中（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
* **Local markdown**：问题作为文件存放在此仓库的 `.scratch/<feature>/` 下（适合单人项目或没有远程仓库的仓库）
* **Other** (Jira, Linear 等)：请用户用一段话描述工作流；技能将把它记录为自由文本

将选择记录在 `docs/agents/issue-tracker.md` 中。GitHub 和 GitLab 模板携带一个 "PRs 作为请求面" 标志，默认为 **off**。保持关闭状态，不要将其开启：想要在分流队列中有外部 PR 的用户可以在文件中稍后翻转该标志。

部分 B：Triage label vocabulary。如果未安装 `triage` 技能（探索已告诉你），则完全跳过此部分，因为未安装的技能不需要标签。

如果已安装，只问一个问题：

> 你想保留默认的分流标签吗？（推荐：**是**）

默认值是五个标准角色，每个标签字符串等于其名称：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。在 **yes** 时，按原样写入它们。只有当用户说“不”时（通常是因为他们的跟踪器已经使用了其他名称，例如 `needs-triage` 对应 `bug:triage`），才收集覆盖项，以便 `triage` 应用现有标签而不是创建重复项。

部分 C：Domain docs。默认为 **single-context**（仓库根目录下有一个 `CONTEXT.md` + `docs/adr/`）。这几乎适合所有仓库；无需询问即可写入。

仅当探索发现 monorepo 信号时，才提供 **multi-context**（一个根 `CONTEXT-MAP.md` 指向每个上下文的 `CONTEXT.md` 文件）。然后确认他们想要哪种布局。

### 3. 确认并编辑

向用户展示草稿：

* 要添加到正在编辑的 `CLAUDE.md` / `AGENTS.md` 中的 `## Agent skills` 块（选择规则见第 4 步）
* `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md` 的内容（最后一项仅在安装了 `triage` 时）

让他们在写入前进行编辑。

### 4. 写入

**选择要编辑的文件：**

* 如果 `CLAUDE.md` 存在，则编辑它。
* 否则如果 `AGENTS.md` 存在，则编辑它。
* 如果都不存在，请用户选择创建哪一个；不要替他们选择。

当 `CLAUDE.md` 已经存在时（反之亦然），永远不要创建 `AGENTS.md`；始终编辑已存在的那个。

如果所选文件中已有 `## Agent skills` 块，则就地更新其内容，而不是附加重复块。不要覆盖用户对周围部分的编辑。

该块：

```markdown
## Agent skills

### Issue tracker

[one-line summary of where issues are tracked]. See `docs/agents/issue-tracker.md`.

### Triage labels

[one-line summary of the label vocabulary]. See `docs/agents/triage-labels.md`.

### Domain docs

[one-line summary of layout: "single-context" or "multi-context"]. See `docs/agents/domain.md`.
```

仅在 `triage` 已安装且 B 部分运行时，才包含 `### Triage labels` 子块并写入 `docs/agents/triage-labels.md`。否则两者都省略。

然后使用此技能文件夹中的种子模板作为起点写入文档文件：

* [issue-tracker-github.md](./issue-tracker-github.md): GitHub issue tracker
* [issue-tracker-gitlab.md](./issue-tracker-gitlab.md): GitLab issue tracker
* [issue-tracker-local.md](./issue-tracker-local.md): 本地 Markdown issue 跟踪器
* [triage-labels.md](./triage-labels.md): 标签映射（仅当安装了 `triage` 时）
* [domain.md](./domain.md): 领域文档消费规则 + 布局

对于“其他”issue 跟踪器，请根据用户的描述从头编写 `docs/agents/issue-tracker.md`。

### 5. 完成

告诉用户设置已完成，哪些工程技能现在将读取这些文件。提到他们稍后可以直接编辑 `docs/agents/*.md`；只有当他们想切换 issue 跟踪器或从头开始时，才需要重新运行此技能。
