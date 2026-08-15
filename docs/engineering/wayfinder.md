## 它做什么

`wayfinder` 将超出单个 agent [会话](https://www.aihero.dev/ai-coding-dictionary/session) 承载范围的工作——一个你能说出其**目的地**却还看不清路线的想法——绘制为 issue 跟踪器上一张由**决策工单**构成的共享**地图**，然后逐个解决它们，直到道路清晰。

它做规划，不做执行。每张工单承载的是一个问题，其解决结果是决策，而不是一段待执行的构建内容；当在有人去动手构建之前再无任何可决策之事时，地图便完成了。这一条规则正是 wayfinder 工单与普通实现[工单](https://www.aihero.dev/ai-coding-dictionary/ticket)的分野，也是 agent 最常打破的规则。当地图清晰时，wayfinder 便交接出去；它不会继续进入代码。

## 何时使用它

你通过输入 `/wayfinder` 来调用它——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自行使用它。

它是这套流程中最重、最密集的流程，因此触发条件很窄：任务必须真正超出单个 agent 会话能承载的范围，而且通往目的地的路线必须迷雾重重。两者的分工很清晰：`/grill-with-docs` 负责单会话规划，`/wayfinder` 负责多会话规划。

| 你面前的情况                  | 要运行什么                                                                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 一次会话就能敲定的范围明确的特性        | [grill-me](https://aihero.dev/skills-grill-me)，或 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)当存在代码库时                                 |
| 全新项目，或跨越多个会话的构建，且路线仍不清晰 | `/wayfinder`                                                                                                                                         |
| 决策已经做完了的对话线程            | [to-spec](https://aihero.dev/skills-to-spec)——直接跳过地图                                                                                                 |
| 已理清的 wayfinder 地图       | [to-spec](https://aihero.dev/skills-to-spec)，然后 [to-tickets](https://aihero.dev/skills-to-tickets)和 [implement](https://aihero.dev/skills-implement) |
| 一个已经变得过大的现有会话           | 说“交接给 `/wayfinder`”—— [handoff](https://aihero.dev/skills-handoff)既能桥接进入地图，也能从地图桥接出来                                                                 |

全新项目并非必需。wayfinder 在遗留代码库和半成品代码库上也常规使用，而且在那里可以说更显锋芒，因为大量迷雾是“这里已经是什么情况”而非“我们应该做什么”。

## 先决条件

地图及其工单存放在仓库的 issue 跟踪器上，因此 wayfinder 需要 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 铺设的跟踪器接线。该步骤会写入一个“Wayfinding operations”部分，描述地图、其子工单、阻塞边以及前沿查询在 GitHub、GitLab 或本地 markdown 中如何表达。wayfinder 通过你 `CLAUDE.md` / `AGENTS.md` 中的指针而不是固定路径来解析该文档；如果完全没有配置跟踪器，它会回退到本地 markdown 文件。

跟踪器不是装饰品。阻塞关系正是让前沿在跟踪器自身 UI 中可视化呈现的东西；而没有原生依赖链接的跟踪器——比如自托管的 Gitea——会让 wayfinder 退化为从地图文本中推断阻塞关系，这虽然可行，但需要更密切的监督。

## 地图、迷雾与前沿

**地图**是一个标记为 `wayfinder:map` 的单一 issue；它的工单就是它的子 issue。它是**索引，而非存储**——一个决策恰好只存在于一个地方，也就是它的工单中，地图只对它做摘要并给出链接。会话以低分辨率加载地图，并按需放大到单个工单，这让地图可以不断增长，而不必让每个会话为其全部历史买单。

地图上有四样东西：

* **目的地**——走到这张地图尽头是什么样子。命名目的地是绘图的第一步，在任何工单存在之前就要完成，因为目的地确定了衡量每张工单的范围基准。
* **迄今已做的决策**——每个已关闭工单一行，每行都链接到细节实际所在之处。
* **尚未明确**——**战争迷雾**。你能看出即将到来但还无法清晰表述的决策。区分迷雾与工单的检验标准是：你现在能否准确陈述这个问题，而不是能否回答它。解决一张工单会驱散它前方的迷雾，并把现在能够明确化的内容升级为新的工单。
* **范围之外**——被判定为超出目的地的工作。迷雾只会*朝着*目的地聚拢，因此范围之外的工作会被关闭，永远不会升级。

**前沿**是开放的、未被阻塞的、无人认领的工单——已知的边缘。会话在开始任何工作之前先把工单分配给自己来认领它，因此受让人*就是*认领本身，并发的会话会跳过它。工单始终以名称来引用，绝不使用孤零零的 `#42`；一墙的 issue 编号在叙述中无法阅读。

## 四种决策工单类型

每张工单都带有一个 `wayfinder:<type>` 标签，并且要么是 **[HITL](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)**——与一位亲自发声的人类协作——要么是 **[AFK](https://www.aihero.dev/ai-coding-dictionary/afk)**，由 agent 独自驱动。HITL 工单只能通过实时交流来化解；一个自己回答自己[追问](https://www.aihero.dev/ai-coding-dictionary/grilling)问题的 agent 已经破坏了这条规则。

| 类型          | 模式   | 何时使用                                        | 解决方式                                                                                                                                           |
| ----------- | ---- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `grilling`  | HITL | 默认类型。通过沟通就能解决的问题。                           | [追问](https://aihero.dev/skills-grilling)加上 [domain-modeling](https://aihero.dev/skills-domain-modeling)，在全新的会话中                                |
| `prototype` | HITL | “应该长什么样”或“应该如何表现”——一个靠谈话无法解决的问题。            | [prototype](https://aihero.dev/skills-prototype)，并将构建产物作为资产从工单链接出来                                                                             |
| `research`  | AFK  | 工作目录之外的某个事实正在阻塞决策。                          | 一个 [research](https://aihero.dev/skills-research) [子代理](https://www.aihero.dev/ai-coding-dictionary/subagent)，在绘图时触发，并行燃尽于 `research/<name>`分支 |
| `task`      | 两者皆可 | 没有需要决策的事，但手动工作在阻塞决策——开通权限、注册服务、移动数据以便看清其形态。 | 能由 agent 独立完成就由它完成，否则给人类一份精确的检查清单。                                                                                                             |

`task` 是唯一一种*执行*而非决策的类型，它靠解除决策阻塞来赢得自己的位置——绝不是靠交付目的地的一部分。这是实践中出错最多的类型：agent 会把它当作实现步骤，并开始在地图内部编写产品代码。

research 是“每个会话一张工单”这一规则的唯一例外。

## 常见问题

**这与 `/grill-with-docs` 有什么不同？我应该先从哪个开始？**
区别在于会话数量，而非项目规模。`/grill-with-docs` 是单会话规划；wayfinder 是多会话规划。如果你能在一次对话中容纳全部内容，grilling 是更便宜也更好的工具，wayfinder 在这种情况下确实更慢、更重。社区沉淀下来的简略说法是：只有当工作放不进单个会话时，wayfinder 才有意义。这是 wayfinder 被问到最多的问题，而且一直被问，因为文档描述没有告诉你自己的任务在这条线上处于什么位置——你必须自己判断会话数量。

**当它问“目的地”时，指的是本次会话的结束还是所有一切的结束？**
是整个地图——整张地图的目的地，而不仅仅是初始会话。这个问题读起来有歧义，因为 wayfinder 从定义上就是一个多会话工具，所以以会话为范围的答案永远没有意义。典型的目的地包括：要交接的[规格说明](https://www.aihero.dev/ai-coding-dictionary/spec)、规划开始前要锁定的决策、概念验证，或像数据迁移这样就地完成的变更。

**地图已经清理完毕。为什么我还需要 `/to-spec` 和 `/to-tickets`——难道 wayfinder 不是已经写了 spec 并生成了 ticket 吗？**
不。Wayfinder 的 ticket 是决策 ticket，当地图关闭时它们也都关闭了。剩下的是一个充满关联决策的地图，而不是构建计划。[to-spec](https://aihero.dev/skills-to-spec) 将这些关联决策压缩成一个 spec——`/to-spec #<map_issue>`——而 [to-tickets](https://aihero.dev/skills-to-tickets) 则将其切分为曳光弹式的实现 ticket。将地图直接循环进入 [implement](https://aihero.dev/skills-implement) 会跳过压缩，并把关联的细节丢弃。只有当工作量确实很小时才直接进入实现。确实有人运行了简化流程并报告它有效；这两个额外步骤为你换来一个显式的 spec 产物，审阅者或同事可以阅读，而你越不是单干，这一点就越重要。

**我的代理在 wayfinder 会话中途开始编写生产代码。**
这是该技能被报告最多的失败，背后确实有一个漏洞。Wayfinder 的“只计划，不执行”默认值可以在地图的 **Notes** 中被覆盖——但 Notes 是由代理编写的，所以约束及其豁免存在于被约束方拥有的同一个文件中。一位用户看到代理在其自己的 Notes 中写下“这张地图承载执行”，然后在后续会话中将其作为自己的许可读回，并在线上服务器上继续构建。技能内部没有硬性停止来阻止“我指的是默认值”。在那之前：阅读任何不是你亲自绘制的地图上的 Notes，将实现保留在独立的会话中，并将任何看起来像是构建切片的 `wayfinder:task` 视为类型错误。

**我绘制了 27 个 ticket，但当我做到第十三个时，其余的已经不再有意义。**
这是一个真实且被反复报告的结果，逐字来自现场报告。Wayfinder 的默认本能是全面计划，而后面的 ticket 建立在被前面的 ticket 推翻的假设之上的地图，正是该技能被指责的瀑布陷阱。有两件事可以反击它。将地图的范围限定在一个有界的终点，而不是整个产品——实践者一致报告，限定在一个明确 epic 的地图比一个庞大的“实现 V1”表现更好，而且一开始规划非常大的东西就不是目标——交付小的增量才是。并且积极地 [prototype](https://www.aihero.dev/ai-coding-dictionary/prototyping)：路线保持最新的全部原因在于，在实现依赖它之前，不确定性通过廉价的具体产物被冲刷掉。Wayfinder 是“prototypemaxxing”，而不是“planmaxxing”。

**我可以并行处理几个 ticket 吗？**
前沿（frontier）的设计是为了向你展示哪些是可领取的，阻塞边（blocking edges）的存在使得并行工作在纸面上是安全的。实际上，一次一个更安全的默认。同时处理两个 grilling ticket 的用户会在一个会话中被问到另一个会话刚刚回答过的问题，因为这些会话之间没有共享 [context](https://www.aihero.dev/ai-coding-dictionary/context)。在原型 ticket 上也存在一个已知的缺口：有报告称一个代理构建了三个 UI 变体，自己选择了一个，然后关闭了 ticket——选择权在你手中，而该技能目前没有足够大声地说明这一点。如果你确实并行运行，请先自己审查依赖图。

**我必须使用 GitHub Issues 吗？**
不——任何 issue 跟踪器都可以。GitHub 是支持最好的路径，因为它的原生子 issue 和阻塞关系使得无需打开地图就能看到前沿（frontier）；GitLab、Linear、Jira 和本地 markdown 都有人使用。两个诚实的警告。没有原生阻塞的跟踪器意味着依赖图是从文本中推断出来的，需要手动修正。而本地 markdown 将产物放入你的仓库，这不被推荐：在仓库中存储这些材料往往会导致意外持久化。开源维护者遇到相反的问题——公共跟踪器充满了代理生成的规划 ticket——并且他们往往还是选择本地 markdown。

**grilling 过程令人精疲力竭。每个问题都有三段那么长。**
这是关于 wayfinder 最尖锐的现有抱怨，而且尚未解决。一位用户给出的分解是：冗长本身导致决策疲劳，而长度剥夺了*为什么*要问这个问题的原因，所以随着地图变长，你会失去决策与决策之间的链条。这种冗长看起来更像是当前 [models](https://www.aihero.dev/ai-coding-dictionary/model) 集合的一个属性，而不是该技能本身的属性，而且还没有修复方案落地。流传中的实践者缓解措施：运行较低的 [reasoning effort](https://www.aihero.dev/ai-coding-dictionary/effort)，并在你的全局 `CLAUDE.md` 中放入一条平实语言的指令。无论如何，要做好真正思考的准备——wayfinder 要求你投入的思考量不是缺陷，而是它的大部分意义所在。

**一个我已经关闭的决策后来被证明是错误的。我是编辑旧 ticket 还是创建新的？**
没有官方指导，而且代理的本能并没有帮助：它倾向于围绕错误决策进行设计，而不是挑战它，所以你必须手动引导。真正有效的是直截了当地告诉 wayfinder 发生了什么变化——它会更新地图、修订受影响的 ticket，并在已关闭的 ticket 上添加评论。地图中途的范围变更是可以恢复的。一张你*设计*为会变化的地图是一种范围界定的坏味道（scoping smell）。

**`decision-mapping` 去哪里了？**
它就是本技能，在 v1.1 中更名为 `wayfinder`，并以 `/wayfinder` 调用。“决策地图”是行话，而且也不准确，因为四种 ticket 类型中只有一种本身是真正的决策。重新框架化赋予了该技能一个连贯的词汇表——目的地（destination）、战争迷雾（fog of war）、前沿（frontier）、地图（the map）——而不是在顶层叠加一个发明出来的术语。不过，该单元保留了“decision”这个词：**decision ticket** 就是 wayfinder ticket 的称呼，正是为了阻止人们把它读作实现 ticket。

## 如果它起作用了

* 在任何一个 ticket 存在之前，目的地就已经被写下并达成一致。
* 每个打开的 ticket 读起来都像一个问题。任何读起来像“构建 X”的 ticket 要么是类型错误，要么属于地图的下游。
* 你可以查看你的跟踪器，无需打开地图就能看到哪些 ticket 是可领取的——这就是前沿（frontier）通过原生阻塞自我呈现。
* 一个会话解决一个 ticket，将答案作为解决方案评论发布，关闭它，并在地图的 *Decisions so far* 中留下一行。然后停止。
* \*\*Not yet specified（尚未指定）\*\*会随着时间推移而缩小。一片晋升为 ticket 的迷雾会从该部分消失，而不是同时存在于两个地方。
* 当开场的广度优先追问完全没有发现迷雾时，该技能会停下来，告诉你这项工作足够小，可以跳过地图。
* 完成地图的会话会将你引向一份 spec，而不是一个 pull request。

## 它在系统中的位置

`wayfinder` 是一个**情境式入口（situational on-ramp）**，而不是默认的前门。以追问为主导的 创意 → 交付 链条仍然是大多数工作的起点；当创意太大而无法在一次会话中容纳时，wayfinder 就是你踏上的东西，它会在 [to-spec](https://aihero.dev/skills-to-spec) 处合并回那条链条，因为一张已清理的地图是交接而不是构建。

在底层，它大多是其他技能穿着 wayfinder 的调度外衣：[grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 解决默认的 ticket 类型，[prototype](https://aihero.dev/skills-prototype) 解决那些仅靠谈话无法解决的 ticket，而 [research](https://aihero.dev/skills-research) 作为子代理运行，因此它的阅读永远不会进入你的会话。[handoff](https://aihero.dev/skills-handoff) 是进出的桥梁——从一段超越自身的对话进入地图，或在会话中途出现支线任务时退出。对于其他任何情况，[ask-matt](https://aihero.dev/skills-ask-matt) 会在整个集合上进行路由。
