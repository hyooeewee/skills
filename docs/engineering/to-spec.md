## 它做什么

`to-spec` 会将你刚刚进行的对话转化为一份 **[spec](https://www.aihero.dev/ai-coding-dictionary/spec)**，并将其作为一个 issue 发布到你的问题追踪器中。

它不会对你进行访谈。当你使用它时，决策已经完成，所以它会综合已知信息——来自对话线程、代码库、你的 `CONTEXT.md` 和 ADR——而不是开启新一轮提问。spec 是已做出决策的记录，而不是产生新决策的地方。

## 何时使用它

你通过输入 `/to-spec` 来调用它——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自行调用它。

当构建任务对单个 agent [会话](https://www.aihero.dev/ai-coding-dictionary/session)来说过大，且需要跨多个会话生存时，才使用它。这就是全部的触发条件：

| 你的情况                                                                            | 要运行什么                                                            |
| ------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| 还未做出任何决定                                                                        | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)优先   |
| 已决定，且工作适合一个 [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window) | [implement](https://aihero.dev/skills-implement)—— 跳过 spec       |
| 已决定，且工作跨越多个会话                                                                   | `/to-spec`，然后 [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一个 [wayfinder](https://aihero.dev/skills-wayfinder)地图已经生成                       | `/to-spec #<map_issue>`                                          |

## 先决条件

`to-spec` 会将 spec 作为 issue 发布，因此 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须预先为此仓库配置好追踪器和 triage 标签词汇表。两种方式都可行：真实的追踪器（如 GitHub），或 `.scratch/` 下的本地 markdown 文件，后者开箱即用。

## Spec 是一份决策记录

spec 之所以存在，是因为上下文窗口会结束。你在 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 过程中解决的一切——解决方案的形态、你争论过的抉择、你刻意拒绝的东西——都在一段即将被清除的对话中。spec 就是在那之后留存下来的东西。

因此它不会验证任何东西，也不会决定任何东西。它只捕捉已做出的决定，使用你项目自己的词汇表达，这样一个新的会话可以直接接手工作，而无需你重新解释。spec 中断言的任何你从未说过的东西，都是缺陷。

## 先接缝，后正文

在动笔之前，`to-spec` 会先勾勒出该特性将要进行测试的**接缝（seams）**，并与你核对。它倾向于使用已有的接缝而非新造接缝，而且会尽可能选择最高的接缝——一次变更的理想接缝数量是一个。

那些达成一致的接缝随后会继续传递。[tdd](https://aihero.dev/skills-tdd) 只在预先商定的接缝处工作，[code-review](https://aihero.dev/skills-code-review) 会依据 spec 审查 diff，因此任何未经商定的接缝都会作为审查发现暴露出来。这种绑定是间接的——它通过本文档来传递——这正是为什么在这里认真对待接缝对话，而不是把它推迟到实现阶段。

## 常见问题

**`/to-prd` 去哪了？**
它就是本技能，在 v1.1 中更名了。“Spec”现在是唯一的贯穿性术语，旧的 `to-prd` slug 已废弃——请以新名称重新安装。取代旧词汇体系的一对术语是 *spec* 和 *tickets*：spec 是目的地以及确定它的决策，[tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 是到达那里的执行步骤。如果你改变了方向，请删除未完成的 tickets 并保留 spec。

**为什么 spec 会带有 `ready-for-agent` 标签？我不希望 agent 依据它来实施。**
这个标签的意思是“无需进一步分诊”——文档已经足够完整，agent 可以据此工作。它是一个输入标记，而不是工作指令。但如果你运行轮询 `ready-for-agent` 的 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) agents，它们看不到这种区别，会很乐意尝试一次性构建整个 spec，而不是拾取 ticket 切片。这是该技能收到报告最多的粗糙之处。在这一问题改变之前，请在 AFK agent 的提示词中显式排除父 spec，或者在 `/to-tickets` 运行后移除该标签。

**为什么不直接从 grilling 进入 `/to-tickets` 并跳过 spec？**
通常你应该这样做——spec 只在多会话工作中才值得占据这一步。它的价值在于：tickets 是一次性的，而 spec 不是：每个 ticket 的规模都针对一个全新的上下文窗口，且会被删除或关闭，而 spec 仍然是承载其背后推理的唯一场所。在单会话变更中，这不会给你带来任何好处，而你却多付出了一步综合（synthesis），[model](https://www.aihero.dev/ai-coding-dictionary/model) 可能会在此处发生偏移。直接 grilling → `/implement` 吧。

**我刚完成一张 wayfinder 地图。我该给它喂什么？**
主地图 issue——`/to-spec #<map_issue>`，而不是各个独立的决策 ticket。[wayfinder](https://aihero.dev/skills-wayfinder) 产出的是一张地图上散布的决策而非交付物；`to-spec` 就是将它们折叠成一份可构建文档的步骤。如果把地图直接循环进 `/implement`，就会丢掉这种折叠。

**spec 是给我审查的，还是只给 agent 看的？**
主要是给 agent 看的，读起来也正是如此——完整、密集、引用繁重。值得你关注的部分是接缝和 out-of-scope（范围之外）部分，因为在这两处，错误决策的发现成本最低，而事后发现的代价最高。通读全文是人们真实存在的一个抱怨，而且没有摘要模式：诚实的回答是，如果 spec 让你感到意外，那是 grilling 不够深入，而不是 spec 太长。

**tickets 开始后，我是让 spec 保持冻结，还是让 agent 重写它？**
没有任何机制让它保持同步，所以实际上它是你当时所知信息的快照，而且第一次实现教会你新东西时它就会过时。一旦工作交付，就把它当作可弃之物。那些旨在比它更长命的工件是你的 `CONTEXT.md` 和你的 ADR——如果在实现过程中学到的某些东西值得留存，它们应该放在那里，而不是放在被编辑过的 spec 中。

**我的工作是重构或模块边界，而不是特性。模板适用吗？**
不太适用，这是一个已知的局限。模板严重依赖用户故事（user stories），这对架构类工作来说形态不对——你会围绕那些真正关乎接口和不变量（invariants）的决策，写出没人要的故事。请改用 implementation-decisions 和 testing-decisions 部分，并让那些持久的架构决策通过 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 落为 ADR，而不是试图让 spec 来承载它们。

**它会检查追踪器中相关工作，或引用它所遵循的 ADR 吗？**
两者都不会。它会读取并遵循它触及领域内的 ADR，但不会链接它们，也不会在起草前搜索追踪器中是否有重叠的 issue——因此一份 spec 可能会悄悄重复某人已经提交的工作。如果该领域很繁忙，请先自行搜索追踪器。

**`/to-tickets` 读不了我的 spec——它一直截断。**
非常大的 spec 可能会超出追踪器 issue 能干净回传的规模，而且没有本地副本可以回退。解决办法是上下文卫生：不要在 `/to-spec` 和 `/to-tickets` 之间 [clear](https://www.aihero.dev/ai-coding-dictionary/clearing) 或 [compact](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一个窗口中运行它们，spec 就根本不需要被重新获取。

## 如果它起作用了

* 它开始写作，而不是向你提出新一轮问题。
* 它在写作之前就把接缝摆在你面前，并且提出的接缝数量尽可能少。
* 它使用你项目的名词来表达，而不是通用的产品管理套话。
* 其中的每一个决策都是你记得做过的。没有任何内容是为了填充某个章节而编造的。
* out-of-scope 部分有真实的内容——你拒绝的东西通常是页面上最有用的几行。

## 它在系统中的位置

`to-spec` 是主构建链中的一个步骤，而且只在其中的多会话分支上：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它上游的邻居是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)——它负责做本技能只负责记录的决策——以及 [wayfinder](https://aihero.dev/skills-wayfinder)，其完成的地图正好在此处合并到链上。下游的 [to-tickets](https://aihero.dev/skills-to-tickets) 会把 spec 切分为曳光弹式（tracer-bullet）tickets，供 [implement](https://aihero.dev/skills-implement) 构建。当你不确定哪种技能或流程合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
