# 问题跟踪器：本地 Markdown

本仓库的 issue 和规格说明以 markdown 文件形式存放在 `.scratch/` 中。

## 约定

* 每个功能一个目录：`.scratch/<feature-slug>/`
* 规格说明文件为 `.scratch/<feature-slug>/spec.md`
* 实现问题每个工单一个文件，位于 `.scratch/<feature-slug>/issues/<NN>-<slug>.md`，编号从 `01` 开始，绝不使用单个合并的工单文件
* Triage 状态以 `Status:` 行记录在每个 issue 文件顶部附近（角色字符串参见 `triage-labels.md`）。
* 评论和对话历史追加到文件末尾的 `## Comments` 标题下。

## 当某个技能说“发布到 issue tracker”时

在 `.scratch/<feature-slug>/` 下创建一个新文件（如有必要，创建该目录）。

## 当某个技能说“获取相关工单”时

读取所引用路径处的文件。用户通常会直接传入路径或 issue 编号。

## 寻路操作

由 `/wayfinder` 使用。**map** 是一个文件，每个工单对应一个 **child** 文件。

* **Map**: `.scratch/<effort>/map.md`（Notes / Decisions-so-far / 迷雾正文）
* **Child ticket**：`.scratch/<effort>/issues/NN-<slug>.md`，编号从 `01` 开始，问题写在正文中。`Type:` 行记录工单类型（`research`/`prototype`/`grilling`/`task`）；`Status:` 行记录 `claimed`/`resolved`。
* **Blocking**：顶部附近的 `Blocked by: NN, NN` 行。当某个工单列出的所有文件均为 `resolved` 时，该工单解除阻塞。
* **Frontier**：扫描 `.scratch/<effort>/issues/`，查找打开、未阻塞且未被认领的文件；编号最小者优先。
* **Claim**：在任何工作之前，将 `Status: claimed` 设置好并保存。
* **Resolve**：在 `## Answer` 标题下追加答案，将 `Status: resolved` 设置好，然后在 `map.md` 中向 map 的 Decisions-so-far 追加一个上下文指针（gist + link）。
