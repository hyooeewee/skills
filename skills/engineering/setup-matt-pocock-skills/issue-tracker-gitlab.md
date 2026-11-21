# 问题跟踪器：GitLab

此仓库的问题和规格以 GitLab issue 的形式存在。所有操作均使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI。

## 约定

* **创建 issue**：`glab issue create --title "..." --description "..."`。对于多行描述，使用 heredoc。传递 `--description -` 以打开编辑器。
* **读取 issue**：`glab issue view <number> --comments`。使用 `-F json` 获取机器可读的输出。
* **列出 issues**：使用适当的 `--label` 过滤器执行 `glab issue list -O json`。
* **评论 issue**：`glab issue note <number> --message "..."`。GitLab 将评论称为 "notes"。
* **应用/移除标签**：`glab issue update <number> --label "..."` / `--unlabel "..."`。多个标签可以用逗号分隔，或重复该标志。
* **关闭**：`glab issue close <number>`。`glab issue close` 不接受关闭评论，因此先用 `glab issue note <number> --message "..."` 发布说明，然后关闭。
* "Merge requests"：GitLab 将 PR 称为 "merge requests"。使用 `glab mr create`、`glab mr view`、`glab mr note` 等，其结构类似于 `gh pr ...`，将 `mr` 替换 `pr`，将 `note`/`--message` 替换 `comment`/`--body`。

从 `git remote -v` 推断出仓库；`glab` 在克隆内部运行时会自动执行此操作。

## 合并请求作为分流入口

**MR 作为请求入口：否。** *（如果此仓库将外部合并请求视为功能请求，则设置为 `yes`；`/triage` 会读取此标志。）*

当设置为 `yes` 时，MR 使用与 issues 相同的标签和状态，使用对应的 `glab mr` 命令：

* **读取 MR**：`glab mr view <number> --comments` 以及用于查看 diff 的 `glab mr diff <number>`。
* **列出用于分流的外部 MR**：执行 `glab mr list -O json`，然后仅保留作者不是项目成员/所有者的 MR（即贡献者的 MR，而非维护者正在进行的工作）。
* **评论 / 添加标签 / 关闭**：`glab mr note`、`glab mr update --label`/`--unlabel`、`glab mr close`。

与 GitHub 不同，GitLab 对 issues 和 MR 分别编号，因此一旦你知道维护者指的是哪个层面，`#42` 就没有歧义。

## 当某个技能说“发布到 issue tracker”时

创建 GitLab issue。

## 当某个技能说“获取相关工单”时

运行 `glab issue view <number> --comments`。

## 寻路操作

由 `/wayfinder` 使用。**地图**是一个单一的 issue，**子** issues 作为工单。

* **地图**：一个标记为 `wayfinder:map` 的单一 issue，包含 Notes / Decisions-so-far / Fog 正文。`glab issue create --label wayfinder:map`。（在具有原生 epic 的 GitLab 层级上，epic 可以代替保存地图；但在任何地方，带标签的 issue 都能工作。）
* **子工单**：一个 issue，其描述顶部带有 `Part of #<map>`，链接到地图（`glab issue create --linked-issues <map> --link-type relates_to`），并带有标签 `wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。一旦被认领，该工单将分配给驱动开发者。
* "Blocking"：GitLab 的**原生阻塞链接**，这是规范、UI 可见的表示形式。使用 `/blocked_by #<n>` 快速操作添加它，作为 note 发布（`glab issue note <child> --message "/blocked_by #<blocker>"`）。原生阻塞链接是 Premium/Ultimate 功能；在免费层（或不可用的地方）回退到描述顶部的 `Blocked by: #<n>, #<n>` 行。当每个阻塞者都关闭时，工单即被解除阻塞。
* **前沿查询**：使用 `glab api projects/:id/issues/<map>/links` 列出地图的子项，保留带有 `wayfinder:<type>` 标签的开放 issue；丢弃有开放阻塞者的项：指向开放 issue 的原生 `blocked_by` 链接（`glab api projects/:id/issues/<child-iid>/links`）、`Blocked by` 行中的开放 issue，或有分配者；按地图顺序第一个获胜。
* "Claim"：`glab issue update <n> --assignee @me`，即会话中的首次写入。
* **解决**：`glab issue note <n> --message "<answer>"`，然后 `glab issue close <n>`，接着在地图的 Decisions-so-far 中追加一个上下文指针（gist + link）。
