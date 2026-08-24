## 它做什么

`grill-with-docs` 采访你关于计划或设计的想法，直到你和 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 对它达成一致的理解，并在此过程中将词汇和关键决策写入你的仓库。它运行的是与 [grill-me](https://aihero.dev/skills-grill-me) 相同的访谈（一轮问题，然后等待，然后下一轮），但针对的是代码库。

它是 **[有状态的（stateful）](https://www.aihero.dev/ai-coding-dictionary/stateful)**。其他所有 grilling 技能都会把 [会话（session）](https://www.aihero.dev/ai-coding-dictionary/session) 留在你的脑海中；这个技能则会把文件留在磁盘上。一个术语一经解析，就会在解析的那一刻落入 `CONTEXT.md`，而不是在最后批量写入。一个决定通过三道门槛后，就会作为 ADR 落地。这就是全部的区别，也是人们对这个技能感到困扰的大部分原因：这些产物是真实仓库中的真实文件，因此它们可能在你期望它们存在时缺席，并且在不止一个人写入时发生漂移。

## 何时使用它

你通过输入 `/grill-with-docs` 来调用它；代理不会主动使用它。

在仓库中开始一项变更时，当计划仍然模糊、事物的用词尚未确定时，请使用它。它是单会话工具。你想要的 grilling 技能取决于你面前的情况：

| 你拥有什么                       | 选择                                                             |
| --------------------------- | -------------------------------------------------------------- |
| 你完全没有在一个工作目录中工作             | [grill-me](https://aihero.dev/skills-grill-me)                 |
| 一个仓库，以及一个你能在一次会话中解决的变更      | `grill-with-docs`                                              |
| 一次会话无法容纳的努力（一个全新的构建，一个大型功能） | [wayfinder](https://aihero.dev/skills-wayfinder)               |
| 一个没有任何领域文档的仓库，而且你心里也没有特定的功能 | `grill-with-docs`，目标是仓库而不是某个变更                                 |
| 一个因为所需知识在别人脑子里而被卡住的决定       | [to-questionnaire](https://aihero.dev/skills-to-questionnaire) |

wayfinder 的分流归结为会话次数：`/grill-with-docs` 用于单会话规划，`/wayfinder` 用于多会话规划。

## 先决条件

该技能会写入你的仓库，所以你需要在一个可以安全写入的地方。解析后的术语会进入根目录下的 `CONTEXT.md` 词汇表，或者进入相关上下文的 `CONTEXT.md`，如果根目录下的 `CONTEXT-MAP.md` 将仓库标记为多上下文。决策会进入 `docs/adr/`。两者都是按需创建的；在第一个术语或决策结晶之前，什么都不存在，所以不需要预先搭建结构。

它还需要另外两个技能在场，因为它自己的 `SKILL.md` 只有一行，委托给它们：[grilling](https://aihero.dev/skills-grilling) 提供访谈，[domain-modeling](https://aihero.dev/skills-domain-modeling) 提供写作。单独安装 `grill-with-docs`，你会得到一个无法工作的技能。

## 书面记录

一次会话会产生三样东西，而它们并不对等。

| 被确定的内容                         | 落点                           |
| ------------------------------ | ---------------------------- |
| 一个术语：项目对某事物的专用词                | `CONTEXT.md`，以内联形式，在它被解析的那一刻 |
| 一个难以撤销、在缺少上下文时令人意外、且是真正权衡取舍的决定 | 位于 `docs/adr/`               |
| 你决定的其他一切                       | 对话，仅此而已                      |

那第三行是让人上当的地方。`CONTEXT.md` 是一个词汇表，并故意保持为一个：没有实现细节，没有 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，没有草稿笔记。ADR 需要同时满足三个条件，因此大多数决定不符合资格，大多数会话也不会产生 ADR。一个产生更清晰词汇表且零 ADR 的会话是按设计工作的，但这意味着你同意的大部分内容仅存在于你达成一致的那个 [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window) 中。将同样的对话交给 [to-spec](https://www.aihero.dev/skills-to-spec)，而不是 [清除](https://www.aihero.dev/ai-coding-dictionary/clearing) 它。

词汇表是关键。领域语言是这个技能实际上在构建的东西：项目的专用词，达成一致一次，这样你、代理和同事就停止为了重新推导它们而付出代价。值得说的是，并非每个人都同意这能提升代理性能：最尖锐的公开反对意见是，一个术语及其通俗英语扩展从 [模型](https://www.aihero.dev/ai-coding-dictionary/model) 那里得到相同的结果，而且该词汇表确实压缩了共享它的那些人类之间的沟通。这种解读仍然让词汇表有价值；它只是转移了价值。

## 常见问题

**我应该使用这个还是 `/wayfinder`？**
范围决定它。对于任何你能在一次会话中解决的事情，使用这个；当工作量太大无法在一次会话中容纳，并且它首先将工作绘制为决策 [票据](https://www.aihero.dev/ai-coding-dictionary/ticket) 地图时，使用 [wayfinder](https://www.aihero.dev/skills-wayfinder)。Wayfinder 更慢、更密集，在范围明确的功能上使用它是常见的错误。它不能替代这个技能：它可以进入访谈会话，处理地图中适合的部分。

**它运行了，但 `CONTEXT.md` 和 ADR 都没有出现。**
两个已知原因。平凡的那个：没有符合资格的内容。ADR 需要三个门槛，而关于没有新词汇的变更的会话确实没有什么可写的。真正的 bug：当该技能在另一个编排层（一个驱动开发的规范包装器、一个多代理框架、一个将其作为他人流程中的步骤调用的规则）内部运行时，文件写入部分被报告为静默失败，而访谈仍在运行。这个问题已被记录但未修复。如果你处于这种设置中，在信任会话输出之前检查工作目录。

**它一次性问了一切，没有建议，并且从未提及 `CONTEXT.md`。**
这是该技能未能加载其两个依赖项的迹象。由于 `SKILL.md` 是一行委托，如果没有拾取 [grilling](https://www.aihero.dev/skills-grilling) 和 [domain-modeling](https://www.aihero.dev/skills-domain-modeling) 的代理会猜测 grilling 的含义，你会得到一个未区分的问题倾倒。部分加载是更令人困惑的情况：`grilling` 加载了，`domain-modeling` 没有加载，你会得到一个没有书面记录的良好访谈。它与模型和 [工作量](https://www.aihero.dev/ai-coding-dictionary/effort) 级别相关，并且是这个技能报告最多的问题。如果你怀疑这一点，直接问代理它加载了哪些技能。

**我的其他所有决定去哪了？**
仅在对话中。这是关于该技能最实质性的公开投诉：词汇表不是规范，大多数答案不会获得 ADR，也没有账本将每个已解决的答案通过关联到规范、票据和测试。精确答案（排序保证、负面需求、数字默认值）在下游被软化成较弱的散文，结果看起来很完整，但缺少你实际上决定的东西。今天可用的缓解措施是保持会话并将其直接喂给 [to-spec](https://www.aihero.dev/skills-to-spec)，并根据你自己的答案重新阅读规范，而不是假设它捕获了它们。

**我可以将其指向一个根本没有文档的现有仓库吗？**
可以。这是没有 ADR、没有领域语言和没有设计原则的代码库的正确技能：调用它并说“帮我记录我的仓库”。社区模式将其与 [improve-codebase-architecture](https://www.aihero.dev/skills-improve-codebase-architecture) 配对，用于构建或修复 `CONTEXT.md`。预期要引导它：它会读取代码并询问它发现的内容，而你要说出代码库中已有的哪些词是正确的。

**会话结束时我该怎么办？**
该技能的结束消息往往是开放式的，这是一个已知的粗糙之处。在主流程中，答案是继续同一个对话并转至 [to-spec](https://aihero.dev/skills-to-spec)。如果变更足够小、可以立即构建，那就直接进入 [implement](https://aihero.dev/skills-implement)。

**为什么叫这个名字？**
没有人对这个名字满意。目前有一个开放的提议，将其改名为 `grill-domain-model`，这个名称更诚实地描述了其行为。这方面没有任何进展。如果重命名最终落地，文档页面和 URL 也会随之改变。

## 如果它起作用了

* `CONTEXT.md` 在*会话期间*逐条术语地变化，而不是在结束时一次性出现。
* 词汇表读起来像纯词汇表（你的项目术语带有紧凑的定义），不包含实现细节或类似规范的散文。
* 凡是代码库能回答的问题，都应通过阅读代码库来回答，而不是来问你。
* 你几乎得不到或根本得不到 ADR，而得到的那些也是你不得不重新讨论时会感到恼火的决定。
* 它会质疑你使用的某个词，因为你现有的词汇表对它有不同的定义。

## 它在系统中的位置

`grill-with-docs` 是主构建链的起点：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它在任何内容被写成规范之前：它产生共享的理解和已确定的词汇，然后 [to-spec](https://www.aihero.dev/skills-to-spec) 在不再次采访你的情况下综合它们。它的近邻是 [grill-me](https://www.aihero.dev/skills-grill-me)，同一个访谈但没有仓库和文件，以及 [domain-modeling](https://www.aihero.dev/skills-domain-modeling)，它推动的词汇表和 ADR 规范；两者都位于 [grilling](https://www.aihero.dev/skills-grilling) 原语之上。在它上游，[wayfinder](https://www.aihero.dev/skills-wayfinder) 绘制太大无法一次会话容纳的努力，并可以将地图的部分返回给它。当你不确定哪个技能或流程适合时，[ask-matt](https://www.aihero.dev/skills-ask-matt) 会为你路由。
