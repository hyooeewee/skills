技能按桶文件夹组织在 `skills/` 目录下：

* `engineering/` — 日常代码工作
* `productivity/` — 日常非代码工作流工具
* `misc/` — 保留但很少使用，不推广
* `in-progress/` — 测试版：有意公开，欢迎反馈，未随插件发布
* `deprecated/` — 已弃用，不再使用

`engineering/` 或 `productivity/`（**已推广**的桶）中的每个技能都必须在顶层 `README.md` 中有引用，并在 `.claude-plugin/plugin.json` 的 `skills` 数组中有一个条目（Claude Code 插件只打包已推广的技能集）。`misc/`、`in-progress/` 和 `deprecated/` 中的技能不得出现在这两处中的任何一处。

安装命令从 [.agents/install-block.md](./.agents/install-block.md) 原样复制。`.claude-plugin/marketplace.json` 使该仓库成为自己的单插件 marketplace——这是安装块所解释的备用方案，而非文档化的路线。修改任一清单文件后，运行 `claude plugin validate . --strict`。为什么是 Claude 插件而不是（尚未）Codex 插件，详见 [.agents/adr/0002-ship-as-a-claude-code-plugin.md](./.agents/adr/0002-ship-as-a-claude-code-plugin.md)。

顶层 `README.md` 中的每个技能条目都必须将技能名称链接到其 `SKILL.md`。

每个桶文件夹都有一个 `README.md`，列出该桶中的每个技能并附一行描述，技能名称链接到其 `SKILL.md`。已推广桶的 `README.md` 和顶层 `README.md` 将条目分组为**用户调用**和**模型调用**；未推广桶的 `README.md`（`misc/`、`in-progress/`）使用平铺列表。

`engineering/` 和 `productivity/` 中的技能还有面向人类的文档页面，位于 `docs/<bucket>/<skill-name>.md`（文档树镜像了 `skills/` 下的这两个桶文件夹）。无论桶如何，发布的 URL 都是 `https://aihero.dev/skills-<skill-name>`——文档路径仅用于仓库组织。当你添加、重命名或更改 `engineering/` 或 `productivity/` 中技能的行为时，请按照 [.agents/writing-docs.md](./.agents/writing-docs.md) 创建或重新同步其文档页面。完成的页面包含四个部分——**它做什么**、**何时使用**、**常见问题**、**如果有效则表现为**——`writing-docs.md` 中保存了模板、部分顺序以及在哪里寻找问题。未推广桶（`misc/`、`in-progress/`、`deprecated/`）中的技能**没有**文档页面。

每个 `SKILL.md` 要么是用户调用的（在 `agents/openai.yaml` 中设置 `disable-model-invocation: true` 和 `policy.allow_implicit_invocation: false`，仅可由人类访问），要么是模型调用的（模型或用户均可访问）。请参阅 [.agents/invocation.md](./.agents/invocation.md)。

[`ask-matt`](./skills/engineering/ask-matt/SKILL.md) 是映射每个用户可访问技能及其关系的路由器。重新同步文档页面的同一触发条件也适用于它：每当你添加、重命名、删除或更改用户可访问技能在工作流中的适配方式时，请重新阅读 `ask-matt` 的 `SKILL.md` 并更新它，以使映射保持准确——一个从未提及的新技能，或一个仍然路由到的过时技能，就是一个撒谎的路由器。

要将每个技能（重新）链接到本地工具的技能目录（`~/.claude/skills`、`~/.agents/skills`），请运行 `scripts/link-skills.sh`。每个条目都是指向此仓库的符号链接，因此 `git pull` 可使已安装的技能保持最新；添加、删除或重命名技能后，请重新运行该脚本。
