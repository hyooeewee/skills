## 它做什么

`code-review` 会检查 `HEAD` 与你指定的固定点（一个提交、一个分支、一个标签、`main`、`HEAD~5`）之间的差异，并从两个维度进行审查。**Standards（标准）** 询问代码是否遵循了该仓库的编码方式。**Spec（规格）** 询问代码是否执行了原始 issue 或 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 要求的内容。每个维度都在其自己的 [子代理](https://www.aihero.dev/ai-coding-dictionary/subagent) 中运行，因此它们无法看到对方的推理过程。

该技能从不合并或重新排序这两个维度。报告以每个维度*最严重的一个问题*结束，并拒绝在它们之间选出单一的赢家。一个变更可能通过一个维度却在另一个维度失败。遵循所有约定但实现了错误功能的代码，通过 Standards 却失败于 Spec。完全按 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 要求实现却破坏了仓库约定的代码，则情况相反。混合裁决会让通过的维度掩盖失败的那个。

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

你必须提供固定点。如果不提供，技能会询问而不是猜测。在生成任何东西之前，它会检查 ref 是否可解析且 diff 不为空，因此拼错的分支名会在你面前报错，而不是在两个子代理内部报错。

## 先决条件

Standards 维度不需要任何东西。它会读取仓库中记录的内容（`CODING_STANDARDS.md`、`CONTRIBUTING.md` 等），当仓库没有任何文档时，它会回退到内置的基线。

Spec 维度需要一个存在且可被找到的规格。它按以下顺序查找：

1. 提交消息中的 Issue 引用（`#123`、`Closes #45`、GitLab 的 `!67`），通过 tracker 文档获取。
2. 你作为参数传入的一个路径。
3. 位于 `docs/`、`specs/` 或 `.scratch/` 下、与分支或功能名称匹配的规格文件。
4. 询问你。

步骤 1 依赖于 tracker 文档，该文档由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 编写。没有它，如果你提供路径，该维度仍能工作。如果完全没有规格，技能会跳过 Spec 子代理，报告显示“无可用规格”，而不是凭空捏造需求。

## 两个维度

|         | Standards（标准）                  | Spec（规格）                |
| ------- | ------------------------------ | ----------------------- |
| 问题      | 它构建得对吗？                        | 它是正确的事情吗？               |
| 读取      | 仓库记录的规范，加上坏味道基线                | 原始的 issue 或规格           |
| 报告      | 记录在案的违规（可能是硬性的），以及坏味道（始终是主观判断） | 缺失或部分实现的需求、范围蔓延、错误实现的需求 |
| 每个发现都引用 | 规范文件和规则，或具名的坏味道外加代码块（hunk）     | 规格中的那一行                 |

这种设计是为了避免通用的审查技能不了解你的标准。这样的技能会标记你代码库中刻意的做法，却遗漏了你代码库依赖的不变量。因此，仓库自身的文档是 Standards 维度的 [primary source](https://www.aihero.dev/ai-coding-dictionary/primary-source)，且 **仓库始终拥有最高优先级**。

**坏味道基线**位于仓库标准之下。它源自 Fowler《重构》第 3 章的十二种代码坏味道：神秘命名、重复代码、特性依恋、数据泥团、基本类型偏执、重复 Switch、散弹式修改、发散式变化、投机性泛化、消息链、中间人、被拒遗赠。每种都是一个带标签的启发式判断（“可能存在特性依恋”），绝非硬性违规。每种都说明了什么是坏味道以及如何修复，因此发现问题时会附带修复建议，而不仅仅是抱怨。两个维度都会跳过你的 linter 已强制执行的内容。

## 常见问题

**它与 Claude Code 自带的 `/code-review`冲突。我该怎么办？**

这是该技能最常被报告的问题，且尚未修复。Claude Code 自带 `/code-review`，功能不同：它在 diff 中猎取 bug，而本技能检查规格合规性和仓库标准。安装本库后，二者只能有一个生效，具体哪个取决于安装方式：

* **插件市场。** 每个技能都获得 `mattpocock-skills:` 前缀，内置命令在无前缀名称下变得难以触达。
* **普通技能安装。** 本地文件获胜，本技能遮蔽了内置命令。

一种解决方法是完全移除 Claude Code 的内置技能。这能节省大量 [context](https://www.aihero.dev/ai-coding-dictionary/context)，冲突也就不再重要。遮蔽行为本质上可以说是 Claude Code [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 的 bug（技能作者应有自由为技能任意命名），另一种办法是重命名本地副本。`npx skills update` 会撤销对 frontmatter 的编辑或目录重命名。用户反馈的持久变通方案是将技能 fork 为新名字，并从受管集合中移除 `code-review`。记下 fork 点的提交，以便日后手动同步。

**它的子代理不断调用 `/code-review`，并再次生成更多代理。**

这是一个已知的开放 bug。多人在不止一种 harness 中复现。Standards 和 Spec 的提示词未禁止委托，因此子代理可能再次发现该技能并再次扩散。有报告达到 50 多个代理。fork 版本常用的修复是给两个子代理简报各追加一行：“Do not invoke `/code-review` or spawn additional agents: perform this review directly.” 也有人倾向在 harness 层面处理，让所有技能继承护栏。官方版本尚未包含这些修复。若无人值守运行，请留意代理数量。

**我应该在编写代码的同一个 [会话](https://www.aihero.dev/ai-coding-dictionary/session)中运行它吗？**

建议使用一个新的会话。正如一位读者所言：“同一上下文审查自己不是审查，是带斜杠命令的确认偏误。”在编写会话中审查的代理，其上下文包含了塑造代码的所有假设。独立审查者不会拥有这些上下文。这也是为什么人们要求 [implement](https://aihero.dev/skills-implement) 去掉内置审查步骤，因为那步在刚写完 diff 的会话里运行审查。独立的做法是从干净会话自行调用 `/code-review`。

**每个 ticket 之后，还是最后统一一次？**

两种方式都可行，技能不会替你决定。按 ticket 审查能让每个 diff 足够小，使 Spec 维度有一个明确的规格可供核对，这也是 `implement` 使用的模式。批量到分支末尾可以捕捉到逐个 ticket 审查各自会错过的 ticket 之间的交互。如果你不确定，就按 ticket 审查，并在分支点运行一次最终检查。

**我能相信这些发现吗？**

不检查就不能信任。子代理输出是假设，不是证据。有团队报告，基于文档的审查遗漏了十几处破坏性变更，本技能却发现了。技能原样或略加整理地合并两份报告。它不会逐条对文件复核，所以发现可能引用错位置或夸大影响。行动前请阅读每条发现的引用。技能要求每条发现必须带引用（标准规则、坏味道及其 hunk、或规格行），这才使发现可核查。

**为什么我每次运行它都会发现新的问题？**

每次修复都会增加新的待审查代码，且 Standards 维度中依赖判断力的那半部分每次运行结果不同。一位读者描述了循环："/code-review 和 /improve-code-architecture 每次都能找到新问题。我实现修复，重跑这些技能，又是新问题，周而复始。" 不存在收敛保证。把一次通过当作线索清单。处理那些有规则引用支撑的，然后停手。别在循环里跑到它干净为止，因为它永远不会干净。

**它会审查我未提交的工作吗？**

不会。它对比 `<fixed-point>...HEAD`。三点形式从合并基测量，排除暂存区和工作区变更。如果 `implement` 没有做过临时提交，审查就看不到即将进入下一次提交的工作。先提交，再审查，再 amend 或加 fixup。

## 如果它起作用了

* 在任何子代理生成之前，如果遇到无效的 ref 或空的 diff，它就会拒绝启动。
* 报告以两个独立块的形式出现在 `## Standards` 和 `## Spec` 下，而不是合并成一个列表。
* 每条 Standards 发现要么指出仓库某个文件中的一条规则，要么指出十二种坏味道之一，并引用对应的 hunk；每条 Spec 发现都会引用规范中的一行。
* 结尾总结给出每个轴的最差问题，并拒绝选出整体最佳。
* 当没有规范可用时，Spec 块会说明这一点，而不是列出它从代码中推断出的需求。

## 它在系统中的位置

`code-review` 是构建链末端附近的审查步骤：`grill-with-docs → to-spec → to-tickets → implement → code-review → retro`。它也能单独用于你指向的任何分支或 PR。

* [implement](https://aihero.dev/skills-implement) 是最近的邻居。它驱动构建，并在提交前将此技能作为自己的最终审查进行调用。[implement-spec](https://aihero.dev/skills-implement-spec) 对整个集成分支执行一次相同的操作。
* [retro](https://aihero.dev/skills-retro) 在其后运行并对其进行调优。当某次会话显示审查遗漏了一类错误时，`retro` 会提出检查项或 `CODING_STANDARDS.md` 规则，随后由 Standards 轴读取。
* [pr](https://aihero.dev/skills-pr) 在被审查的工作推送后编写拉取请求正文。
* [to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets) 生成 Spec 轴用于核对的文档，因此模糊的规格会导致该轴也变得模糊。
* [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是全代码库的对应技能，因为此技能仅关注单个差异。

[ask-matt](https://aihero.dev/skills-ask-matt) 会路由到整个技能集合，当你不确定当前情况需要哪个技能时。
