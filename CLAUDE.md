技能按桶文件夹组织在 `skills/` 目录下：

* `engineering/`: 日常编码工作
* `productivity/`: 日常非编码工作流工具
* `misc/`: 保留但很少使用，未推广
* `in-progress/`: 测试版：故意公开，寻求反馈，插件中未发布
* `deprecated/`: 不再使用

`engineering/` 或 `productivity/`（**已推广**的桶）中的每个技能都必须在顶层 `README.md` 中有引用，并在 `.claude-plugin/plugin.json` 的 `skills` 数组中有一个条目（Claude Code 插件只打包已推广的技能集）。`misc/`、`in-progress/` 和 `deprecated/` 中的技能不得出现在这两处中的任何一处。

安装命令直接从 [.agents/install-block.md](./.agents/install-block.md) 复制而来。`.claude-plugin/marketplace.json` 将此仓库打造为一个单一插件市场（安装块中解释了这是一种后备方案，并非文档记录的路径）。在修改任一清单文件后，请运行 `claude plugin validate . --strict`。为什么使用 Claude 插件而不是（目前）Codex 插件，请参阅 [.agents/adr/0002-ship-as-a-claude-code-plugin.md](./.agents/adr/0002-ship-as-a-claude-code-plugin.md)。

顶层 `README.md` 中的每个技能条目都必须将技能名称链接到其 `SKILL.md`。

每个桶文件夹都有一个 `README.md`，列出该桶中的每个技能并附一行描述，技能名称链接到其 `SKILL.md`。已推广桶的 `README.md` 和顶层 `README.md` 将条目分组为**用户调用**和**模型调用**；未推广桶的 `README.md`（`misc/`、`in-progress/`）使用平铺列表。

`engineering/` 和 `productivity/` 中的技能在 `docs/<bucket>/<skill-name>.md` 处也有面向人类的文档页面（文档树镜像了 `skills/` 下这两个桶文件夹）。无论桶是什么，发布的 URL 都是 `https://aihero.dev/skills-<skill-name>`：文档路径仅用于仓库组织。当你在 `engineering/` 或 `productivity/` 中添加、重命名或更改技能的行为时，请按照 [.agents/writing-docs.md](./.agents/writing-docs.md) 创建或重新同步其文档页面。完成的页面包含四个部分：**功能**、**何时使用**、**常见问题**和 **如何判断生效**。`writing-docs.md` 包含模板、章节顺序以及问题来源位置。未推广桶（`misc/`、`in-progress/`、`deprecated/`）中的技能**没有**文档页面。

每个 `SKILL.md` 要么是用户调用的（在 `agents/openai.yaml` 中设置 `disable-model-invocation: true` 和 `policy.allow_implicit_invocation: false`，仅可由人类访问），要么是模型调用的（模型或用户均可访问）。请参阅 [.agents/invocation.md](./.agents/invocation.md)。

[`ask-matt`](./skills/engineering/ask-matt/SKILL.md) 是一个路由器，映射每个用户可访问的技能及其关系。重新同步文档页面的触发条件也适用于它：每当添加、重命名、删除或更改用户可访问技能的流程适配方式时，请重新阅读 `ask-matt` 的 `SKILL.md` 并进行更新，以确保映射保持准确：如果它从未提及某个新技能，或者仍将某个过时技能路由到那里，那它就是一个不准确的路由器。

要将 `deprecated/` 和 `misc/` 之外的每个技能（重新）链接到本地 harness 技能目录（`~/.claude/skills`、`~/.agents/skills`），请运行 `scripts/link-skills.sh`。每个条目都是指向此仓库的符号链接，因此 `git pull` 即可保持已安装技能为最新状态；添加、删除或重命名技能后请重新运行该脚本。

此仓库的任何叙述文本（`SKILL.md` 文件、文档、`README.md`、`CHANGELOG.md`、ADR、变更集、代码注释）中不得出现破折号。如果句子需要使用破折号，请改用逗号、冒号、句号、括号或连词来重写，以符合句子的实际需求；切勿盲目替换字符。
