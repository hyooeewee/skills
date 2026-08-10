# Issue 追踪器: GitHub

此仓库的 Issues 和规范以 GitHub Issues 的形式存在。使用 `gh` CLI 执行所有操作。

## 约定

* **创建 Issue**：`gh issue create --title "..." --body "..."`。对于多行内容，使用 heredoc。
* **查看 Issue**：`gh issue view <number> --comments`，使用 `jq` 过滤评论并获取标签。
* **列出 Issues**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，配合适当的 `--label` 和 `--state` 过滤器。
* **评论 Issue**：`gh issue comment <number> --body "..."`
* **应用 / 移除标签**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
* **关闭**：`gh issue close <number> --comment "..."`

从 `git remote -v` 推断仓库 — `gh` 在克隆仓库内运行时会自动完成此操作。

## Pull requests 作为分类界面

**PRs 作为请求表面：否。** *(如果此仓库将外部 PR 视为功能请求，则设为 `yes`；`/triage` 会读取此标志。)*

当设置为 `yes` 时，PR 运行与 Issues 相同的标签和状态，使用 `gh pr` 等价命令：

* **查看 PR**：`gh pr view <number> --comments`，使用 `gh pr diff <number>` 查看差异。
* **列出用于分类的外部 PR**：`gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments`，然后仅保留 `CONTRIBUTOR`、`FIRST_TIME_CONTRIBUTOR` 或 `NONE` 的 `authorAssociation`（舍弃 `OWNER`/`MEMBER`/`COLLABORATOR`）。
* **评论 / 标签 / 关闭**：`gh pr comment`，`gh pr edit --add-label`/`--remove-label`，`gh pr close`。

GitHub 在 Issues 和 PRs 之间共享一个数字空间，因此裸 `#42` 可能是两者之一 — 使用 `gh pr view 42` 解决，如果不行则回退到 `gh issue view 42`。

## 当技能说 "发布到问题追踪器" 时

创建一个 GitHub Issue。

## 当技能说 "获取相关票据" 时

运行 `gh issue view <number> --comments`。

## 导航操作

被 `/wayfinder` 使用。**地图** 是一个带有 **子** Issues 作为票据的单个 Issue。

* **地图**：一个标记为 `wayfinder:map` 的单个 Issue，包含 Notes / Decisions-so-far / Fog 正文。`gh issue create --label wayfinder:map`。
* **子票据**：作为 GitHub 子 Issue 链接到地图的 Issue（在子 Issues 端点使用 `gh api`）。如果未启用子 Issues，将子 Issue 添加到地图正文中的任务列表，并在子 Issue 正文顶部放置 `Part of #<map>`。标签：`wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`)。一旦认领，该票据将分配给主导开发人员。
* **阻塞**：GitHub 的 **原生 Issue 依赖** — 规范的、UI 可见的表示。使用 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` 添加依赖，其中 `<blocker-db-id>` 是阻塞者的数字 **数据库 ID**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`，*不是* `#number` 或 `node_id`）。GitHub 报告 `issue_dependencies_summary.blocked_by`（仅限开放阻塞 — 实时门控）。如果依赖不可用，则回退到子 Issue 正文顶部的 `Blocked by: #<n>, #<n>` 行。当所有阻塞者都关闭时，票据解除阻塞。
* **前沿查询**：列出地图的开放子 Issues（`gh issue list --state open`，范围限定在地图的子 Issues / 任务列表），丢弃任何有开放阻塞者（`issue_dependencies_summary.blocked_by > 0`，或 `Blocked by` 行中有开放 Issue）或分配者的；按地图顺序优先。
* **认领**：`gh issue edit <n> --add-assignee @me` — 会话中的首次写入。
* **解决**：`gh issue comment <n> --body "<answer>"`，然后 `gh issue close <n>`，最后将上下文指针（gist + 链接）追加到地图的 Decisions-so-far 中。
