## 它做什么

`code-review` 会检查 `HEAD` 与你指定的固定点（一个提交、一个分支、一个标签、`main`、`HEAD~5`）之间的差异，并从两个维度进行审查。**Standards（标准）** 询问代码是否遵循了该仓库的编码方式。**Spec（规格）** 询问代码是否执行了原始 issue 或 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 要求的内容。每个维度都在其自己的 [子代理](https://www.aihero.dev/ai-coding-dictionary/subagent) 中运行，因此它们无法看到对方的推理过程。

这两个维度永远不会被合并，也永远不会重新排序。报告以 *每个维度* 的最严重问题作为结尾，并拒绝在它们之间选出一个唯一的赢家，因为一个变更可能在一个维度上通过而在另一个维度上失败：代码遵守了所有约定却实现了错误的东西，会在 Standards 上通过而在 Spec 上失败；代码完全按照 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 所要求的做了，却破坏了仓库的约定，情况则相反。混合的结论会让通过的维度掩盖失败的维度。

## 何时使用它

输入 `/code-review`，或者当你要求审查一个分支、一个 PR、进行中的工作，或任何“自 X 以来”的内容时，代理会自动使用它。

| 你的情况                                             | 选择                                                                                       |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| 存在一个 diff，你想知道它是否构建正确 *和&#x20;*&#x5E76;且做的是正确的事情 | `code-review`                                                                            |
| 你想在 diff 中查找 bug：空路径、竞态条件、边界错误                   | Claude Code 自带的审查，而不是这个（见下文的名字冲突）                                                        |
| 还没有写任何代码，你想以测试优先的方式编写它                           | [tdd](https://aihero.dev/skills-tdd)                                                     |
| 需要构建整个规格，包括审查                                    | [implement](https://aihero.dev/skills-implement)，它会自行调用本技能                               |
| 整个代码库已经漂移，而不是单个 diff                             | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) |
| 有东西坏了，而你不知道原因                                    | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)                             |

你必须提供固定点。如果你不提供，技能会要求你给出一个，而不是猜测；然后它会先检查 ref 能否解析以及 diff 是否非空，之后才会启动任何子代理，因此拼写错误的分支名会在你面前失败，而不是在两个子代理内部失败。

## 先决条件

Standards 维度不需要任何东西。它会读取仓库中记录的内容（`CODING_STANDARDS.md`、`CONTRIBUTING.md` 等），当仓库没有任何文档时，它会回退到内置的基线。

Spec 维度需要一个存在且可被找到的规格。它按以下顺序查找：

1. 提交信息中的 issue 引用（`#123`、`Closes #45`、GitLab 的 `!67`），通过 `docs/agents/issue-tracker.md` 获取。
2. 你作为参数传入的一个路径。
3. 位于 `docs/`、`specs/` 或 `.scratch/` 下、与分支或功能名称匹配的规格文件。
4. 询问你。

第 1 步依赖于 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 写入的 `docs/agents/issue-tracker.md`。没有它时，如果你传入路径，该维度仍然可以工作。如果完全没有规格，Spec 子代理会被跳过，报告会显示“没有可用规格”，而不是凭空编造需求。

## 两个维度

|         | Standards（标准）                  | Spec（规格）                |
| ------- | ------------------------------ | ----------------------- |
| 问题      | 它构建得对吗？                        | 它是正确的事情吗？               |
| 读取      | 仓库记录的规范，加上坏味道基线                | 原始的 issue 或规格           |
| 报告      | 记录在案的违规（可能是硬性的），以及坏味道（始终是主观判断） | 缺失或部分实现的需求、范围蔓延、错误实现的需求 |
| 每个发现都引用 | 规范文件和规则，或具名的坏味道外加代码块（hunk）     | 规格中的那一行                 |

一个不知道你标准的通用审查技能正是本设计试图避免的东西：它会标记出代码库中有意为之的部分，却漏掉了代码库实际依赖的不变量。因此，仓库自己的文档是 Standards 维度上的[主要来源](https://www.aihero.dev/ai-coding-dictionary/primary-source)，并且**仓库总是会覆盖**。

**坏味道基线** 是其下限，即《重构》第 3 章中的十二种 Fowler 代码坏味道：神秘的名称、重复代码、特征依恋、数据聚类、基本类型偏执、重复的 switch 语句、 Shotgun 手术（大泥球）、发散式变化、 speculative generality（推测一般性）、消息链、中间人、被拒绝的继承。每一个都是带标签的启发式规则（如“可能的特征依恋”），绝不是硬性的违规，并且每个都被表述为 *它是什么* → *如何修复*，因此发现时会附带一个修正方案，而不是抱怨。你的 linter 已经强制执行的内容会被两个维度都跳过。

## 常见问题

**它与 Claude Code 自带的 `/code-review`冲突。我该怎么办？**

这是该技能最常被报告的问题，而且尚未修复。Claude Code 内置了自己的 `/code-review`，它做的是不同的事情：它在 diff 中查找 bug，而这个技能检查的是规范合规性和仓库标准。安装这个库意味着其中之一会胜出，而谁胜出取决于你是如何安装的。通过插件市场，所有内容都在 `mattpocock-skills:` 前缀下有别名，内置技能在未限定名称时很难访问；通过普通技能安装，本地文件会胜出，而这个技能会覆盖内置技能。一个干净的解决方案是彻底移除 Claude Code 的内置技能：这能大幅节省 [上下文](https://www.aihero.dev/ai-coding-dictionary/context)，而且冲突就不再重要了。这种覆盖行为可以说是一个 Claude Code [Harness](https://www.aihero.dev/ai-coding-dictionary/harness) 的 bug（技能作者应该可以自由地给技能命名任何东西），所以另一个解决方案是重命名本地副本。编辑 frontmatter 或重命名目录会被 `npx skills update` 撤销；用户报告的持久化变通方法是 fork 这个技能到一个新名字，并从管理集合中移除 `code-review`，同时记录你 fork 的那个 commit，以便你可以手动重新同步。

**它的子代理不断调用 `/code-review`，并再次生成更多代理。**

已知存在的开放 bug，被多人在多个 Harness 中复现。Standards 和 Spec 提示词没有禁止委托，所以子代理可以重新发现这个技能并再次扩散：一份报告涉及了 50 多个代理。人们在 fork 上应用的修复方法是在两个子代理简报中各追加一行：“不要调用 `/code-review` 或生成额外的代理：直接执行此审查。”有些人更喜欢在 Harness 级别处理，以便所有技能继承这个防护。两者都还没有在已发布的技能中。如果你在无人值守的情况下运行它，请注意代理数量。

**我应该在编写代码的同一个 [会话](https://www.aihero.dev/ai-coding-dictionary/session)中运行它吗？**

最好使用一个全新的会话。正如一位读者所说：“相同的上下文审查自身不是审查，那是带有斜杠命令的确认偏误。”编写会话中的审查代理掌握了塑造代码的所有假设，这正是独立审查者所没有的上下文。这也是为什么人们要求 [implement](https://aihero.dev/skills-implement) 时不带其内置审查步骤的原因：它会在刚刚生成 diff 的会话内部运行审查。从干净的会话中自己调用 `/code-review` 是更诚实的做法。

**每个 ticket 之后，还是最后统一一次？**

两种方式都可行，技能不会替你决定。按 ticket 审查能让每个 diff 足够小，使 Spec 维度有一个明确的规格可供核对，这也是 `implement` 使用的模式。批量到分支末尾可以捕捉到逐个 ticket 审查各自会错过的 ticket 之间的交互。如果你不确定，就按 ticket 审查，并在分支点运行一次最终检查。

**我能相信这些发现吗？**

但需要检查。子代理的输出是一个假设，而不是证据：一个团队报告了十几个被基于文本的审查忽略的破坏性变更。该技能将两份报告逐字或轻度清理后聚合，而不是再次根据文件验证每个声明，因此发现可能会引用错误的位置或夸大影响。在采取行动之前，请阅读每个发现上的引用。每个发现都必须附带一条（一条标准规则、一种坏味道加上其代码块，或一行规范），这正是它可被检查的原因。

**为什么我每次运行它都会发现新的问题？**

因为修复会创造新的表面，而且 Standards 维度的判断性一半在运行之间不是确定性的。一位读者清楚地描述了这个循环：“/code-review 和 /improve-code-architecture 总是每次都发现新东西。我实施修复，重新运行这些技能，然后一次又一次。”没有收敛保证。将一次通过视为一系列线索，对那些有引用规则支撑的线索采取行动，然后停止：不要在循环中运行它，直到它返回干净的结果，因为它是不会的。

**它会审查我未提交的工作吗？**

不会。它比较的是 `<fixed-point>...HEAD`，三点式 diff，从 merge-base 开始测量，并排除暂存区和工作区中的更改。如果 `implement` 没有进行过中间提交，那么即将提交的工作对审查就是不可见的。先提交，再审查，然后修改或添加 fixup。

## 如果它起作用了

* 在任何子代理生成之前，如果遇到无效的 ref 或空的 diff，它就会拒绝启动。
* 报告以两个独立块的形式出现在 `## Standards` 和 `## Spec` 下，而不是合并成一个列表。
* 每条 Standards 发现要么指出仓库某个文件中的一条规则，要么指出十二种坏味道之一，并引用对应的 hunk；每条 Spec 发现都会引用规范中的一行。
* 结尾总结给出每个轴的最差问题，并拒绝选出整体最佳。
* 当没有规范可用时，Spec 块会说明这一点，而不是列出它从代码中推断出的需求。

## 它在系统中的位置

`code-review` 是构建链末尾的审查步骤：`grill-with-docs → to-spec → to-tickets → implement → code-review`。它也可以独立运行在你指向的任何分支或 PR 上。

* [implement](https://aihero.dev/skills-implement) 是最接近的邻居：它驱动构建，并在提交前调用本技能作为自己的收尾审查。
* [to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets) 生成 Spec 轴对照的文档；模糊的规范会让该轴也变得模糊。
* [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是整个代码库的对应技能：本技能只查看一个 diff。

[ask-matt](https://aihero.dev/skills-ask-matt) 会路由到整个技能集合，当你不确定当前情况需要哪个技能时。
