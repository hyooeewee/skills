## 它做什么

`grill-with-docs` 会就一个计划或设计对你进行访谈，直到你和 [智能体](https://www.aihero.dev/ai-coding-dictionary/agent) 对它形成共同的理解，并在访谈过程中把词汇和艰难的决定写入你的仓库。它与 [grill-me](https://aihero.dev/skills-grill-me) 运行的是同一种访谈——一轮问题、然后等待、再下一轮——只不过这次指向的是代码库。

它是 **[有状态的（stateful）](https://www.aihero.dev/ai-coding-dictionary/stateful)**。其他所有 grilling 技能都会把 [会话（session）](https://www.aihero.dev/ai-coding-dictionary/session) 留在你的脑海中；这个技能则会把文件留在磁盘上。一个术语一经解析，就会在解析的那一刻落入 `CONTEXT.md`，而不是在最后批量写入。一个决定通过三道门槛后，就会作为 ADR 落地。这就是全部的区别，也是人们对这个技能感到困扰的大部分原因：这些产物是真实仓库中的真实文件，因此它们可能在你期望它们存在时缺席，并且在不止一个人写入时发生漂移。

## 何时使用它

你通过输入 `/grill-with-docs` 来调用它——智能体不会自行使用它。

在仓库中开始一项变更时，当计划仍然模糊、事物的用词尚未确定时，请使用它。它是单会话工具。你想要的 grilling 技能取决于你面前的情况：

| 你拥有什么                           | 选择                                                             |
| ------------------------------- | -------------------------------------------------------------- |
| 你完全没有在一个工作目录中工作                 | [grill-me](https://aihero.dev/skills-grill-me)                 |
| 一个仓库，以及一个你能在一次会话中解决的变更          | `grill-with-docs`                                              |
| 一项过于庞大、无法在一次会话中完成的工作——全新构建、大型功能 | [wayfinder](https://aihero.dev/skills-wayfinder)               |
| 一个没有任何领域文档的仓库，而且你心里也没有特定的功能     | `grill-with-docs`，目标是仓库而不是某个变更                                 |
| 一个因为所需知识在别人脑子里而被卡住的决定           | [to-questionnaire](https://aihero.dev/skills-to-questionnaire) |

wayfinder 的分流归结为会话次数：`/grill-with-docs` 用于单会话规划，`/wayfinder` 用于多会话规划。

## 先决条件

该技能会写入你的仓库，因此你需要处于一个可以安全写入的地方。已解析的术语会进入根目录下的 `CONTEXT.md` 术语表——或者，如果根目录下的 `CONTEXT-MAP.md` 将仓库标记为多上下文，则进入相关上下文的 `CONTEXT.md`。决定会进入 `docs/adr/`。两者都是惰性创建的；在第一个术语或决定结晶之前，什么都不存在，因此前期无需搭建任何脚手架。

它还需要另外两个技能在场，因为它自己的 `SKILL.md` 只有一行，委托给它们：[grilling](https://aihero.dev/skills-grilling) 提供访谈，[domain-modeling](https://aihero.dev/skills-domain-modeling) 提供写作。单独安装 `grill-with-docs`，你会得到一个无法工作的技能。

## 书面记录

一次会话会产生三样东西，而它们并不对等。

| 被确定的内容                         | 落点                           |
| ------------------------------ | ---------------------------- |
| 一个术语——项目自己对某事物的称呼              | `CONTEXT.md`，以内联形式，在它被解析的那一刻 |
| 一个难以撤销、在缺少上下文时令人意外、且是真正权衡取舍的决定 | 位于 `docs/adr/`               |
| 你决定的其他一切                       | 对话，仅此而已                      |

第三行才是让人措手不及的地方。`CONTEXT.md` 是一个术语表，并且刻意保持为术语表——没有实现细节，没有 [规范（spec）](https://www.aihero.dev/ai-coding-dictionary/spec)，没有草稿笔记。ADR 需要同时通过三个门槛，所以大多数决定并不符合条件，大多数会话也不会产生 ADR。一次会话若能产出更清晰的术语表和零个 ADR，那就是按设计在工作，但这意味着你达成一致的大部分内容只存在于你达成一致的 [上下文窗口（context window）](https://www.aihero.dev/ai-coding-dictionary/context-window) 中。与其 [清空（clearing）](https://www.aihero.dev/ai-coding-dictionary/clearing) 这段对话，不如把它交给 [to-spec](https://aihero.dev/skills-to-spec)。

术语表才是重点。领域语言是这个技能真正在构建的东西——项目自己的词汇，一次达成一致，这样你、智能体以及你的同事就不必再付出代价去重新推导它们。值得说明的是，并非所有人都认同这会为你带来智能体性能的提升：最尖锐的公开反驳是，一个术语与它的平白英语解释从 [模型（model）](https://www.aihero.dev/ai-coding-dictionary/model) 那里得到的结果相同，而且这套词汇真正压缩的是共享它的那些人类之间的沟通。这种解读仍然让术语表有价值；只是把价值转移到了别处。

## 常见问题

**我应该用这个还是 `/wayfinder`？**
看范围决定。任何你能在一次会话中解决的事情都用这个；当工作量太大、一次会话装不下时，使用 [wayfinder](https://aihero.dev/skills-wayfinder)，它会先把工作绘制成一张由决策 [工单（ticket）](https://www.aihero.dev/ai-coding-dictionary/ticket) 组成的地图。Wayfinder 更慢、更密集，在一个范围明确的功能上去使用它是常见的错误。它并不会取代这个技能——它可以进入一场 grilling 会话，来处理地图中适合单会话的部分。

**它运行了，但没有出现 `CONTEXT.md`，也没有出现 ADR。**
有两个已知原因。平常的那个：没有符合条件的内容。ADR 需要同时通过三道门槛，而一个关于没有新词汇的变更的会话确实无话可写。真正的 bug：当该技能在另一个编排层内部运行时——规范驱动开发的包装器、多智能体框架、或者一条把它作为别人流水线中的步骤来调用的规则——据说写入文件的那一半会悄悄不执行，而访谈仍然会运行。这个问题已立案但尚未修复。如果你处于这种设置中，在相信会话的输出之前，请先检查工作目录。

**它一次性把所有问题都问了，没有给出任何建议，而且从未提到 `CONTEXT.md`。**
这是该技能没能加载它的两个依赖项。因为 `SKILL.md` 只是一行委托，一个没有拾取 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 的智能体会去猜测 grilling 是什么意思，于是你会得到一场不加区分的提问倾倒。部分加载是更令人困惑的情况——`grilling` 加载了，而 `domain-modeling` 没有，于是你得到一场不错的访谈，却没有书面记录。这与模型和 [effort](https://www.aihero.dev/ai-coding-dictionary/effort) 水平相关，也是这个技能被报告最多的问题。如果你怀疑这一点，直接问智能体它加载了哪些技能。

**我所有其他的决定都去哪儿了？**
只进入了对话。这是关于该技能最实质性的公开抱怨：术语表不是规范，大多数答案也够不上 ADR，而且没有一本账本把每个已解答的答案对应到规范、工单和测试。精确的答案——顺序保证、否定性需求、数值默认值——会在下游被淡化成更弱的表述，结果可能看起来很完整，却恰恰缺少了你实际决定的东西。目前可用的缓解措施是保留会话，并直接把它喂给 [to-spec](https://aihero.dev/skills-to-spec)，然后根据你自己的答案重新阅读规范，而不是假设它已经捕捉到了这些答案。

**我可以把它指向一个完全没有任何文档的现有仓库吗？**
可以。对于没有 ADR、没有领域语言、没有设计原则的代码库，这正是合适的技能——调用它并说“帮我记录我的仓库”。社区的模式是将它与 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 搭配使用，来构建或修复 `CONTEXT.md`。要做好引导它的准备：它会阅读代码，并就它发现的内容向你提问，而你才是那个指出代码库中已有的哪些词是正确的词的人。

**会话结束时我该怎么办？**
该技能的结束消息往往是开放式的，这是一个已知的粗糙之处。在主流程中，答案是继续同一个对话并转至 [to-spec](https://aihero.dev/skills-to-spec)。如果变更足够小、可以立即构建，那就直接进入 [implement](https://aihero.dev/skills-implement)。

**为什么叫这个名字？**
没有人对这个名字满意。目前有一个开放的提议，将其改名为 `grill-domain-model`，这个名称更诚实地描述了其行为。这方面没有任何进展。如果重命名最终落地，文档页面和 URL 也会随之改变。

## 如果它起作用了

* `CONTEXT.md` 在*会话期间*逐条术语地变化，而不是在结束时一次性出现。
* 词汇表读起来应当只有纯粹的词汇——你项目中的词语及其精确定义——不包含任何实现细节或规格说明式的文字。
* 凡是代码库能回答的问题，都应通过阅读代码库来回答，而不是来问你。
* 你几乎得不到或根本得不到 ADR，而得到的那些也是你不得不重新讨论时会感到恼火的决定。
* 它会质疑你使用的某个词，因为你现有的词汇表对它有不同的定义。

## 它在系统中的位置

`grill-with-docs` 是主构建链的起点：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它在任何内容被写成规格之前出现——它产生共同理解和定型的词汇，随后 [to-spec](https://aihero.dev/skills-to-spec) 无需再次访谈你即可将它们综合成文档。它的近邻是 [grill-me](https://aihero.dev/skills-grill-me)——没有仓库和文件的同一访谈，以及 [domain-modeling](https://aihero.dev/skills-domain-modeling)——它所驱动的词汇表与 ADR 规范；两者都建立在 [grilling](https://aihero.dev/skills-grilling) 原语之上。在它的上游，[wayfinder](https://aihero.dev/skills-wayfinder) 会规划那些一次会话难以完成的工作，并可将部分地图回传给它。当你不确定哪个技能或流程合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
