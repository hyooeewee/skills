## 功能说明

`wayfinder` 接管一项超出单个代理 [会话](https://www.aihero.dev/ai-coding-dictionary/session) 承载能力的努力 —— 一个你**目的地**已知但**路线**尚不清晰的创意 —— 并将其绘制为问题跟踪器上的一张共享**决策工单**地图，然后逐一解决，直到道路畅通。

它负责规划，而非执行。每个工单都包含一个问题，其解决结果是一个决策，而不是待执行的构建片段。当在有人去构建该事物之前不再有任何决策需要做出时，地图即告完成。这一规则是区分 `wayfinder` 工单与普通实现 [工单](https://www.aihero.dev/ai-coding-dictionary/ticket) 的关键，也是代理最容易违反的规则。当地图清理完毕时，`wayfinder` 会移交任务；它不会继续进入代码编写阶段。

## 何时使用

你通过输入 `/wayfinder` 来调用它 —— 代理不会自行启动它。

它是整个集合中最复杂、最沉重的流程，因此触发条件很严格：付出的努力必须真正超出单个代理会话的承载能力，且通往目的地的路线必须模糊不清。两者的区分很明确：单会话规划使用 `/grill-with-docs`，多会话规划使用 `/wayfinder`。

| 你面前有什么                      | 运行什么                                                                                                                                                 |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 一个范围明确且可以在一次会话中完成的功能        | [grill-me](https://aihero.dev/skills-grill-me)， [grill-with-docs](https://aihero.dev/skills-grill-with-docs)当有代码库时                                   |
| 一个全新的项目，或者跨越多个会话的构建，且路线仍不清晰 | `/wayfinder`                                                                                                                                         |
| 决策已完成的线程                    | [to-spec](https://aihero.dev/skills-to-spec)— 直接跳过地图                                                                                                 |
| 已清理的 wayfinder 地图           | [to-spec](https://aihero.dev/skills-to-spec)，然后 [to-tickets](https://aihero.dev/skills-to-tickets)和 [implement](https://aihero.dev/skills-implement) |
| 一个已经变得太大的现有会话               | 移交给 `/wayfinder`” [handoff](https://aihero.dev/skills-handoff)—                                                                                      |

新项目并非必要条件。\`wayfinder\` 在遗留代码库和半成品的代码库上被例行使用，那里的应用可能更敏锐，因为很多迷雾是“这里已经存在什么”，而不是“我们应该做什么”。

## 前置条件

地图及其工单存在于仓库的问题跟踪器上，因此 `wayfinder` 需要由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 设置的跟踪器连接。该步骤会写入一个“Wayfinding operations”部分，描述地图、其子工单、阻塞边和前沿查询在 GitHub、GitLab 或本地 markdown 中的表达方式。`wayfinder` 通过你 `CLAUDE.md` / `AGENTS.md` 中的指针来解析该文档，而不是通过固定路径；如果没有配置跟踪器，它将回退到本地 markdown 文件。

跟踪器不是装饰品。阻塞是跟踪器自身 UI 中视觉化呈现前沿的东西，而没有原生依赖链接的跟踪器 —— 比如自托管 Gitea —— 会使 \`wayfinder\` 降级为从地图文本推断阻塞项，这虽然可行但需要更密切的监督。

## 地图、迷雾与前沿

**地图**是一个标记为 `wayfinder:map` 的单一问题；其工单是其子问题。它是一个**索引，而非存储** —— 决策存在于唯一的地方，即其工单，而地图只对其进行摘要和链接。会话以低分辨率加载地图，并按需放大查看单个工单，这就是地图能够持续增长而无需每个会话都为其全部历史付费的原因。

有四样东西存在于其中：

* **目的地** —— 到达地图终点看起来是什么样的。命名它是绘制地图的第一步，在此之前任何工单都不存在，因为目的地规定了每个工单所衡量的范围。
* **已做出的决策** —— 每个已关闭的工单一行，每个链接到细节实际所在之处。
* **尚未指定** —— **战争迷雾**。你可以判断即将到来的决策，但还无法清晰表述。迷雾与工单的区别在于，你能否在*现在*精确地陈述问题，而不是能否回答它。解决一个工单会清除它前面的迷雾，并将现在可规范化的内容升级为新的工单。
* **超出范围** —— 被判定在目的地之外的工作。迷雾只会向*目的地*聚集，因此超出范围的工作会被关闭，且永远不会升级。

**前沿**是开放的、未被阻塞且未被占用的工单 —— 已知事物的边缘。会话通过在开始任何工作之前将自己分配给工单来“认领”工单，因此被分配者*就是*认领，并发会话会跳过它。工单在整个过程中都通过名称引用，从不使用裸露的 `#42`；一串工单号在叙述中是不可读的。

## 四种决策工单类型

每个工单都带有 `wayfinder:<type>` 标签，且要么是 **[HITL](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)** —— 与一个代表自己说话的人类协作 —— 要么是 **[AFK](https://www.aihero.dev/ai-coding-dictionary/afk)** —— 仅由代理驱动。HITL 工单只能通过实时交互解决；回答自己 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 问题的代理已经破坏了规则。

| 类型          | 模式   | 何时使用                                        | 由...解决                                                                                                                                                                          |
| ----------- | ---- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `grilling`  | HITL | 默认。该问题可以通过讨论来解决。                            | [grill-me](https://aihero.dev/skills-grilling)加上 [domain-modeling](https://aihero.dev/skills-domain-modeling)，在一个新会话中                                                           |
| `prototype` | HITL | “它应该看起来如何”或“它的行为应该如何” —— 这是一个无法通过讨论解决的问题。   | [prototype](https://aihero.dev/skills-prototype)，并将构建的工件作为资产链接到工单上                                                                                                              |
| `research`  | AFK  | 工作目录之外的事实正在阻塞一个决策。                          | 一个 [research](https://aihero.dev/skills-research) [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent)，在绘制地图时触发，并在 \`research/\<name>\` 分支上并行清理 `research/<name>`分支 |
| `task`      | 两者皆可 | 无需决策，但手动工作会阻塞决策 —— 配置访问权限、注册服务、移动数据以便看到其形状。 | 代理在能够做到的地方独自完成，否则为人类提供一份精确的检查清单                                                                                                                                                 |

`task` 是唯一一种*做*而不是决策的类型，它通过解除决策阻塞而获得一席之地 —— 而非通过交付目的地的一部分。这是在实践中最容易出错的类型：代理将其解释为实施步骤，并在地图内开始编写产品代码。

`research` 是 *每个会话一个工单* 规则的唯一例外。

## 常见问题

**这与 `/grill-with-docs` 有什么不同？我应该从哪个开始？**

**当它要求“目的地”时，是指本次会话的结束还是一切的结束？**

地图已经清理了。为什么我仍然需要 `/to-spec` 和 `/to-tickets` —— `wayfinder` 不是已经写好了规范并创建了工单吗？不是的。Wayfinder 的工单是决策工单，当地图关闭时，它们也都已关闭。剩下的只是一张充满了链接决策的地图，这不是构建计划。[to-spec](https://aihero.dev/skills-to-spec) 将这些链接的决策合并为一个规范 —— `/to-spec #<map_issue>` —— 而 [to-tickets](https://aihero.dev/skills-to-tickets) 将其切分为“追踪弹”式的实施工单。直接将地图接入 [implement](https://aihero.dev/skills-implement) 会跳过合并步骤并丢弃链接的细节。只有当工作量确实很小时，才直接进行实施。人们确实会运行简化的流程并报告其有效；这两个额外的步骤能让你获得一个明确的规范产物，供审阅者或同事阅读，这一点在你越少需要独立完成时越重要。

我的代理在 wayfinder 会话中途开始编写生产代码。这是此技能报告最多的失败案例，背后确实存在一个真实的缺陷。Wayfinder 的“规划，不执行”默认设置可以在地图的 **Notes** 中被覆盖 —— 但这些 Notes 是由代理编写的，因此约束及其豁免条款存在于受约束方拥有的同一个文件中。一位用户看到代理在自己的 Notes 中写下“此地图承载执行”的内容，然后在后续会话中将其作为自己的许可读回，并在实时服务器上构建。目前没有硬性的技能内阻断机制来说“我是指默认行为”。直到出现这种机制为止：阅读任何非你亲自绘制的地图的 Notes，将实施保持在独立的会话中，并将任何看起来像是构建片段的 `wayfinder:task` 视为误操作。

我绘制了 27 个工单，但当我做到第十三个时，其余的变得不再合理。这是一个真实且反复报告的结果，原话来自现场报告。Wayfinder 的默认本能是全面规划，而一张后期的工单依赖于早期的工单所推翻的假设的地图，正是该技能被指责的“瀑布陷阱”。有两件事在抵制它。将地图的范围限定在有限的终点，而不是整个产品 —— 从业者一致报告说，范围限定在一个定义的 epic 的地图表现比那种铺开的“实施 V1”更好，而规划非常庞大的东西本身就不是目标 —— 交付小的增量才是。并且要激进地进行 [prototype](https://www.aihero.dev/ai-coding-dictionary/prototyping)：路线保持当前状态的唯一原因是，在实施依赖它之前，不确定性被廉价的实体产物冲刷掉了。Wayfinder 是“原型最大化”，而不是“规划最大化”。

我可以并行处理多个工单吗？前沿被构建来展示什么是可被领取的，阻塞边存在是为了让并行工作在理论上安全。实际上，一次一个是最安全的默认设置。同时处理两个 grilling 工单的用户会在一个会话中被问到他们刚刚在另一个会话中回答过的问题，因为会话不共享 [context](https://www.aihero.dev/ai-coding-dictionary/context)。在原型工单上也存在一个已知缺口：据报道一个代理构建了三种 UI 变体，自己选择了其中一种，然后关闭了工单 —— 选择权在你，而该技能目前没有明确说明这一点。如果你确实并行运行，请先自己审查依赖图。

我必须使用 GitHub Issues 吗？不 —— 任何问题跟踪器都可以工作。GitHub 是支持最好的路径，因为其原生子问题和阻塞关系是使前沿在没有打开地图的情况下可见的关键；GitLab、Linear、Jira 和本地 markdown 都有被使用。两个诚实的警告。没有原生阻塞功能的跟踪器意味着依赖图是从文本推断出来的，需要手动修正。而本地 markdown 将产物放在你的仓库中，这是不推荐的：将此材料存储在仓库中往往会导致意外持久化。开源维护者面临相反的问题 —— 公共跟踪器被代理生成的规划工单填满 —— 并且无论如何倾向于选择本地 markdown。

grilling 很累人。每个问题都有三段长。这是对 wayfinder 最尖锐的实时投诉，且未解决。一位用户给出的分解是：冗长本身导致了决策疲劳，而长度的剥离移除了*为什么*提出问题，所以随着地图变长，你失去了从决策到决策的链条。这种冗长看起来像是当前 [models](https://www.aihero.dev/ai-coding-dictionary/model) 集合的属性，而不是技能的属性，且没有修复方案落地。流通中的从业者缓解措施是：运行较低的 [reasoning effort](https://www.aihero.dev/ai-coding-dictionary/effort)，并在你的全局 `CLAUDE.md` 中放入一个普通语言的指令。无论怎样，都预期要在这里投入真正的思考 —— wayfinder 要求你思考的量不是缺陷，它是它的主要用途。

**A decision I already closed turned out to be wrong. Do I edit the old ticket or make a new one?**
There is no official guidance, and the agent's instinct is unhelpful: it tends to design around the bad decision rather than challenge it, so you have to steer manually. What does work is telling wayfinder plainly what changed — it updates the map, revises the affected tickets, and comments on already-closed ones. Scope changes mid-map are recoverable. A map you *designed* to change is a scoping smell.

**Where did `decision-mapping` go?**
It is this skill, renamed to `wayfinder` in v1.1 and invoked as `/wayfinder`. "Decision map" was jargon and was also inaccurate, since only one of the four ticket types is really a decision by itself. The reframe gave the skill one coherent vocabulary — destination, fog of war, frontier, the map — instead of an invented term layered on top. The unit kept the "decision" word, though: a **decision ticket** is what a wayfinder ticket is called, precisely to stop people reading it as an implementation ticket.

## 判断是否生效

* 目标在单张工单存在之前就已经被记录并达成一致。
* 每个开放的工单都表现为一个问题。任何读起来像“构建 X”的工单要么是拼写错误，要么属于地图下游。
* 你可以查看你的问题追踪器，而不打开地图就能看到哪些工单是可执行的——这就是边界通过原生阻塞关系呈现自己的方式。
* 一个会话解决一个工单，将答案作为决议评论发布，关闭它，并在地图的“已做出的决策”中留下一条记录。随后它停止工作。
* “尚未指定”部分会随时间收缩。一个升级为工单的迷雾区域会从该部分消失，而不是同时存在于两个地方。
* 当初始的广度优先提问没有发现任何迷雾时，技能会停止并告诉你工作量足够小，可以跳过地图。
* 完成地图的会话会将你引向规范，而不是拉取请求。

## 在系统中的位置

`wayfinder` is a **situational on-ramp**, not the default front door. The grill-led idea → ship chain is still where most work starts; wayfinder is what you climb onto when the idea is too big to hold in one session, and it merges back onto that chain at [to-spec](https://aihero.dev/skills-to-spec), because a cleared map hands off rather than builds.

Underneath, it is mostly other skills wearing wayfinder's scheduling: [grilling](https://aihero.dev/skills-grilling) and [domain-modeling](https://aihero.dev/skills-domain-modeling) resolve the default ticket type, [prototype](https://aihero.dev/skills-prototype) resolves the tickets that talking cannot, and [research](https://aihero.dev/skills-research) runs as a subagent so its reading never lands in your session. [handoff](https://aihero.dev/skills-handoff) is the bridge in and out — into a map from a conversation that outgrew itself, out of one when a side quest appears mid-session. For anything else, [ask-matt](https://aihero.dev/skills-ask-matt) routes over the whole set.
