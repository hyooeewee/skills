## 它做什么

`grill-with-docs` 采访你关于计划或设计的想法，直到你和 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 对它达成一致的理解，并在此过程中将词汇和关键决策写入你的仓库。它运行的是与 [grill-me](https://aihero.dev/skills-grill-me) 相同的访谈（一轮问题，然后等待，然后下一轮），但针对的是代码库。

它是 **[有状态的](https://www.aihero.dev/ai-coding-dictionary/stateful)**。其他所有 grilling 技能都只在你脑中留下 [会话](https://www.aihero.dev/ai-coding-dictionary/session)；这个技能则在磁盘上留下文件。当一个术语被确定时，技能会立即将其写入 `GLOSSARY.md`，而不是在最后批量写入。当一个决策通过三道关卡时，技能会将其作为 ADR 写入。这就是全部区别，这也是人们在使用该技能时遇到的大多数麻烦的根源。这些产物是真实仓库中的真实文件，所以当你期望它们存在时它们可能缺失，当不止一个人编写它们时它们可能会出现漂移。

## 何时使用它

你通过输入 `/grill-with-docs` 来调用它，agent 不会自行主动使用它。

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

该技能会写入你的仓库，所以你需要处于一个可以安全写入的位置。确定的术语会写入根目录下的 `GLOSSARY.md` 词汇表，如果根目录下的 `GLOSSARY-MAP.md` 将仓库标记为多上下文，则写入相关上下文的 `GLOSSARY.md`。决策写入 `docs/adr/`。技能仅在需要时创建它们。在第一个术语或决策确定之前，什么都不存在，所以你无需提前做任何设置。

它还需要另外两个技能存在，因为它自己的 `SKILL.md` 只有一行委托给它们。[grilling](https://aihero.dev/skills-grilling) 提供访谈，[domain-modeling](https://aihero.dev/skills-domain-modeling) 提供写入。仅安装 `grill-with-docs` 得到的是一个无法工作的技能。

## 书面记录

一次会话会产生三样东西，而它们并不对等。

| 被确定的内容                         | 落点                            |
| ------------------------------ | ----------------------------- |
| 一个术语：项目对某事物的专用词                | `GLOSSARY.md`，以内联形式，在它被解析的那一刻 |
| 一个难以撤销、在缺少上下文时令人意外、且是真正权衡取舍的决定 | 位于 `docs/adr/`                |
| 你决定的其他一切                       | 对话，仅此而已                       |

第三行才是让人掉坑的地方。`GLOSSARY.md` 只是一个词汇表。它不包含实现细节、没有 [规格](https://www.aihero.dev/ai-coding-dictionary/spec)，也没有草稿笔记。ADR 需要同时满足三个条件，所以大多数决策不符合资格，大多数会话也不会产生 ADR。一个产出更锐利的词汇表但零 ADR 的会话是按设计工作的，但这意味着你达成的大多数共识只存在于你达成共识时的 [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window) 中。请将同一对话交给 [to-spec](https://aihero.dev/skills-to-spec)，而不是 [清除](https://www.aihero.dev/ai-coding-dictionary/clearing) 它。

词汇表是主要产出。这个技能构建领域语言：项目自己的词汇，达成一次共识，这样你、agent 和你的同事就不必再反复推敲它们。并非所有人都同意这能提升 agent 性能。最强烈的反对意见是：一个术语及其通俗英语展开从 [模型](https://www.aihero.dev/ai-coding-dictionary/model) 得到的结果相同，词汇表主要缩短了共享它的人类之间的沟通。从这个角度看词汇表仍有价值，但价值归属于人类。

## 常见问题

**我该用这个还是 `/wayfinder`？**
范围决定一切。能在一次会话搞定的用这个；努力太大、一场装不下时用 [wayfinder](https://aihero.dev/skills-wayfinder)，它先把工作绘制成决策 [工单](https://www.aihero.dev/ai-coding-dictionary/ticket) 地图。Wayfinder 更慢更密，对一个范围明确的特性却伸手用 wayfinder 是常见错误。它不取代本技能，且能为地图中适合单会话的部分启动 grilling 会话。

**它跑完了，但没有出现 `GLOSSARY.md` 也没有 ADR。**
有两个已知原因。第一是没东西达标。ADR 需要同时过三道关卡，一个没有新词汇的变更会话自然无东西可写。第二是个真 bug。当技能跑在另一个编排层里（spec-driven-development 包装器、多 agent 框架、别人管道里把它当步骤调用的规则）时，用户反馈写文件的那半边悄悄不执行，访谈却照常跑。Bug 已登记但未修复。如果你处于那种环境，信会话产出前先核对工作目录。

**它一次性问完所有问题，没有建议，从未提及 `GLOSSARY.md`。**
这是技能加载其两个依赖失败。因为 `SKILL.md` 是单行委托，没把 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 都拾起来的 agent 会自己猜 grilling 是啥，结果是所有问题一次性抛出、毫无结构。部分加载更令人困惑：`grilling` 进了，`domain-modeling` 没进，访谈体验不错却无纸质痕迹。发生频率取决于模型和 [effort](https://www.aihero.dev/ai-coding-dictionary/effort) 级别，这是该技能被报告最多的问题。怀疑时，直接问 agent 加载了哪些技能。

**我其他的决定都去哪儿了？**
只留在对话里。这是该技能最严重的公开投诉。词汇表不是规格，大多数答案不配得上 ADR，也没有记录把每个确定的答案关联到规格、工单和测试。后续步骤会把精确的答案（顺序保证、否定性需求、数字默认值）软化成较弱的散文，结果看似完整却漏掉了你原本决定的东西。目前的做法是保留会话，直接喂给 [to-spec](https://aihero.dev/skills-to-spec)。然后拿规格对照你自己的答案重读，别假设它全捕捉到了。

**能指向一个完全没有文档的现有仓库吗？**
能。这是给没有 ADR、没有领域语言、没有设计原则的代码库用的正确技能：调用它并说“帮我给仓库写文档”。用户常把它和 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 配对，用来构建或修复 `GLOSSARY.md`。预期你要引导它。它读代码、问你它发现的东西，由你决定代码库里已有的哪些词是对的。

**会话结束后我该干什么？**
技能的结束语常是开放式的，这是个已知问题。主流程里答案是 [to-spec](https://aihero.dev/skills-to-spec)，在同一对话里。如果变更足够小、能马上构建，直接去 [implement](https://aihero.dev/skills-implement)。

**为什么叫这个名字？**
没人对这名字满意。有个公开建议改名为 `grill-domain-model`，更准确地描述行为。目前毫无进展。如果重命名真落地，文档页会随之迁移且 URL 会变。

## 如果它起作用了

* `GLOSSARY.md` 在会话*期间*逐条变更，而不是在最后一次性出现。
* 词汇表读起来像纯词汇表（你的项目术语带有紧凑的定义），不包含实现细节或类似规范的散文。
* 凡是代码库能回答的问题，都应通过阅读代码库来回答，而不是来问你。
* 你得到极少或零 ADR，而得到的那些是你不想再争一次的决定。
* 它会质疑你使用的某个词，因为你现有的词汇表对它有不同的定义。

## 它在系统中的位置

`grill-with-docs` 是主构建链的起点：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

它在任何东西被写成规格之前介入。它产出共享理解和已定词汇，[to-spec](https://aihero.dev/skills-to-spec) 随后综合它们而无需再次访谈你。它的近邻是 [grill-me](https://aihero.dev/skills-grill-me)，同款访谈但无仓库无文件；以及 [domain-modeling](https://aihero.dev/skills-domain-modeling)，它驱动的词汇表与 ADR 规范；两者都用 [grilling](https://aihero.dev/skills-grilling) 原语做访谈。它的上游是 [wayfinder](https://aihero.dev/skills-wayfinder)，为单会话装不下的工作绘制地图，并能把地图的部分下放给它。不确定用哪个技能或流程时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你导向。
