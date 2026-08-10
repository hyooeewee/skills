## 功能说明

`grill-with-docs` 会就计划或设计对你进行访谈，直到你和 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 对它达成一致的理解，并在访谈过程中将词汇和关键决策写入你的仓库。它运行的是与 [grill-me](https://aihero.dev/skills-grill-me) 相同的访谈——一轮提问，然后等待，然后下一轮——但针对的是一个代码库。

它是 **[stateful](https://www.aihero.dev/ai-coding-dictionary/stateful)** 的。其他的 grilled 技巧将 [session](https://www.aihero.dev/ai-coding-dictionary/session) 留在你的脑海中；而这个技巧将文件留在磁盘上。术语一旦解决，就会立即出现在 `CONTEXT.md` 中，而不是在最后批量处理。一个决策通过三个关卡后，它就会作为 ADR 存在。这就是全部的区别，也是人们在使用该技巧时遇到的大部分麻烦的根源：这些产物是仓库中的真实文件，所以当你期望它们时它们可能不存在，而且当有多个人在编写它们时，它们可能会漂移。

## 何时使用

你通过输入 `/grill-with-docs` 来调用它——代理不会主动使用它。

在仓库中变更的开始阶段，当计划仍然模糊且事物的词汇尚未确定时，使用它。这是一个单会话工具。你想要哪个 grilled 技巧取决于你面前有什么：

| 你拥有什么                           | 使用                                                             |
| ------------------------------- | -------------------------------------------------------------- |
| 你根本没有在某个工作目录中工作                 | [grill-me](https://aihero.dev/skills-grill-me)                 |
| 一个仓库，以及你可以在一个会话中解决的一个变更         | `grill-with-docs`                                              |
| 一个太大的努力——无法在一个会话中完成——即全新构建或大型功能 | [wayfinder](https://aihero.dev/skills-wayfinder)               |
| 一个完全没有领域文档的仓库，并且心中没有特定的功能       | `grill-with-docs`针对仓库而非针对变更                                    |
| 一个受阻于某人脑海中知识的决策                 | [to-questionnaire](https://aihero.dev/skills-to-questionnaire) |

Wayfinder 的区别在于会话数量：`/grill-with-docs` 用于单会话规划，`/wayfinder` 用于多会话规划。

## 前置条件

该技巧会写入你的仓库，所以你需要处于一个可以写入的安全位置。已解决的术语会去到根目录的 `CONTEXT.md` 词汇表——或者去到相关上下文的 `CONTEXT.md`，如果根目录的 `CONTEXT-MAP.md` 将仓库标记为多上下文的话。决策会去到 `docs/adr/`。两者都是按需创建的；直到第一个术语或决策结晶之前，什么都不会存在，所以没有什么需要预先搭建的。

它还需要另外两个技巧存在，因为它自己的 `SKILL.md` 只有一行，委托给了它们：[grilling](https://aihero.dev/skills-grilling) 提供访谈，[domain-modeling](https://aihero.dev/skills-domain-modeling) 提供写作。单独安装 `grill-with-docs` 得到的技巧是无法工作的。

## 痕迹

会话会产生三件事，而且它们是不平等的。

| 已解决的内容                  | 去向                               |
| ----------------------- | -------------------------------- |
| 一个术语——项目对某事物的自有词汇       | `CONTEXT.md`内联，即时解析              |
| 一个难以逆转、无上下文令人惊讶且真正权衡的决策 | \`docs/adr/\` 下的 ADR `docs/adr/` |
| 你决定的其他所有内容              | 对话，仅此而已                          |

第三行是让人困惑的地方。`CONTEXT.md` 是一个词汇表，并且有意保持为一个——没有实现细节，没有 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，没有草稿笔记。ADR 需要同时满足所有三个条件才能通过，所以大多数决策不符合资格，大多数会话不会产生 ADR。一个产生更清晰词汇表且零 ADR 的会话是按设计工作的，但这意味着你达成的大部分内容只存在于你达成协议的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window) 中。将同样的对话交给 [to-spec](https://www.aihero.dev/skills-to-spec) 而不是 [clearing](https://www.aihero.dev/ai-coding-dictionary/clearing) 它。

词汇表是关键。领域语言是这个技巧实际上在构建的东西——项目的自有词汇，一旦达成一致，你、代理和你的同事就不必再花费精力去重新推导它们。值得指出的是，并非每个人都认为这能提升代理性能：最尖锐的公开反驳是，一个术语及其通俗英语解释从 [model](https://www.aihero.dev/ai-coding-dictionary/model) 得到的结果是一样的，而且这个词汇表确实压缩了共享它的人之间的沟通。这种理解仍然使词汇表有价值；它只是将价值转移了。

## 它假设只有一个作者

状态化输出假设只有一个人在管理它们。一个在同一个仓库中运行四个月的双人开发团队报告称，在约 20% 的抽样合并 PR 中出现了状态漂移，其中 ADR 引用和 README 声明是漂移最严重的表面——精心策划的人类管理的文档漂移得比代理记忆还差。修剪过时的文档并没有起到作用；同样的清理在几天内又变得过时了。行之有效的方法是直接删除影子状态，并在 CI 中添加确定性的引用和链接检查器。

Related: running the skill repeatedly across unrelated changes in one repo tends to accumulate mixed-topic docs, because nothing separates one session's output from another's. Neither of these is fixed in the skill today.

## 常见问题

**我应该用这个还是 `/wayfinder`？**
Scope 决定它。对于任何你可以在一个会话中解决的事情，使用这个；当努力太大无法在一次会话中完成时，使用 [wayfinder](https://www.aihero.dev/skills-wayfinder)，它首先将工作绘制为决策 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 的地图。Wayfinder 更慢更密集，在范围明确的功能上使用它是常见的错误。它不会取代这个技巧——它可以进入 grilled 会话，用于地图中适合该技巧的部分。

**它运行了，但没有出现 `CONTEXT.md` 也没有出现 ADR。**
两个已知原因。平凡的那个：没有符合条件的东西。ADR 需要通过所有三个关卡，而关于没有任何新词汇的变更的会话确实没有什么可写的。真正的 bug：当技巧在另一个编排层内运行——一个基于规格的开发包装器，一个多代理框架，一个将其作为其他人管道中步骤调用的规则——文件写入部分被报告为静默失败，而访谈仍在运行。这个问题已经被记录但尚未修复。如果你处于这种设置中，在信任会话输出之前检查工作目录。

**它一次性问了所有问题，没有建议，也从未提到 `CONTEXT.md`。**
这是技巧未能加载其两个依赖项。因为 `SKILL.md` 是一行委托代码，如果没有拾取 [grilling](https://www.aihero.dev/skills-grilling) 和 [domain-modeling](https://www.aihero.dev/skills-domain-modeling) 的代理会猜测 grilled 的意思，你会得到一个未区分的问题堆。部分加载是更令人困惑的情况——`grilling` 加载了，`domain-modeling` 没有加载，你得到一个没有书面记录的好访谈。这与 [effort](https://www.aihero.dev/ai-coding-dictionary/effort) 水平和模型相关，也是这个技巧报告最多的问题。如果你怀疑这一点，直接问代理它加载了哪些技巧。

**我的其他决定都去哪了？**
只在对话中。这是对该技巧最实质性的公开抱怨：词汇表不是规格说明，大多数答案不会获得 ADR，也没有账本将每个已解决的答案串联到规格说明、票据和测试。精确答案——顺序保证、否定要求、数字默认值——会被软化成下游较弱的散文，结果看起来是完整的，却缺少你实际决定的东西。今天可用的缓解方法是保留会话并将其直接提供给 [to-spec](https://www.aihero.dev/skills-to-spec)，并根据你自己的答案重新阅读规格说明，而不是假设它捕获了它们。

**我可以把它指向一个根本没有文档的现有仓库吗？**
是的。这是没有 ADR、没有领域语言且没有设计原则的代码库的正确技巧——调用它并说“帮我记录我的仓库”。社区模式将其与 [improve-codebase-architecture](https://www.aihero.dev/skills-improve-codebase-architecture) 配对，用于构建或修复 `CONTEXT.md`。预期要引导它：它会读取代码并询问它发现的内容，而你是指定代码库中已有的哪些词是正确的那个。

会话结束时我该做什么？
技能的结束语往往是开放式的，这是一个已知的不足。在主流程中，答案是 [to-spec](https://aihero.dev/skills-to-spec)，在同一个会话中。如果变更足够小可以立即构建，请直接前往 [implement](https://aihero.dev/skills-implement)。

为什么叫这个名字？
没有人对这个名字满意。有个公开的建议将其重命名为 `grill-domain-model`，这更诚实地描述了其行为。这件事还没有进展。如果重命名最终确定，文档页面也会随之移动，URL 也会改变。

## 判断是否生效

* CONTEXT.md 在会话*期间*逐项发生变化，而不是在最后一次性出现。
* 词汇表读起来纯粹是词汇——你项目的词汇，定义严谨——并且不包含实现细节或类似规格的散文。
* 代码库可以回答的问题通过阅读代码库来回答，而不是问你。
* 你得到的 ADR 很少或没有，而且你得到的那些是你不想不得不重新争论的决策。
* 它会挑战你使用的词，因为你的现有词汇表对它的定义不同。

## 在系统中的位置

`grill-with-docs` 是主构建链的头部：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它在一切被写成规范之前——它产生了 [to-spec](https://aihero.dev/skills-to-spec) 随后进行综合所需要共享的理解和确定的词汇。它的邻近技能是 [grill-me](https://aihero.dev/skills-grill-me)，即没有仓库和文件的相同访谈，以及 [domain-modeling](https://aihero.dev/skills-domain-modeling)，即它所驱动的词汇表和 ADR 规范；两者都建立在 [grilling](https://aihero.dev/skills-grilling) 原语之上。在它上游，[wayfinder](https://aihero.dev/skills-wayfinder) 绘制着超出一次会话规模的努力，并且可以将地图的部分区域传回给它。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
