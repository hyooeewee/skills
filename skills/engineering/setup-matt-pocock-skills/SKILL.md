---
name: setup-matt-pocock-skills
description: 配置此仓库以供工程技能使用 — 设置其 issue 跟踪器、分流标签词汇表和领域文档布局。在其他工程技能首次使用前运行一次。
disable-model-invocation: true

---

# 设置 Matt Pocock 的技能

搭建工程技能所依赖的每仓库配置：

* **Issue 跟踪器** — issue 所在位置（默认 GitHub；开箱即用地支持本地 Markdown）
* **分流标签** — 用于五种标准分流角色的标签字符串
* **领域文档** — `CONTEXT.md` 与 ADR 的存放位置，以及读取它们的使用方规则

这是一个提示驱动的技能，不是确定性脚本。先探索，展示发现，与用户确认，然后写入。

## 流程

### 1. 探索

查看当前仓库以了解其初始状态。读取所有已存在的内容，不要臆测：

* `git remote -v` 和 `.git/config` — 这是 GitHub 仓库吗？是哪一个？
* 仓库根目录下的 `AGENTS.md` 和 `CLAUDE.md` — 两者是否存在？其中是否已有 `## Agent skills` 部分？
* 仓库根目录下的 `CONTEXT.md` 和 `CONTEXT-MAP.md`
* `docs/adr/` 以及任何 `src/*/docs/adr/` 目录
* `docs/agents/` — 此技能之前的输出是否已经存在？
* `.scratch/` — 表示本地 Markdown issue 跟踪器约定已在使用的迹象
* 是否安装了 `triage` 技能？（与此技能并列的 `triage` 技能文件夹，或你的可用技能中的 `triage`。）这决定了 B 部分是否要运行。
* Monorepo 信号 — `pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 字段，或带有自身 `src/` 的已填充 `packages/*`。只有真正的大型多包仓库才会出现这些；它们的缺失意味着单一上下文，而几乎所有仓库都是如此。

### 2. 展示发现并询问

总结哪些已存在、哪些缺失。然后按顺序逐一处理各部分 — 一个部分一个问题，得到答案后再进入下一部分。

每个部分都以推荐答案开头，以便用户能用一句话接受。只有当选择确实有分支时才给出单行解释；当探索已经确定答案时，完全跳过该部分（未安装 `triage` 时跳过 B 部分，没有 monorepo 时跳过 C 部分）。

**A 部分 — Issue 跟踪器。**

> 解释：“issue 跟踪器”是此仓库 issue 的存放位置。诸如 `to-tickets`、`triage` 和 `to-spec` 之类的技能会读取并写入它 — 它们需要知道是调用 `gh issue create`、在 `.scratch/` 下写入 Markdown 文件，还是遵循你描述的其他工作流。选择你实际操作中跟踪此仓库工作的地方。

默认姿态：这些技能是为 GitHub 设计的。如果 `git remote` 指向 GitHub，则推荐 GitHub。如果 `git remote` 指向 GitLab（`gitlab.com` 或自托管主机），则推荐 GitLab。否则（或用户偏好），提供：

* **GitHub** — issue 存在于仓库的 GitHub Issues 中（使用 `gh` CLI）
* **GitLab** — issue 存在于仓库的 GitLab Issues 中（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
* **本地 Markdown** — issue 作为此仓库 `.scratch/<feature>/` 下的文件存在（适合个人项目或无远程仓库的仓库）
* **其他**（Jira、Linear 等）— 请用户用一段话描述工作流；技能会将其记录为自由文本

将选择记录在 `docs/agents/issue-tracker.md` 中。GitHub 和 GitLab 模板带有“将 PR 作为请求表面”标志，默认**关闭** — 保持关闭且不要提及；希望外部 PR 进入分流队列的用户可以稍后在文件中切换该标志。

B 部分 — 分流标签词汇表。如果未安装 `triage` 技能（探索已告知），则完全跳过此部分 — 未安装的技能不需要标签。

如果已安装，只问一个问题：

> 你想保留默认的分流标签吗？（推荐：**是**）

默认值是五种标准角色，每个标签字符串与其名称相同：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。如果回答**是**，则按原样写入。只有当用户说否 — 通常是因为其跟踪器已经使用了其他名称（例如 `bug:triage` 代替 `needs-triage`）— 才收集覆盖项，以便 `triage` 应用现有标签而不是创建重复标签。

C 部分 — 领域文档。默认为**单一上下文** — 仓库根目录下一个 `CONTEXT.md` + `docs/adr/`。这几乎适合所有仓库；直接写入，无需询问。

仅当探索发现 monorepo 信号时，才提供**多上下文** — 根目录 `CONTEXT-MAP.md` 指向各上下文的 `CONTEXT.md` 文件。然后确认他们想要哪种布局。

### 3. 确认并编辑

向用户展示草稿：

* 要添加到正在编辑的 `CLAUDE.md` / `AGENTS.md` 中的 `## Agent skills` 块（选择规则见第 4 步）
* `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md` 的内容（最后一项仅在安装了 `triage` 时）

让他们在写入前进行编辑。

### 4. 写入

**选择要编辑的文件：**

* 如果 `CLAUDE.md` 存在，则编辑它。
* 否则如果 `AGENTS.md` 存在，则编辑它。
* 如果两者都不存在，询问用户要创建哪一个 — 不要替他们选择。

当 `CLAUDE.md` 已存在时，绝不创建 `AGENTS.md`（反之亦然）— 始终编辑已存在的那个文件。

如果所选文件中已有 `## Agent skills` 块，则就地更新其内容，而不是附加重复块。不要覆盖用户对周围部分的编辑。

该块：

```markdown
## Agent skills

### Issue tracker

[one-line summary of where issues are tracked]. See `docs/agents/issue-tracker.md`.

### Triage labels

[one-line summary of the label vocabulary]. See `docs/agents/triage-labels.md`.

### Domain docs

[one-line summary of layout — "single-context" or "multi-context"]. See `docs/agents/domain.md`.
```

仅在 `triage` 已安装且 B 部分运行时，才包含 `### Triage labels` 子块并写入 `docs/agents/triage-labels.md`。否则两者都省略。

然后使用此技能文件夹中的种子模板作为起点写入文档文件：

* [issue-tracker-github.md](./issue-tracker-github.md) — GitHub issue 跟踪器
* [issue-tracker-gitlab.md](./issue-tracker-gitlab.md) — GitLab issue 跟踪器
* [issue-tracker-local.md](./issue-tracker-local.md) — 本地 Markdown issue 跟踪器
* [triage-labels.md](./triage-labels.md) — 标签映射（仅在安装了 `triage` 时）
* [domain.md](./domain.md) — 领域文档使用方规则 + 布局

对于“其他”issue 跟踪器，请根据用户的描述从头编写 `docs/agents/issue-tracker.md`。

### 5. 完成

告诉用户设置已完成，以及哪些工程技能现在会读取这些文件。提及他们以后可以直接编辑 `docs/agents/*.md` — 只有在他们想要更换 issue 跟踪器或从头重新开始时，才需要重新运行此技能。
