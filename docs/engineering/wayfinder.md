## 它做什么

`wayfinder` 接管了一个单次 agent [会话](https://www.aihero.dev/ai-coding-dictionary/session) 无法承载的任务：一个你能命名其**目的地**但还看不见其路线的想法，将其绘制为你在 issue 跟踪器上共享的由**决策工单**组成的**地图**，然后逐一解决这些工单，直到路线清晰。

它做规划，不做执行。每张工单承载的是一个问题，其解决结果是决策，而不是一段待执行的构建内容；当在有人去动手构建之前再无任何可决策之事时，地图便完成了。这一条规则正是 wayfinder 工单与普通实现[工单](https://www.aihero.dev/ai-coding-dictionary/ticket)的分野，也是 agent 最常打破的规则。当地图清晰时，wayfinder 便交接出去；它不会继续进入代码。

## 何时使用它

你通过输入 `/wayfinder` 来调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自行调用它。

它是这套流程中最重、最密集的流程，因此触发条件很窄：任务必须真正超出单个 agent 会话能承载的范围，而且通往目的地的路线必须迷雾重重。两者的分工很清晰：`/grill-with-docs` 负责单会话规划，`/wayfinder` 负责多会话规划。

| 你面前的情况                  | 要运行什么                                                                                                                                                |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 一次会话就能敲定的范围明确的特性        | [grill-me](https://aihero.dev/skills-grill-me)，或 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)当存在代码库时                                 |
| 全新项目，或跨越多个会话的构建，且路线仍不清晰 | `/wayfinder`                                                                                                                                         |
| 决策已经做完了的对话线程            | [to-spec](https://aihero.dev/skills-to-spec)直接跳过地图                                                                                                   |
| 已理清的 wayfinder 地图       | [to-spec](https://aihero.dev/skills-to-spec)，然后 [to-tickets](https://aihero.dev/skills-to-tickets)和 [implement](https://aihero.dev/skills-implement) |
| 一个已经变得过大的现有会话           | 说“交接给 `/wayfinder`" ([handoff](https://aihero.dev/skills-handoff)既能接入地图也能退出地图                                                                        |

全新项目并非必需。wayfinder 在遗留代码库和半成品代码库上也常规使用，而且在那里可以说更显锋芒，因为大量迷雾是“这里已经是什么情况”而非“我们应该做什么”。

## 先决条件

地图及其工单存放在仓库的 issue 跟踪器上，因此 wayfinder 需要 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 铺设的跟踪器接线。该步骤会写入一个“Wayfinding operations”部分，描述地图、其子工单、阻塞边以及前沿查询在 GitHub、GitLab 或本地 markdown 中如何表达。wayfinder 通过你 `CLAUDE.md` / `AGENTS.md` 中的指针而不是固定路径来解析该文档；如果完全没有配置跟踪器，它会回退到本地 markdown 文件。

The tracker is not decoration. Blocking is what renders the frontier visually in the tracker's own UI, and a tracker without native dependency links (a self-hosted Gitea, say) degrades wayfinder to inferring blockers from the map text, which works but needs closer supervision.

## 地图、迷雾与前沿

**地图**是一个标记为 `wayfinder:map` 的单个 issue；它的工单是其子 issue。它是一个**索引，而非存储**：决策只存在于唯一的地方，即其工单，而地图仅对其进行 gist 并建立链接。会话以低分辨率加载地图，并根据需要放大查看单个工单，这就是地图能不断增长而无需每个会话都为其完整历史买单的原因。

地图上有四样东西：

* **目的地**：到达地图终点意味着什么。命名它是绘图的第一步，在任何工单存在之前，因为目的地决定了每个工单的衡量标准。
* **已做出的决策**：每个已关闭的工单一行，每一行都链接到细节实际所在的位置。
* **尚未指定**：**战争迷雾**。你能感觉到决策即将到来但还无法清晰表述的决策。判断迷雾与工单的标准是，你现在能否精确地陈述问题，而不是能否回答它。解决工单会清除其前方的迷雾，并将现在可规格化的内容升级为新的工单。
* **超出范围**：被判定在目的地之外的工作。迷雾只会在**朝向**目的地的方向聚集，因此超出范围的工作会被关闭，永远不会升级。

**前沿**是开放的、未被阻塞且未被认领的工单（已知区域的边缘）。会话在开始工作之前通过将工单分配给自己来认领它，因此受让人*就是*认领，并发会话会跳过它。整个过程中工单都以其名称指代，绝不用裸露的 `#42`；一串 issue 编号在叙述中是无法辨认的。

## 四种决策工单类型

每张工单都带有 `wayfinder:<type>` 标签，要么是\*\*[HITL](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)**（与为自己发声的人类协作），要么是**[AFK](https://www.aihero.dev/ai-coding-dictionary/afk)\*\*，完全由 agent 驱动。HITL 工单只能通过实时交换解决；如果 agent 回答自己的 [追问](https://www.aihero.dev/ai-coding-dictionary/grilling) 问题，就破坏了规则。

| 类型          | 模式   | 何时使用                                            | 解决方式                                                                                                                                           |
| ----------- | ---- | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `grilling`  | HITL | 默认类型。通过沟通就能解决的问题。                               | [追问](https://aihero.dev/skills-grilling)加上 [domain-modeling](https://aihero.dev/skills-domain-modeling)，在全新的会话中                                |
| `prototype` | HITL | 默认类型。通过沟通就能解决的问题。                               | [prototype](https://aihero.dev/skills-prototype)，并将构建产物作为资产从工单链接出来                                                                             |
| `research`  | AFK  | 工作目录之外的某个事实正在阻塞决策。                              | 一个 [research](https://aihero.dev/skills-research) [子代理](https://www.aihero.dev/ai-coding-dictionary/subagent)，在绘图时触发，并行燃尽于 `research/<name>`分支 |
| `task`      | 两者皆可 | 没有需要决策的内容，但手动工作会阻塞决策，例如配置访问权限、注册服务或移动数据以便看清其形态。 | 能由 agent 独立完成就由它完成，否则给人类一份精确的检查清单。                                                                                                             |

`task` 是唯一一种*做*而非决策的类型，它通过解除决策阻塞来获得其位置，而不是通过交付目的地的一部分。这是实践中最容易出错的一种类型：agent 将其理解为实现步骤，并开始在地图内编写产品代码。

research 是“每个会话一张工单”这一规则的唯一例外。

## 常见问题

**这与 `/grill-with-docs` 有什么不同？我应该从哪个开始？**
会话数量，而非项目大小。`/grill-with-docs` 是单会话规划；wayfinder 是多会话规划。如果你能在一次对话中涵盖整个事情，grilling 是更便宜且更好的工具，而在这种情况下 wayfinder 确实更慢、更密集。社区形成的简写共识是：如果工作无法放入单个会话，wayfinder 才有意义。这是被问得最多的问题，而且不断被问是因为描述没有告诉你自己的任务处于哪条线上。你必须自己判断会话数量。

**当它询问“目的地”时，是指本次会话的终点还是一切的终点？**
整个地图。这意味着整个地图的目的地，而不仅仅是初始会话。这个问题读起来有歧义，因为 wayfinder 从定义上讲是多会话工具，所以会话范围的答案从未讲得通。典型的目的地是用于交接的 [规范](https://www.aihero.dev/ai-coding-dictionary/spec)、规划开始前要锁定的决策、概念验证，或者像数据迁移这样的就地更改。

**地图已清理。wayfinder 不是已经写好规范并创建工单了吗？为什么我仍然需要 `/to-spec` 和 `/to-tickets`？**
没有。wayfinder 的工单是决策工单，当地图关闭时，它们也都已关闭。剩下的就是一个充满链接决策的地图，这不是构建计划。[to-spec](https://aihero.dev/skills-to-spec) 将那些链接的决策合并为一个规范 (`/to-spec #<map_issue>`)，而 [to-tickets](https://aihero.dev/skills-to-tickets) 将其切分为弹道子弹式的实现工单。直接将地图循环进入 [implement](https://aihero.dev/skills-implement) 会跳过合并并丢弃链接的细节。只有在工作量真正很小的情况下，才直接进入实现。人们确实会运行简化的管道并报告其有效；那两个额外的步骤为你提供了一份明确的规范产物，供审查者或同事阅读，这在你独自工作时尤为重要。

**我的 agent 在 wayfinder 会话中途开始编写生产代码。**
这是此技能最常被报告的失败，其背后确实存在一个漏洞。wayfinder 的“规划，不做”默认设置可以在地图的 **Notes** 中被覆盖，但 Notes 是由 agent 编写的，所以约束及其豁免权存在于受限方拥有的同一个文件中。一位用户看到 agent 将“this map carries execution”写入其自己的 Notes，然后在后续会话中将其读回作为自己的许可，并在实时服务器上构建。目前没有硬性的技能内停止机制来说“我的意思是默认设置”。在出现之前：阅读任何不是你绘制的地图上的 Notes，将实现保持在独立的会话中，并将任何看起来像构建片段的 `wayfinder:task` 视为拼写错误。

我绘制了 27 张工单，等到第 13 张时，剩下的就不再有意义了。这是一个真实且反复出现的后果，直接摘自现场报告。Wayfinder 的默认本能是全面规划，而一张后期的工单依赖于早期工单所推翻的假设的地图，正是该技能被指责的瀑布陷阱。有两件事对此构成阻碍。将地图的范围限定在有限的终点，而不是整个产品。从业者一致报告，限定在单一定义的史诗内的地图比那种 sprawling 的“实现 V1”表现更好，规划非常宏大的东西根本不是最初的目标：交付小的增量才是。并且要激进地 [原型](https://www.aihero.dev/ai-coding-dictionary/prototyping)：路线保持更新的唯一原因是在实现依赖它之前，不确定性被廉价的具体化产物冲刷掉了。Wayfinder 是“prototypemaxxing”，而不是“planmaxxing”。

我可以同时处理多个工单吗？前沿（Frontier）的构建是为了向你展示什么是可执行的，而阻塞边（blocking edges）的存在是为了让并行工作在纸面上是安全的。实际上，一次处理一个是更安全的默认选择。同时处理两个追问工单的用户会在一个会话中被问到他们刚在另一个会话中回答过的问题，因为会话之间不共享 [上下文](https://www.aihero.dev/ai-coding-dictionary/context)。在原型工单上也存在一个已知的漏洞：据报道，一个代理构建了三种 UI 变体，自己选择了一种，然后关闭了工单。选择权由你决定，而且该技能目前还没有大声说清楚这一点。如果你确实要并行运行，请先自行检查依赖图。

我必须使用 GitHub Issues 吗？不。任何问题跟踪器都可以工作。GitHub 是支持最好的路径，因为其原生的子问题和阻塞关系正是让前沿无需打开地图就变得可见的原因；GitLab、Linear、Jira 和本地 markdown 都有在使用。两个诚实的注意事项。一个没有原生阻塞功能的跟踪器意味着依赖图是从文本推断出来的，需要手动修正。本地 markdown 将产物放在你的仓库中，这并不推荐：将此材料存储在仓库中 tends to lead to accidental persistence（倾向于导致意外持久化）。开源维护者遇到了相反的问题（公共跟踪器被代理生成的规划工单填满）并且倾向于无论如何都选择本地 markdown。

追问令人精疲力竭。每个问题都有三段那么长。这是关于 wayfinder 最尖锐的实时抱怨，而且尚未解决。一位用户给出的分解是：冗长本身导致了决策疲劳，而且长度剥离了提问的 *原因*，所以随着地图变长，你失去了从决策到决策的链条。这种冗长看起来像是当前模型集的属性，而不是该技能的属性，而且还没有修复方案。正在流传的从业者缓解措施：运行较低的 [推理强度](https://www.aihero.dev/ai-coding-dictionary/effort)，并在你的全局 `CLAUDE.md` 中放入一条自然语言指令。无论怎样，都要预期在这里投入真正的思考，因为 wayfinder 要求你思考的量并不是缺陷，而是它大部分的作用所在。

我已经关闭的一个决定结果是错误的。我应该编辑旧工单还是创建一个新工单？没有官方指导，而且代理的本能是无益的：它倾向于围绕错误的决定进行设计，而不是挑战它，所以你必须手动引导。什么有效的是明确告诉 wayfinder 发生了什么变化；它会更新地图，修改受影响的工单，并评论已经关闭的工单。地图中途的范围变更是可以恢复的。一张你 *设计* 用于改变的地图是一个范围异味。

`decision-mapping` 去哪了？它就是这个技能，在 v1.1 中重命名为 `wayfinder` 并作为 `/wayfinder` 调用。“决策地图”是行话，而且也不准确，因为四种工单类型中只有一种是真正独立的决策。重构给了该技能一个连贯的词汇（终点、战争迷雾、前沿、地图），而不是一个叠加在上面的发明术语。尽管如此，该单元保留了“决策”这个词：**决策工单（decision ticket）** 是 wayfinder 工单的名称，正是为了阻止人们将其读作实现工单。

## 如果它起作用了

* 在任何一个 ticket 存在之前，目的地就已经被写下并达成一致。
* 每个打开的 ticket 读起来都像一个问题。任何读起来像“构建 X”的 ticket 要么是类型错误，要么属于地图的下游。
* 你可以查看你的跟踪器，看看哪些工单是可执行的，而无需打开地图，因为这就是前沿通过原生阻塞关系自我呈现。
* 一个会话解决一个 ticket，将答案作为解决方案评论发布，关闭它，并在地图的 *Decisions so far* 中留下一行。然后停止。
* \*\*Not yet specified（尚未指定）\*\*会随着时间推移而缩小。一片晋升为 ticket 的迷雾会从该部分消失，而不是同时存在于两个地方。
* 当开场的广度优先追问完全没有发现迷雾时，该技能会停下来，告诉你这项工作足够小，可以跳过地图。
* 完成地图的会话会将你引向一份 spec，而不是一个 pull request。

## 它在系统中的位置

`wayfinder` 是一个**情境式入口（situational on-ramp）**，而不是默认的前门。以追问为主导的 创意 → 交付 链条仍然是大多数工作的起点；当创意太大而无法在一次会话中容纳时，wayfinder 就是你踏上的东西，它会在 [to-spec](https://aihero.dev/skills-to-spec) 处合并回那条链条，因为一张已清理的地图是交接而不是构建。

在底层，它主要是其他技能使用 wayfinder 的调度：[grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 解析默认的工单类型，[prototype](https://aihero.dev/skills-prototype) 解析那些无法通过交谈解决的工单，而 [research](https://aihero.dev/skills-research) 作为子代理运行，这样它的阅读内容永远不会落在你的会话中。[handoff](https://aihero.dev/skills-handoff) 是进出的桥梁：从一段超出了自身范围的对话进入地图，从中段出现支线任务时退出。对于其他任何事情，[ask-matt](https://aihero.dev/skills-ask-matt) 路由过整个集合。
