# 将技能集作为原生 Claude Code 插件交付；原生 Codex 插件暂缓

这些技能一直都可以通过 [skills.sh](https://skills.sh/mattpocock/skills)（`npx skills add mattpocock/skills`）安装，它会将可编辑的技能文件复制到用户的项目中，适用于 Claude Code、Codex 以及其他遵循 Agent Skills 标准的工具。用户反复提出的需求是**即插即用**的交付方式：将整套技能作为一个只读、始终最新、无需自行编辑的捆绑包订阅，而不是维护一个自己拥有的 fork。这正是原生插件系统所提供的。

我们将交付原生 **Claude Code 插件**，同时暂时**暂缓**原生 **Codex 插件**。这种区分是由各生态系统的插件清单选择技能的方式，与本仓库的分桶布局共同决定的。

## 约束：分桶技能与单路径选择

技能存放在 `skills/` 下的分桶文件夹中——`engineering/` 和 `productivity/` 是**已发布**（随产品发布）的；`misc/`、`personal/`、`in-progress/` 和 `deprecated/` 则**不是**。插件必须只暴露已发布的技能集，而该技能集横跨其中两个分桶文件夹。

* **Claude Code** —— `.claude-plugin/plugin.json` 将 `skills` 接受为**显式技能目录路径的数组**。我们逐一列出已发布的技能，明确排除其他所有内容，并添加 `.claude-plugin/marketplace.json`，使该仓库成为自己的单插件市场。已做端到端验证：`claude plugin validate . --strict` 通过，`marketplace add` → `install` 可解析所有已发布的技能。

* **Codex** —— `.codex-plugin/plugin.json` 只接受 `skills` 作为**单个路径字符串**（数组会被拒绝并提示 `missing or invalid plugin.json`），Codex 会递归发现该路径下的 `SKILL.md` 文件。无法通过一个路径指定两个分桶文件夹，或精选一个子集。测试并否决了两种变通方案：
  * 将路径指向 `./skills/` 也会一并发布 `deprecated/`、`in-progress/`、`personal/` 和 `misc/`——这些是我们刻意不发布的已弃用、草稿和个人技能。
  * 一个由指向各分桶的**符号链接**组成的精选扁平目录在安装后无法保留：Codex 会将插件树复制到其缓存中并**丢弃符号链接**，因此技能内容为空。

要为 Codex 提供单一且仅含已发布技能的路径，唯一的稳健方式是：(a) **重构**，让 `skills/` 只包含已发布的技能（将非发布分桶移出——这会波及 `CLAUDE.md`、`scripts/link-skills.sh`、各分桶的 README，以及依赖 `in-progress/` 和 `personal/` 的本地开发工作流），或者 (b) **提交重复副本**，将已发布技能复制到一个扁平目录中（这会造成同步负担，并形成第二个事实来源）。这两者都属于结构性决策，不应作为 Claude 插件发布的一部分捆绑处理。这很可能就是当初插件未能更早发布的那个被半遗忘的原因：清单格式无法干净地表达分桶仓库中的精选子集。

## 决策

* 现在发布 **Claude Code 插件**（`.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json`），精选到已发布的技能集，作为 v1.2 的主要交付物。
* 继续将 **skills.sh** 作为通用安装器——它目前已经服务于 Codex 和其他工具，因此不会有 Codex 用户缺少安装途径。
* **暂缓**原生 Codex 插件，直到我们决定是将 `skills/` 重构为仅含已发布技能，还是提交一份生成的扁平副本。当 Codex 支持 `skills` 数组/包含列表，或在安装时保留符号链接时，再重新评估。

## 由此产生的不变项

* 每个已发布的技能都必须在 `.claude-plugin/plugin.json` 的 `skills` 数组中有一条记录（这原本已是 `CLAUDE.md` 中的规则；现在它同样决定插件的内容）。
* `.claude-plugin/plugin.json` 的 `version` 与 `package.json` 的版本保持一致——发布时两者需同步升级。Claude 使用插件的 `version` 来决定已安装用户何时看到更新。

## 更新于 2026-08-05

`mattpocock-skills` 已被收录进 **Claude Code 官方市场**——配置名为 `claude-plugins-official`，源仓库为 `anthropics/claude-plugins-official`——这是每个 Claude Code 安装默认自带的市场。现在，`claude plugins install mattpocock-skills` 是文档记录的安装方式，上述 `marketplace add` → `install` 路径已被取代。安装说明见 [.agents/install-block.md](../install-block.md)。

官方列表指向本仓库的 git URL，并直接读取 `.claude-plugin/plugin.json`，因此它不依赖 `.claude-plugin/marketplace.json`。该文件仅保留作为直接安装仓库（未发布的提交或 fork）时的后备方案。

已于 2026-08-05 在 Claude Code 2.1.222 上对照实时列表验证：

* `claude plugins install mattpocock-skills` 无需先添加市场即可解析，并显示 `mattpocock-skills@claude-plugins-official`。
* 随后 `claude plugin details mattpocock-skills` 显示版本 1.2.0，并加载已发布的技能。
* 列表的 `source` 为 `{"source": "url", "url": "https://github.com/mattpocock/skills.git", "sha": …}` —— **sha 被固定**，因此发布会在该固定值移动时到达已安装用户，而不是在我们打标签时。撰写本文时，该固定值落后 `main` 两个提交，这正是它列出 22 个技能而非 `plugin.json` 中 24 个技能的原因。
* 会话内的 `/plugin install mattpocock-skills` **未**实际执行——在无头（`claude -p`）会话中无法使用 `/plugin`。它运行与 CLI 相同的解析器，文档中的示例形式为 `/plugin install <name>@claude-plugins-official`。
