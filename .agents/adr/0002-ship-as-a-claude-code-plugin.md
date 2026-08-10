# 将技能集作为原生 Claude Code 插件发布；推迟原生 Codex 插件

这些技能一直可以通过 [skills.sh](https://skills.sh/mattpocock/skills) (`npx skills add mattpocock/skills`) 安装，它会将可编辑的技能文件复制到用户的项目中，覆盖 Claude Code、Codex 以及其他基于 Agent-Skills 标准的工具。一个持续的需求是**即插即用**的分发方式：将技能集作为只读、始终最新的包进行订阅，而不是拥有一个你自己的分支。这正是原生插件系统所提供的。

我们发布了一个原生的 **Claude Code 插件**，目前推迟了原生的 **Codex 插件**。这种分离是由每个生态系统的插件清单如何选择技能，以及本仓库的分桶布局所强制的。

## 约束：分桶技能与单路径选择的对比

技能存放在 `skills/` 下的桶文件夹中 —— `engineering/` 和 `productivity/` 是**精选**（已发布）；`misc/`、`personal/`、`in-progress/` 和 `deprecated/` 则**不是**。一个插件必须只展示精选的集合，这跨越了两个桶文件夹。

* **Claude Code** — `.claude-plugin/plugin.json` 接受 `skills` 作为 **显式技能目录路径的数组**。我们逐个列出精选技能，零歧义地排除其他所有内容，并添加 `.claude-plugin/marketplace.json`，使该仓库成为自己的单一插件市场。端到端验证：`claude plugin validate . --strict` 通过，且 `marketplace add` → `install` 解析了所有精选技能。

* **Codex** — `.codex-plugin/plugin.json` 只接受 `skills` 作为 **单个路径字符串**（数组会被 `missing or invalid plugin.json` 拒绝），Codex 会在其下递归发现 `SKILL.md` 文件。没有办法通过单个路径来命名两个桶文件夹，或从中精选一个子集。测试并拒绝了两个逃生舱口：
  * 指向 `./skills/` 也会发布 `deprecated/`、`in-progress/`、`personal/` 和 `misc/` —— 这些是我们故意不精选的已退役、草稿和私人技能。
  * 精选的指向桶的 **符号链接** 的扁平目录在安装后无法保留：Codex 将插件树复制到其缓存中并 **丢弃符号链接**，因此技能到达时是空的。

给 Codex 一个单一的仅精选路径的唯一稳健方法是 (a) **重组**，使 `skills/` 只包含精选技能（将未精选的桶移出 —— 这会在 `CLAUDE.md`、`scripts/link-skills.sh`、桶 README 以及依赖 `in-progress/` 和 `personal/` 的本地开发工作流上产生巨大的影响范围），或者 (b) **提交**精选技能的重复副本到一个扁平目录（同步负担和第二个事实来源）。两者都是结构性决策，而不是打包进 Claude 插件发布中的内容。这很可能是插件之前没有发布的原始、依稀记得的原因：清单格式无法清晰地表达桶式仓库的精选子集。

## 决策

* 现在发布 **Claude Code 插件** (`.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json`)，精选到精选集合，作为主要的 v1.2 交付物。
* 保持 **skills.sh** 作为通用安装程序 —— 它目前已经在服务 Codex 和其他工具，因此没有任何 Codex 用户会缺少安装路径。
* **推迟** 原生 Codex 插件，直到我们决定是将 `skills/` 重组为仅精选，还是提交生成的扁平副本。当 Codex 支持了 `skills` 数组 / 包含列表或在安装时保留符号链接时再进行审查。

## 这创建的不变量

* 每个精选技能都在 `.claude-plugin/plugin.json` 的 `skills` 数组中有一个条目（这已经是 `CLAUDE.md` 规则；它现在也控制了插件的内容）。
* `.claude-plugin/plugin.json` 的 `version` 跟踪 `package.json` 的版本 —— 在发布时一起增加两者。Claude 使用插件 `version` 来决定安装的用户何时看到更新。

## 更新，2026-08-05

\`\`mattpocock-skills`已被接受进入 **Claude Code 的官方市场** —— 配置名称为`claude-plugins-official`，源仓库为 `anthropics/claude-plugins-official` —— 这是每个 Claude Code 安装默认具备的。`claude plugins install mattpocock-skills`现在是记录在案的路径，上述`marketplace add`→`install\` 路径已被取代。安装说明位于 [.agents/install-block.md](../install-block.md)。

官方列表指向此仓库的 git URL 并直接读取 `.claude-plugin/plugin.json`，因此它不依赖 `.claude-plugin/marketplace.json`。该文件仅保留作为直接安装仓库的备选方案（未发布的提交或分叉）。

已验证 2026-08-05，在 Claude Code 2.1.222 上，针对实时列表：

* \`\`claude plugins install mattpocock-skills`在没有先添加市场的情况下解析，并报告`mattpocock-skills\@claude-plugins-official\`。
* \`\`claude plugin details mattpocock-skills\` 然后报告版本 1.2.0 并加载精选技能。
* 列表的 `source` 是 `{"source": "url", "url": "https://github.com/mattpocock/skills.git", "sha": …}` —— **sha 已固定**，因此当该固定点移动时，发布才会到达安装的用户，而不是我们打标签的那一刻。撰写本文时，该固定点位于 `main` 之后两个提交，这就是为什么它列出 22 个技能而不是 `plugin.json` 中的 24 个。
* 会话中的 `/plugin install mattpocock-skills` 未被测试 —— `/plugin` 在无头 (`claude -p`) 会话中不可用。它运行与 CLI 相同的解析器，记录的示例形式是 `/plugin install <name>@claude-plugins-official`。
