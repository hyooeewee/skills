# 将技能集作为原生 Claude Code 插件交付；原生 Codex 插件暂缓

这些技能一直都可以通过 [skills.sh](https://skills.sh/mattpocock/skills)（`npx skills add mattpocock/skills`）安装，它会将可编辑的技能文件复制到用户的项目中，适用于 Claude Code、Codex 以及其他遵循 Agent Skills 标准的工具。用户反复提出的需求是**即插即用**的交付方式：将整套技能作为一个只读、始终最新、无需自行编辑的捆绑包订阅，而不是维护一个自己拥有的 fork。这正是原生插件系统所提供的。

我们将交付原生 **Claude Code 插件**，同时暂时**暂缓**原生 **Codex 插件**。这种区分是由各生态系统的插件清单选择技能的方式，与本仓库的分桶布局共同决定的。

## 约束：分桶技能与单路径选择

技能位于 `skills/` 下的分桶文件夹中：`engineering/` 和 `productivity/` 是**已发布**（已交付）的；`misc/`、`personal/`、`in-progress/` 和 `deprecated/` 则**不是**。插件只能公开已发布的集合，该集合跨越了其中两个分桶文件夹。

* **Claude Code**: `.claude-plugin/plugin.json` 将 `skills` 接受为**显式技能目录路径的数组**。我们逐个列出已发布的技能，零歧义地排除其他所有内容，并添加 `.claude-plugin/marketplace.json`，使该仓库成为其自己的单插件市场。端到端验证：`claude plugin validate . --strict` 通过，`marketplace add` → `install` 解析所有已发布的技能。

* **Codex**: `.codex-plugin/plugin.json` 仅将 `skills` 接受为**单个路径字符串**（数组会被 `missing or invalid plugin.json` 拒绝），Codex 会递归发现其下的 `SKILL.md` 文件。无法从一个路径中指定两个分桶文件夹，也无法精选一个子集。测试并拒绝了两个变通方案：
  * 指向 `./skills/` 也会交付 `deprecated/`、`in-progress/`、`personal/` 和 `misc/`：这是我们故意不发布的已退役、草稿和个人技能。
  * 一个由指向各分桶的**符号链接**组成的精选扁平目录在安装后无法保留：Codex 会将插件树复制到其缓存中并**丢弃符号链接**，因此技能内容为空。

给 Codex 提供单个仅已发布路径的唯一可靠方法是 (a) **重构**，使 `skills/` 仅包含已发布的技能（将未发布的分桶移出，在 `CLAUDE.md`、`scripts/link-skills.sh`、分桶 README 以及依赖 `in-progress/` 和 `personal/` 的本地开发工作流中产生巨大影响），或者 (b) 将已发布技能的**重复副本**提交到扁平目录（这带来同步负担并成为第二个真相来源）。两者都是结构决策，而不是捆绑到交付 Claude 插件中的内容。这很可能就是插件之前未交付的原始、半记得的原因：清单格式无法干净地表达分桶仓库的精选子集。

## 决策

* 现在发布 **Claude Code 插件**（`.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json`），精选到已发布的技能集，作为 v1.2 的主要交付物。
* 将 **skills.sh** 保留为通用安装程序：它今天已经服务于 Codex 和其他工具，因此没有 Codex 用户会缺少安装路径。
* **暂缓**原生 Codex 插件，直到我们决定是将 `skills/` 重构为仅含已发布技能，还是提交一份生成的扁平副本。当 Codex 支持 `skills` 数组/包含列表，或在安装时保留符号链接时，再重新评估。

## 由此产生的不变项

* 每个已发布的技能都必须在 `.claude-plugin/plugin.json` 的 `skills` 数组中有一条记录（这原本已是 `CLAUDE.md` 中的规则；现在它同样决定插件的内容）。
* `.claude-plugin/plugin.json` 的 `version` 跟踪 `package.json` 的版本：发布时同时提升两者。Claude 使用插件 `version` 来决定安装的用户何时看到更新。

## 更新于 2026-08-05

`mattpocock-skills` 已被接受进入 **Claude Code 的官方市场**（配置名称 `claude-plugins-official`，源仓库 `anthropics/claude-plugins-official`），每个 Claude Code 安装默认都包含该市场。`claude plugins install mattpocock-skills` 现在是文档化的安装路径，上述 `marketplace add` → `install` 路径已被取代。安装说明位于 [.agents/install-block.md](../install-block.md)。

官方列表指向本仓库的 git URL，并直接读取 `.claude-plugin/plugin.json`，因此它不依赖 `.claude-plugin/marketplace.json`。该文件仅保留作为直接安装仓库（未发布的提交或 fork）时的后备方案。

已于 2026-08-05 在 Claude Code 2.1.222 上对照实时列表验证：

* `claude plugins install mattpocock-skills` 无需先添加市场即可解析，并显示 `mattpocock-skills@claude-plugins-official`。
* 随后 `claude plugin details mattpocock-skills` 显示版本 1.2.0，并加载已发布的技能。
* 列表的 `source` 是 `{"source": "url", "url": "https://github.com/mattpocock/skills.git", "sha": …}`：**sha 被固定**，因此发布会在该 pin 移动时触达已安装的用户，而不是我们打标签的那一刻。在撰写本文时，pin 位于 `main` 两个提交之后，这就是它列出 22 个技能而不是 `plugin.json` 中的 24 个的原因。
* 会话中的 `/plugin install mattpocock-skills` **未被执行**：`/plugin` 在无头（`claude -p`）会话中不可用。它运行与 CLI 相同的解析器，文档化的示例形式是 `/plugin install <name>@claude-plugins-official`。
