技能被组织在 `skills/` 下的桶文件夹中：

* `engineering/` — 日常代码工作
* `productivity/` — 日常非代码工作流工具
* `misc/` — 保留但很少使用，未推广
* `personal/` — 与我自己的配置相关，未推广
* `in-progress/` — 尚未准备好发布的草稿
* `deprecated/` — 不再使用

`engineering/` 或 `productivity/` 中的每个技能（**promoted** 桶）必须在顶层 `README.md` 中有一个引用，并在 `.claude-plugin/plugin.json` 的 `skills` 数组中有一个条目（Claude Code 插件只发布推广的那一组技能）。`misc/`、`personal/`、`in-progress/` 和 `deprecated/` 中的技能不得出现在两者中。

该仓库也是其自己的单插件 Claude Code 市场：`.claude-plugin/marketplace.json` 列出了唯一的 `mattpocock-skills` 插件。在升级发布版本时，请保持 `.claude-plugin/plugin.json` 的 `version` 与 `package.json` 同步 — Claude 使用插件 `version` 来决定已安装的用户何时看到更新。在修改任一清单文件后，运行 `claude plugin validate . --strict`。为什么是 Claude 插件而不是（尚未）Codex 插件，请参阅 [.agents/adr/0002-ship-as-a-claude-code-plugin.md](./.agents/adr/0002-ship-as-a-claude-code-plugin.md)。

顶层 `README.md` 中的每个技能条目必须将技能名称链接到其 `SKILL.md`。

每个桶文件夹都有一个 `README.md`，列出了桶中的每个技能及其一行描述，并将技能名称链接到其 `SKILL.md`。推广桶的 `README.md` 和顶层 `README.md` 将条目分为 **User-invoked** 和 **Model-invoked**；非推广桶的 `README.md`（`misc/`、`personal/`）使用扁平列表。

`engineering/` 和 `productivity/` 中的技能在 `docs/<bucket>/<skill-name>.md` 处还有一个面向人类的文档页面（文档树在 `skills/` 下镜像了那两个桶文件夹）。发布的 URL 是 `https://aihero.dev/skills-<skill-name>`，无论属于哪个桶 — 文档路径仅用于仓库组织。当你在 `engineering/` 或 `productivity/` 中添加、重命名或更改技能的行为时，请按照 [.agents/writing-docs.md](./.agents/writing-docs.md) 创建或重新同步其文档页面。非推广桶（`misc/`、`personal/`、`in-progress/`、`deprecated/`）中的技能**没有**文档页面。

每个 `SKILL.md` 要么是用户调用的（`disable-model-invocation: true` 加上 `agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`，仅人类可访问），要么是模型调用的（模型或用户可访问）。请参阅 [.agents/invocation.md](./.agents/invocation.md)。

`ask-matt` 是映射每个用户可访问技能及其关系的路由器。重新同步文档页面的触发器也适用于它：每当添加、重命名、删除或更改用户可访问技能在流程中的适配方式时，请重新读取 `ask-matt` 的 `SKILL.md` 并更新它，以保持映射准确 — 如果它从未提到一个新技能，或者仍然路由到一个过时的技能，那就是一个撒谎的路由器。

要将每个技能（重新）链接到本地工具技能目录（`~/.claude/skills`、`~/.agents/skills`），请运行 `scripts/link-skills.sh`。每个条目都是指向此仓库的符号链接，因此 `git pull` 可以保持已安装技能的更新；在添加、删除或重命名技能后，请重新运行该脚本。
