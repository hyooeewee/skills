# Issue 追踪器：GitLab

本仓库的 Issues 和规范以 GitLab Issues 的形式存在。请使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI 进行所有操作。

## 约定

* **创建 Issue**：`glab issue create --title "..." --description "..."`。对于多行描述，请使用 heredoc。传递 `--description -` 以打开编辑器。
* **阅读 Issue**：`glab issue view <number> --comments`。使用 `-F json` 获取机器可读的输出。
* **列出 Issues**：`glab issue list -F json` 并配合适当的 `--label` 过滤器。
* **评论 Issue**：`glab issue note <number> --message "..."`。GitLab 将评论称为 "notes"（注释）。
* **应用 / 移除标签**：`glab issue update <number> --label "..."` / `--unlabel "..."`。多个标签可以用逗号分隔或重复使用该标志。
* **关闭**：`glab issue close <number>`。`glab issue close` 不接受关闭评论，因此请先使用 `glab issue note <number> --message "..."` 发布说明，然后再关闭。
* **合并请求**：GitLab 将 PR 称为 "merge requests"（合并请求）。使用 `glab mr create`、`glab mr view`、`glab mr note` 等 —— 形状与 `gh pr ...` 相同，用 `mr` 替换 `pr`，用 `note`/`--message` 替换 `comment`/`--body`。

从 `git remote -v` 推断仓库 —— `glab` 在克隆内部运行时会自动执行此操作。

## 合并请求作为分类表面

**MR 作为请求表面：否。** *(如果此仓库将外部合并请求视为功能请求，请设置为 `yes`；`/triage` 会读取此标志。)*

设置为 `yes` 时，MR 将像 Issues 一样经过相同的标签和状态，使用 `glab mr` 等效命令：

* **阅读 MR**：`glab mr view <number> --comments` 和 `glab mr diff <number>` 查看差异。
* **列出用于分类的外部 MR**：`glab mr list -F json`，然后仅保留作者不是项目成员/所有者的 MR（贡献者的 MR，而非维护者的进行中的工作）。
* **评论 / 标签 / 关闭**：`glab mr note`、`glab mr update --label`/`--unlabel`、`glab mr close`。

与 GitHub 不同，GitLab 分别编号 Issues 和 MR，因此一旦你知道维护者指的是哪个表面，`#42` 就不会产生歧义。

## 当技能说 "发布到问题追踪器" 时

创建一个 GitLab issue。

## 当技能说 "获取相关票据" 时

运行 `glab issue view <number> --comments`。

## 导航操作

被 `/wayfinder` 使用。**地图** 是一个带有 **子** Issues 作为票据的单个 Issue。

* **地图**：一个标记为 `wayfinder:map` 的单个 Issue，包含 Notes / Decisions-so-far / Fog 内容。`glab issue create --label wayfinder:map`。（在具有原生 Epic 的 GitLab 层级中，Epic 可能会持有地图；标记的 Issue 适用于所有地方。）
* **子票据**：一个在描述顶部携带 `Part of #<map>` 并带有 `wayfinder:<type>` 标签的 Issue（`research`/`prototype`/`grilling`/`task`）。一旦认领，该票据将被分配给主开发人员。
* **阻塞**：GitLab 的 **原生阻塞链接** —— 标准的、UI 可见的表示形式。使用 `/blocked_by #<n>` 快捷操作添加它，作为注释发布（`glab issue note <child> --message "/blocked_by #<blocker>"`）。原生阻塞链接是 Premium/Ultimate 功能；在免费层级（或不可用的情况下）回退到描述顶部的 `Blocked by: #<n>, #<n>` 行。当每个阻塞者都关闭时，票据即被解除阻塞。
* **前沿查询**：`glab issue list -F json` 限定在地图的子项中，排除任何有开放阻塞者的情况 —— 即对开放 Issue 的原生 `blocked_by` 链接（`glab api projects/:id/issues/:iid/links`），或 `Blocked by` 行中的开放 Issue —— 或指派给某人；按地图顺序优先者胜出。
* **认领**：`glab issue update <n> --assignee @me` —— 会话中的第一次写入。
* **解决**：`glab issue note <n> --message "<answer>"`，然后 `glab issue close <n>`，然后将上下文指针（gist + 链接）附加到地图的 Decisions-so-far。
