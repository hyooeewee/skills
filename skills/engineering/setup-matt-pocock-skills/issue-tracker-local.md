# Issue tracker: 本地 Markdown

此仓库的 Issues 和规范（你可能知道规范就是 PRD）以 Markdown 文件的形式存放在 `.scratch/` 中。

## 约定

* 每个功能一个目录：`.scratch/<feature-slug>/`
* 规范位于 `.scratch/<feature-slug>/spec.md`
* 实现问题按票据对应一个文件，位于 `.scratch/<feature-slug>/issues/<NN>-<slug>.md`，编号从 `01` 开始——绝不使用单个合并的票据文件
* 分类状态记录在每个 issue 文件顶部附近的 `Status:` 行中（参见 `triage-labels.md` 了解角色字符串）
* 评论和对话历史追加到文件底部，位于 `## Comments` 标题下

## 当技能说 "发布到问题追踪器" 时

在 `.scratch/<feature-slug>/` 下创建一个新文件（如果需要，先创建目录）。

## 当技能说 "获取相关票据" 时

读取指定路径下的文件。用户通常会直接传递路径或票据编号。

## 导航操作

由 `/wayfinder` 使用。**地图** 是一个包含每个票据一个**子文件**的文件。

* **地图**: `.scratch/<effort>/map.md` — 笔记 / 至今的决策 / Fog 正文。
* **子票据**: `.scratch/<effort>/issues/NN-<slug>.md`，编号从 `01` 开始，主体包含问题。`Type:` 行记录票据类型 (`research`/`prototype`/`grilling`/`task`)；`Status:` 行记录 `claimed`/`resolved`。
* **阻塞**: 顶部附近的一个 `Blocked by: NN, NN` 行。当列出的每个文件都是 `resolved` 时，票据解除阻塞。
* **前沿**: 扫描 `.scratch/<effort>/issues/` 以查找处于开放、未阻塞且未认领状态的文件；编号优先。
* **认领**: 在开始任何工作之前设置 `Status: claimed` 并保存。
* **解决**: 在 `## Answer` 标题下附加答案，设置 `Status: resolved`，然后将上下文指针（gist + 链接）追加到 `map.md` 中地图的“至今的决策”部分。
