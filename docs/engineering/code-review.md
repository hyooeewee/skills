## 它做什么

`code-review` 会沿着两个维度审查 `HEAD` 与你指定的固定点（一个提交、一个分支、一个标签、`main`、`HEAD~5`）之间的差异。**Standards** 询问代码是否符合该仓库编写代码的方式。**Spec** 询问代码是否实现了原始 issue 或 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 所要求的内容。每个维度都在各自的 [sub-agent](https://www.aihero.dev/ai-coding-dictionary/subagent) 中运行，因此两者互不看到对方的推理。

这两个维度永远不会被合并，也永远不会重新排序。报告以 *每个维度* 的最严重问题作为结尾，并拒绝在它们之间选出一个唯一的赢家，因为一个变更可能在一个维度上通过而在另一个维度上失败：代码遵守了所有约定却实现了错误的东西，会在 Standards 上通过而在 Spec 上失败；代码完全按照 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 所要求的做了，却破坏了仓库的约定，情况则相反。混合的结论会让通过的维度掩盖失败的维度。

## 何时使用它

输入 `/code-review`，或者当你要求审查一个分支、一个 PR、进行中的工作，或任何“自 X 以来”的内容时，代理会自动使用它。

| 你的情况                                             | 选择                                                                                       |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| 存在一个 diff，你想知道它是否构建正确 *和&#x20;*&#x5E76;且做的是正确的事情 | `code-review`                                                                            |
| 你想在 diff 中查找 bug——空指针路径、竞态、差一错误                  | Claude Code 自带的审查，而不是这个（见下文的名字冲突）                                                        |
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

一个不了解你标准的通用审查技能正是这个设计试图避免的——它会标记你代码库中刻意为之的东西，却错过你代码库真正依赖的不变量。因此，仓库自身的文档是 Standards 维度上的 [主要来源](https://www.aihero.dev/ai-coding-dictionary/primary-source)，并且 **仓库始终优先**。

**坏味道基线** 是它下面的底层地板：*重构* 第 3 章中的十二个 Fowler 坏味道——神秘命名（Mysterious Name）、重复代码（Duplicated Code）、依恋情结（Feature Envy）、数据泥团（Data Clumps）、基本类型偏执（Primitive Obsession）、重复的 switch（Repeated Switches）、霰弹式修改（Shotgun Surgery）、发散式变化（Divergent Change）、夸夸其谈的未来性（Speculative Generality）、消息链（Message Chains）、中间人（Middle Man）、被拒绝的遗赠（Refused Bequest）。每一个都是带标签的启发式（“可能的 Feature Envy”），绝不是硬性违规，每一个都表述为 *它是什么* → *如何修复*，因此一个发现会附带一个动作，而不是一句抱怨。你的 linter 已经强制执行的任何内容都会被这两个维度跳过。

## 常见问题

**它与 Claude Code 自带的 `/code-review`冲突。我该怎么办？**

这是关于这个技能被报告最多的问题，而且它还没有被修复。Claude Code 自带自己的 `/code-review`，它做的事情不同——它在 diff 中寻找 bug，而这个技能检查的是规格符合性和仓库标准。安装这个库意味着其中一个会胜出，而哪个胜出取决于你如何安装。通过插件市场安装时，所有东西都会以 `mattpocock-skills:` 前缀作为别名，内置命令在未限定名称下难以访问；通过普通的技能安装，本地文件会胜出，这个技能会遮蔽内置命令。一个干净的解决方案是彻底移除 Claude Code 的内置技能：节省大量 [context](https://www.aihero.dev/ai-coding-dictionary/context)，冲突也就不再重要。遮蔽本身可以说是 Claude Code [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 的一个 bug——技能作者应该可以自由地给技能取任何名字——所以另一个解决方案是重命名本地副本。编辑 frontmatter 或重命名目录会被 `npx skills update` 撤销；用户报告的持久有效的方法是把这个技能 fork 到一个新名称，并从受管理的集合中移除 `code-review`，同时记下你 fork 的提交，以便手动重新同步。

**它的子代理不断调用 `/code-review`，并再次生成更多代理。**

已知的未修复 bug，已有数人复现，并且出现在不止一个 harness 中。Standards 和 Spec 的提示并没有禁止委派，因此子代理可能会重新发现该技能并再次扩散——有报告称代理数量达到了 50 多个。人们在 fork 中应用的修复是在两个子代理简报中各追加一行：“不要调用 `/code-review` 或产生额外代理——直接执行本次审查。”有些人更倾向于在 harness 层面处理，这样每个技能都能继承这一防护。两者都还没有进入发布的技能中。如果你无人值守运行，请留意代理数量。

**我应该在编写代码的同一个 [会话](https://www.aihero.dev/ai-coding-dictionary/session)中运行它吗？**

优先选择一个新的会话。正如一位读者所说：“同一上下文审查自己不是审查，而是带斜杠命令的确认偏误。”编写会话中的审查代理掌握着塑造代码的所有假设，而这正是独立审查者不会拥有的上下文。这也是为什么人们会要求使用 [implement](https://aihero.dev/skills-implement) 而不带其内置审查步骤的原因——它会在刚刚编写 diff 的会话内运行审查。从一个干净的会话中自己调用 `/code-review` 才是诚实的做法。

**每个 ticket 之后，还是最后统一一次？**

两种方式都可行，技能不会替你决定。按 ticket 审查能让每个 diff 足够小，使 Spec 维度有一个明确的规格可供核对，这也是 `implement` 使用的模式。批量到分支末尾可以捕捉到逐个 ticket 审查各自会错过的 ticket 之间的交互。如果你不确定，就按 ticket 审查，并在分支点运行一次最终检查。

**我能相信这些发现吗？**

不能不看就信。子代理的输出是一种假设，而不是证据——一个团队报告说有十几个破坏性变更被基于文字的审查放了过去。技能会逐字或轻度整理地聚合两份报告，而不是对照文件重新验证每一条声称，因此一个发现可能引用错误的位置或夸大影响。在采取行动之前，先阅读每个发现上的引用。每个发现都必须附带一个引用——一条标准规则、一个坏味道及其代码块，或一行规格——正是这一点让它可以被核查。

**为什么我每次运行它都会发现新的问题？**

因为修复会带来新的暴露面，也因为 Standards 轴中依赖主观判断的那一半在不同运行之间并非确定。一位读者直白地描述了这一循环：‘/code-review 和 /improve-code-architecture 每次总能发现新东西。我实施修复，重新运行这些技能，然后一次又一次。’不存在收敛保证。把一次运行当作一份线索清单，只处理那些背后有规则引用的条目，然后停止——不要循环运行直到它变干净为止，因为它不会。

**它会审查我未提交的工作吗？**

不会。它比较的是 `<fixed-point>...HEAD`，三点式 diff，从 merge-base 开始测量，并排除暂存区和工作区中的更改。如果 `implement` 没有进行过中间提交，那么即将提交的工作对审查就是不可见的。先提交，再审查，然后修改或添加 fixup。

## 如果它起作用了

* 在任何子代理生成之前，如果遇到无效的 ref 或空的 diff，它就会拒绝启动。
* 报告以两个独立块的形式出现在 `## Standards` 和 `## Spec` 下，而不是合并成一个列表。
* 每条 Standards 发现要么指出仓库某个文件中的一条规则，要么指出十二种坏味道之一，并引用对应的 hunk；每条 Spec 发现都会引用规范中的一行。
* 结尾总结给出每个轴的最差问题，并拒绝选出整体最佳。
* 当没有规范可用时，Spec 块会说明这一点，而不是列出它从代码中推断出的需求。

## 它在系统中的位置

`code-review` 是构建链末尾的审查步骤——`grill-with-docs → to-spec → to-tickets → implement → code-review`——也可以单独用于你指向的任何分支或 PR。

* [implement](https://aihero.dev/skills-implement) 是最接近的邻居：它驱动构建，并在提交前调用本技能作为自己的收尾审查。
* [to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets) 生成 Spec 轴对照的文档；模糊的规范会让该轴也变得模糊。
* [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是整个代码库层面的对应技能——本技能只查看一个 diff。

[ask-matt](https://aihero.dev/skills-ask-matt) 会路由到整个技能集合，当你不确定当前情况需要哪个技能时。
