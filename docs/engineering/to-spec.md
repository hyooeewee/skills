## 它做什么

`to-spec` 会将你刚刚进行的对话转化为一份 **[spec](https://www.aihero.dev/ai-coding-dictionary/spec)**，并将其作为一个 issue 发布到你的问题追踪器中。

它不会采访你。当你使用它时，决定已经做完了，所以它会综合已知信息（来自线程、代码库、你的 `CONTEXT.md` 和 ADR），而不是开启新一轮提问。Spec 是已做出决策的记录，而不是做出新决策的地方。

## 何时使用它

你通过输入 `/to-spec` 来调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自己主动调用它。

当构建任务对单个 agent [会话](https://www.aihero.dev/ai-coding-dictionary/session)来说过大，且需要跨多个会话生存时，才使用它。这就是全部的触发条件：

| 你的情况                                                                            | 要运行什么                                                            |
| ------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| 还未做出任何决定                                                                        | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)优先   |
| 已决定，且工作适合一个 [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window) | [implement](https://aihero.dev/skills-implement): 跳过 spec        |
| 已决定，且工作跨越多个会话                                                                   | `/to-spec`，然后 [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一个 [wayfinder](https://aihero.dev/skills-wayfinder)地图已经生成                       | `/to-spec #<map_issue>`                                          |

## 先决条件

`to-spec` 会将 spec 作为 issue 发布，因此 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须预先为此仓库配置好追踪器和 triage 标签词汇表。两种方式都可行：真实的追踪器（如 GitHub），或 `.scratch/` 下的本地 markdown 文件，后者开箱即用。

## Spec 是一份决策记录

Spec 的存在是因为上下文窗口会结束。你在 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 期间达成的一切（解决方案的形态、你争论过的选择、你刻意拒绝的内容）都在即将被清除的一个对话中。Spec 是幸存下来的东西。

因此它不会验证任何东西，也不会决定任何东西。它只捕捉已做出的决定，使用你项目自己的词汇表达，这样一个新的会话可以直接接手工作，而无需你重新解释。spec 中断言的任何你从未说过的东西，都是缺陷。

## 先接缝，后正文

在它写下任何文字之前，`to-spec` 会勾勒出该功能将被测试的**接缝**，并与你核对。它倾向于使用已有的接缝而非新建的，并尽可能采用最高级别的接缝：一次变更的理想接缝数量是一处。

这些达成一致的接缝随后会发挥作用。[tdd](https://aihero.dev/skills-tdd) 只能在预先商定的接缝处工作，而 [code-review](https://www.aihero.dev/skills-code-review) 会对照 spec 审查差异，因此任何未达成一致的接缝都会作为审查发现出现。这种绑定是间接的：它通过这份文档运行，这正是为什么这里的接缝对话值得认真对待，而不是推迟到实现阶段。

## 常见问题

`/to-prd` 去哪了？它就是本技能，已在 v1.1 中重命名。"Spec" 现在是贯穿始终的术语，旧的 `to-prd` 链接已失效；请在新名称下重新安装。取代旧词汇对的是 *spec* 和 *tickets*：spec 是目的地以及修复它的决策，而 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 是到达那里的执行步骤。如果你改变方向，请删除未完成的 tickets 并保留 spec。

为什么 spec 会被加上 `ready-for-agent` 标签？我不希望 agent 基于它进行实现。该标签的意思是"无需进一步分类处理"：文档已经足够完整，可供 agent 工作。这是一个输入标记，而不是工作指令。但是，如果你运行 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) agents 并轮询 `ready-for-agent`，这种区别对它们来说不可见，它们会高兴地尝试一次性构建整个 spec，而不是处理 ticket 片段。这是该技能报告最多的粗糙边缘情况。在它改变之前，请在 AFK agent 的提示词中明确排除父级 spec，或者在 `/to-tickets` 运行后移除该标签。

为什么不直接从 grilling 到 `/to-tickets` 并跳过 spec？通常你应该这样做；spec 只在多会话工作中才值得有这一步。它的价值在于 tickets 是一次性的，而 spec 不是：每个 ticket 都针对一个新的上下文窗口进行了大小调整，会被删除或关闭，而 spec 则作为推理依据所在的地方保留下来。对于单会话的变更，这样做对你没有任何好处，而且你还需要支付一个额外的合成步骤，在此过程中 [model](https://www.aihero.dev/ai-coding-dictionary/model) 可能会发生漂移。直接 grilling → `/implement`。

我刚完成一个 wayfinder 地图。我该喂给它什么？主地图 issue：`/to-spec #<map_issue>`，而不是单个决策 tickets。[wayfinder](https://www.aihero.dev/skills-wayfinder) 产生的是决策而非交付物，分散在地图中；`to-spec` 是将它们整合成一份可构建文档的步骤。直接将地图循环到 `/implement` 会舍弃这种整合。

spec 是供我审查的，还是仅仅给 agent 看的？主要是给 agent 看的，读起来也是这种感觉：完整、密集、参考资料丰富。值得你关注的部分是接缝和超出范围的部分，因为那是两个地方——错误决定最容易发现（成本最低），但也最晚被发现（成本最高）。从头读到尾是人们的一个真实抱怨，而且没有摘要模式：诚实的回答是，如果 spec 令你惊讶，那说明 grilling 太浅了，而不是 spec 太长。

一旦 tickets 开始，我是应该保持 spec 冻结，还是让 agent 重写它？没有任何东西能保持它的同步，所以实际上它是你当时所知事物的快照，一旦实现阶段教会了你新东西，它就会变得过时。一旦工作发布，就把它当作一次性用品。旨在让它长久存在的工件是你的 `CONTEXT.md` 和 ADRs；如果在实现过程中学到了值得保留的东西，它应该属于那里，而不是属于被编辑过的 spec。

我的工作是重构或模块边界，而不是一个特性。模板适用吗？适用性较差，这是一个已知的限制。模板严重依赖用户故事，这是架构工作的错误形态：你最终会写一些没人要求的"故事"，围绕那些实际上关于接口和不变量的决策。相反，应依赖实现决策和测试决策部分，并通过 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 让持久性的架构决策作为 ADR 落地，而不是试图让 spec 来承载它们。

它会检查追踪器中是否有相关工作，或者引用它所尊重的 ADR 吗？两个都不会。它会读取并尊重它所触及领域的 ADR，但不会链接它们，而且在起草之前也不会搜索追踪器中的重叠 issue，所以 spec 可能会悄无声息地重复某人已经提交的工作。如果该领域很活跃，请先自行搜索追踪器。

`/to-tickets` 无法读取我的 spec：它一直截断。非常大的 spec 可能会超出 tracker issue 能干净地处理的大小，而且没有本地副本可以回退。修复方法是上下文卫生：不要在 `/to-spec` 和 `/to-tickets` 之间 [clear](https://www.aihero.dev/ai-coding-dictionary/clearing) 或 [compact](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一个窗口中运行它们，spec 根本不需要重新获取。

## 如果它起作用了

* 它开始写作，而不是向你提出新一轮问题。
* 它在写作之前就把接缝摆在你面前，并且提出的接缝数量尽可能少。
* 它使用你项目的名词来表达，而不是通用的产品管理套话。
* 其中的每一个决策都是你记得做过的。没有任何内容是为了填充某个章节而编造的。
* 超出范围的部分有实质内容：你拒绝的东西通常是页面上最有用的几行。

## 它在系统中的位置

`to-spec` 是主构建链中的一个步骤，而且只在其中的多会话分支上：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它上游的邻居是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)——它负责做本技能只负责记录的决策——以及 [wayfinder](https://aihero.dev/skills-wayfinder)，其完成的地图正好在此处合并到链上。下游的 [to-tickets](https://aihero.dev/skills-to-tickets) 会把 spec 切分为曳光弹式（tracer-bullet）tickets，供 [implement](https://aihero.dev/skills-implement) 构建。当你不确定哪种技能或流程合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
