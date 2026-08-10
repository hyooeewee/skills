## 功能说明

`to-spec` 将你刚刚进行的对话转换为 **[spec](https://www.aihero.dev/ai-coding-dictionary/spec)**，并将其作为单个问题发布到你的问题跟踪器中。

它不会对你进行采访。当你使用它时，决定已经做出了，因此它会综合已知的信息——来自对话线程、代码库、你的 `CONTEXT.md` 和 ADRs——而不是开启新一轮的提问。规格说明是已做出决定的记录，而不是做出新决定的地方。

## 何时使用

你通过输入 `/to-spec` 来调用它——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会主动去使用它。

当构建任务太大，无法由一个 agent [session](https://www.aihero.dev/ai-coding-dictionary/session) 完成，并且必须能够承受被拆分到多个会话中时，使用它。这就是整个触发条件：

| 所在位置                                                                           | 运行什么                                                             |
| ------------------------------------------------------------------------------ | ---------------------------------------------------------------- |
| 你尚未做出任何决定                                                                      | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)首先   |
| 已做决定，且工作适合 [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window) | [implement](https://aihero.dev/skills-implement)— 跳过规格说明         |
| 已做决定，且工作跨越多个会话                                                                 | `/to-spec`，然后 [to-tickets](https://aihero.dev/skills-to-tickets) |
| 一个 [wayfinder](https://aihero.dev/skills-wayfinder)地图已清除                       | `/to-spec #<map_issue>`                                          |

## 前置条件

`to-spec` 将规格说明发布为一个问题，因此必须先通过 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 为此仓库配置跟踪器和分类标签词汇。两种方式都可以：像 GitHub 这样的真实跟踪器，或 `.scratch/` 下的本地 markdown 文件，后者开箱即支持。

## 规格说明是一份决策记录

规格说明之所以存在，是因为上下文窗口会结束。你在 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 期间达成的所有共识——解决方案的形状、你争论过的选择、你故意拒绝的内容——都在即将被清除的一次对话中。规格说明就是幸存下来的东西。

因此它不验证任何东西，也不做任何决定。它以你项目的自身词汇捕捉已做出的决定，以便新的会话可以接手工作，而无需你重新解释。规格说明中任何你从未实际说过的话都是缺陷。

## 在正文之前先确定测试接口点

在写入一个字之前，`to-spec` 会勾勒出该功能将被测试的 **seams（测试接口点）**，并与你确认。它优先选择已存在的接口点而不是新的，并尽可能采用最高的那个——一次变更中的理想数量是一个。

那些达成一致的接口点随后会被传递。`[tdd](https://aihero.dev/skills-tdd)` 只能在预先达成一致的接口点处工作，而 `[code-review](https://aihero.dev/skills-code-review)` 会对照规格说明审查差异，因此没有人同意的接口点会作为审查发现出现。这种绑定是间接的——它通过本文档——这就是为什么这里的接口点对话值得认真对待，而不是推迟到实现阶段的原因。

## 常见问题

**`/to-prd` 去哪了？**
它就是这个技能，在 v1.1 中重命名了。"Spec" 现在是贯穿始终的术语，旧的 `to-prd` slug 已死——请在新的名称下重新安装。取代旧词汇的一对是 *spec* 和 *tickets*：spec 是目的地以及修复它的决策，[tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 是到达那里的执行步骤。如果你转向，请删除未完成的 tickets 并保留 spec。

**为什么规格说明会获得 `ready-for-agent` 标签？我不想让 agent 基于它进行实现。**
该标签意味着"不需要进一步分类"——文档足够完整，可供 agent 工作。它是一个输入标识，而不是工作指令。但是，如果你运行 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) agent 定期轮询 `ready-for-agent`，它们看不到这种区别，它们会愉快地尝试在一次运行中构建整个规格说明，而不是接手 ticket 片段。这是该技能报告最多的问题之一。直到它改变之前，请在你的 AFK agent 提示词中明确排除父规格说明，或者在 `/to-tickets` 运行后移除标签。

**为什么不直接从 grilling 到 `/to-tickets` 并跳过规格说明？**
通常你应该这样做——规格说明只有在多会话工作中才有存在的价值。它的作用在于 tickets 是一次性的，而规格说明不是：每个 ticket 都针对一个新的上下文窗口进行大小调整并被删除或关闭，而规格说明保留为推理依据所在的地方。对于单会话变更，这对你没有任何好处，而且你还要支付一个额外的合成步骤，在此步骤中 [model](https://www.aihero.dev/ai-coding-dictionary/model) 可能会漂移。请进行 grilling → `/implement`。

**我刚刚完成了一个 wayfinder 地图。我该喂给它什么？**
主地图问题——`/to-spec #<map_issue>`，而不是单个决策 tickets。[wayfinder](https://aihero.dev/skills-wayfinder) 产生的是决策，而不是交付物，分散在地图上；`to-spec` 是将它们合并为一个可构建文档的步骤。将地图直接循环到 `/implement` 会丢弃这种合并。

**规格说明是为了让我审查，还是仅仅为了 agent？**
主要是为了 agent，读起来也是这样——完整、密集、引用繁多。值得你关注的部分是 seams 和超出范围的部分，因为那是两个错误决定最便宜被捕获且稍后最昂贵被发现的地点。从头读到尾是人们真正的抱怨，而且没有摘要模式：诚实的答案是，如果规格说明让你感到意外，那么 grilling 太浅了，而不是规格说明太长。

**一旦 tickets 开始，我是应该保持规格说明冻结，还是让 agent 重写它？**
没有什么能保持它的同步，因此在实践中，它只是你当时所知的一个快照，第一次实现教你某些东西时它就会变得过时。工作发布后将其视为一次性使用。旨在比它更长久存在的工件是你的 `CONTEXT.md` 和你的 ADRs——如果在实现过程中学到的某些东西值得保留，它应该属于那里，而不是在编辑后的规格说明中。

**我的工作是一次重构或模块边界，而不是一个功能。模板合适吗？**
合适度较差，这是一个已知的局限性。模板严重依赖用户故事，这对于架构工作来说形状是错误的——你最终会写出没人要求的故事，围绕那些实际上是关于接口和不变量的决策。相反，应依赖实现决策和测试决策部分，并通过 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 让持久化的架构决策作为 ADRs 落地，而不是试图让规格说明来承载它们。

**它会检查跟踪器中的相关工作，还是引用它所尊重的 ADRs？**
两者都不会。它会读取并尊重它所触及领域的 ADRs，但它不会链接它们，并且在起草前不会搜索跟踪器中的重叠问题——因此规格说明可能会默默地复制别人已经提交的工作。如果该领域很忙，请先自己搜索跟踪器。

**`/to-tickets` 无法读取我的规格说明——它一直被截断。**
非常大的规格说明可能会超出跟踪器问题能干净地返回的范围，而且没有本地副本可供回退。解决方案是上下文卫生：不要在 `/to-spec` 和 `/to-tickets` 之间 [clear](https://www.aihero.dev/ai-coding-dictionary/clearing) 或 [compact](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一个窗口中运行它们，规格说明根本不需要重新获取。

## 判断是否生效

* 它开始编写而不是问新一轮的问题。
* 它在编写之前先提出测试接口点，并尽可能少地提出。
* 它返回的是你项目中的术语，而不是通用的产品管理样板。
* 其中的每一个决定都是你能记得做出的。没有什么是为了填充版面而编造的。
* 超出范围的部分有实质内容——你拒绝的内容通常是页面上最有用的行。

## 在系统中的位置

`to-spec` 是主构建链中的一个步骤，并且仅在该链的多会话分支上：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它的上游邻居是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，该技能只记录决定而不做决定，以及 [wayfinder](https://aihero.dev/skills-wayfinder)，其完成的地图在此处合并到链中。下游，[to-tickets](https://aihero.dev/skills-to-tickets) 将规格说明切割成 tracer-bullet tickets，供 [implement](https://aihero.dev/skills-implement) 构建。当你不确定哪个技能或流程适合时，[ask-matt](https://www.aihero.dev/skills-ask-matt) 会为你路由。
