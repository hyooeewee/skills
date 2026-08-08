技能被组织在 `skills/` 下的桶文件夹中：

* `engineering/` — 日常代码工作
* `productivity/` — 日常非代码工作流工具
* `misc/` — 保留但很少使用，未推广
* `in-progress/` — beta：故意公开，寻求反馈，未包含在插件中
* `deprecated/` — 不再使用

`engineering/` 或 `productivity/` 中的每个技能（**推广**桶）必须在顶层 `README.md` 中有引用，并在 `.claude-plugin/plugin.json` 的 `skills` 数组中有一个条目（Claude Code 插件正好包含推广的那一组）。`misc/`、`in-progress/` 和 `deprecated/` 中的技能不得出现在这两个文件中。

安装命令直接从 [.agents/install-block.md](./.agents/install-block.md) 复制。`.claude-plugin/marketplace.json` 使该仓库成为其自己的单一插件市场 — 安装块解释了这是后备方案，而非文档化的路径。在修改任一清单文件后，运行 `claude plugin validate . --strict`。为什么是 Claude 插件而不是（尚未）Codex 插件，请参阅 [.agents/adr/0002-ship-as-a-claude-code-plugin.md](./.agents/adr/0002-ship-as-a-claude-code-plugin.md)。

顶层 `README.md` 中的每个技能条目必须将技能名称链接到其 `SKILL.md`。

每个桶文件夹都有一个 `README.md`，列出该桶中的每个技能及其一行描述，并将技能名称链接到其 `SKILL.md`。推广桶的 `README.md` 和顶层 `README.md` 将条目分为 **User-invoked** 和 **Model-invoked**；非推广桶的 `README.md`（`misc/`、`in-progress/`）使用扁平列表。

`engineering/` 和 `productivity/` 中的技能在 `docs/<bucket>/<skill-name>.md` 处也有面向人类的文档页面（文档树镜像了 `skills/` 下的那两个桶文件夹）。发布的 URL 是 `https://aihero.dev/skills-<skill-name>`，无论桶是什么 — 文档路径仅用于仓库组织。当你在 `engineering/` 或 `productivity/` 中添加、重命名或更改技能的行为时，请按照 [.agents/writing-docs.md](./.agents/writing-docs.md) 创建或重新同步其文档页面。完成的页面包含四个部分 — **What it does**（它做什么）、**When to reach for it**（何时使用它）、**Common questions**（常见问题）、**It's working if**（工作成功的条件是）— 而 `writing-docs.md` 包含模板、部分顺序以及问题的来源。非推广桶（`misc/`、`in-progress/`、`deprecated/`）中的技能**没有**文档页面。

每个 `SKILL.md` 要么是用户调用的（`disable-model-invocation: true` 加上 `agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`，仅人类可访问），要么是模型调用的（模型或用户可访问）。请参阅 [.agents/invocation.md](./.agents/invocation.md)。

`ask-matt` 是映射每个用户可访问技能及其关系的路由器。重新同步文档页面的触发器也适用于它：每当添加、重命名、删除或更改用户可访问技能在流程中的适配方式时，请重新读取 `ask-matt` 的 `SKILL.md` 并更新它，以保持映射准确 — 如果它从未提到一个新技能，或者仍然路由到一个过时的技能，那就是一个撒谎的路由器。

要将每个技能（重新）链接到本地工具技能目录（`~/.claude/skills`、`~/.agents/skills`），请运行 `scripts/link-skills.sh`。每个条目都是指向此仓库的符号链接，因此 `git pull` 可以保持已安装技能的更新；在添加、删除或重命名技能后，请重新运行该脚本。
