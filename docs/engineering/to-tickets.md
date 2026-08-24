## 它做什么

`to-tickets` 接收一个计划、一个[规格](https://www.aihero.dev/ai-coding-dictionary/spec)，或你当前的对话，并将其分解为问题跟踪器上的一组\*\*[工单](https://www.aihero.dev/ai-coding-dictionary/ticket)**。每张工单都声明其**阻塞边\*\*：在它开始之前必须完成的其它工单。

Every ticket is a **tracer bullet**: a narrow but complete path through every layer of the change (schema, API, UI, tests) that can be demoed on its own the moment it lands. That is the constraint that makes it behave differently from the obvious way to split work, which is to cut one layer at a time and integrate at the end. It also sizes each ticket to fit in a single fresh [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window), because the thing that will pick the ticket up is a [会话](https://www.aihero.dev/ai-coding-dictionary/session) that has never seen your spec.

## 何时使用它

你通过输入 `/to-tickets` 来调用它。[智能体](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自行调用它。

| 你的情况                                                      | 要运行什么                                                                                                        |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| 你有一个规格工单，且实现过程横跨多个会话                                      | `/to-tickets`，或 `/to-tickets #<spec_issue>`                                                                  |
| 计划只存在于对话中，从未写成文档                                          | `/to-tickets`\`/to-tickets\` 直接读取线程，无需规格。                                                                    |
| 整个变更可以装进一个上下文窗口                                           | [implement](https://aihero.dev/skills-implement)\[implement]\(https\://aihero.dev/skills-implement)，跳过工单。    |
| 还没有任何决定                                                   | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，然后 [to-spec](https://aihero.dev/skills-to-spec) |
| 一个 [wayfinder](https://aihero.dev/skills-wayfinder)地图已经生成 | [to-spec](https://aihero.dev/skills-to-spec)先折叠地图，然后 `/to-tickets`                                           |

`to-tickets` 生成的工单天生就是智能体就绪的。不要对它们运行[分诊](https://www.aihero.dev/skills-triage)。分诊是针对来自他人的工作。

## 先决条件

`to-tickets` 发布到一个跟踪器中，因此 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须已经为这个仓库配置好了一个跟踪器，以及分诊标签词汇表。两种都可以：真实的跟踪器（如 GitHub 或 Linear），或者 `.scratch/` 下的本地 markdown 文件——后者开箱即用。

## 曳光弹，而非分层

**水平**切片交付变更的一层。在每一层都落地之前，什么都不工作，而且每张工单的验收标准必须延伸到另一张工单所拥有的工作。**垂直**切片（即曳光弹）一次性交付穿过所有层的一条细路径，因此它可以单独验证，并拥有它评估的所有内容。

这是人们最常打破的规则，后果有详尽的记录。一个团队运行了一个由层切片（语料库、生产者、聚合器、选择器）组成的 26 张工单堆栈，每个已关闭的工单产生了大约 20 次智能体运行，其中约四分之三需要返工。他们自己的事后分析将每个失败类别都追溯到了水平切片，而不是实现。

在发布任何内容之前，会发生两件事。`to-tickets` 会寻找预重构（“使变更变容易，然后再使容易的变更”这一原则）并将该工作排在前面。然后它将分解呈现为一个编号列表并对你进行测验：粒度是否正确，阻塞边是否真实，是否应该合并或拆分。在你批准之前，没有任何内容到达跟踪器，而那个测验就是反驳的地方。

## 阻塞边

这些边正是该产物的意义所在。根据跟踪器的不同，它们有两种读取方式：

| 跟踪器                  | 边存放在哪里                                                          | 如何处理它们                        |
| -------------------- | --------------------------------------------------------------- | ----------------------------- |
| 本地 markdown          | 每个工单一个文件，位于 `.scratch/<feature>/issues/<NN>-<slug>.md`，编号时阻塞者在前 | 自上而下，手动处理                     |
| 真实跟踪器（GitHub、Linear） | 原生阻塞链接，或跟踪器支持的子问题                                               | 任何其阻塞项都已完成、处于 **前沿**的工单都可以被领取 |

无论哪种方式，边都存在于工单中。媒介只决定是否可以并行对它们采取行动。`to-tickets` 生成工件；运行它（一次一个会话，或一个舰队）是你的工作，而不是技能的工作。

## 大规模重构的例外

一种形状打破了曳光弹规则。**大规模重构**是单一的机械变更（重命名一列、重新输入共享符号），其**爆炸半径**向整个代码库扩散，因此一次编辑会破坏数千个调用点，且没有垂直切片能落地绿色。

`to-tickets` 会将其按\*\*扩展—收缩（expand–contract）\*\*来排序：

* **扩展**：在旧的旁边添加新形式，这样就不会破坏任何东西。
* **迁移**：按爆炸半径（每个包、每个目录）分批将调用点移过去，每个批次一张工单，每张都被扩展阻塞。CI 保持绿色，因为旧形式仍然存在。
* **收缩**：删除旧形式，直到没有调用者剩余，在一张被每个迁移批次阻塞的工单中。

即使分批后仍无法单独保持绿色，它们也会共享一条集成分支，并共同阻塞一张最终的集成并验证（integrate-and-verify）工单。只有在那里才承诺绿色。

## 常见问题

**它为一个三行变更产生了十二张工单。**
过度分解是此技能中最常报告的摩擦点，且在从业者中一致：[模型](https://www.aihero.dev/ai-coding-dictionary/model)默认为原子单位并丢失了使其有意义的分组。测验步骤存在正是为了这一点：要求它合并，它会合并。更深层的答案是工单有一个底线：如果整个变更可以装进一个上下文窗口，你根本不需要这个技能。直接去 [implement](https://aihero.dev/skills-implement)。

**工单一个层一个地出来：所有模式在一个里，所有 API 在另一个里。**
这是垂直切片规则所针对的失败，技能有时仍会产生它。在测验步骤中通过为每张工单问一个问题来捕捉它：当它完成时我可以演示什么？没有答案的工单是一个水平切片。出于这个原因，有些人向每张工单添加一行“演示路径”，并报告它推动模型朝向垂直分解。

**在 GitHub 上，工单没有被创建为规格工单的子问题。**
已知且未修复。这已在十几次运行和多个模型中被报告，[最完整的描述在 issue #554](https://github.com/mattpocock/skills/issues/554)，而且在 Codex 上比在 Claude 上更严重。`gh` 从 v2.94 起原生支持这一点：`gh issue create --parent <n>`，以及事后执行 `gh issue edit <parent> --add-sub-issue <n>`。在跟踪器模板倾向于使用这些命令之前，运行后自己手动接好父链接是可靠的做法。

**"Blocked by"被写入了工单正文而不是真正的阻塞链接。**
同类问题，[在 issue #513 中报告](https://github.com/mattpocock/skills/issues/513)，代理甚至断言 GitHub 根本没有原生阻塞关系。它有：`gh issue create --blocked-by 12,15`。因为阻塞项先发布，它们的数字在创建时总是可用的。正文文本旨在用于没有原生边的跟踪器的后备，而不是默认值。

**本地工单去哪了？v1.1 的说明说有一个根级别的 `tickets.md`。**
它们确实这么说了，那是个 bug：当并行代理写入时，单个共享文件也会发生竞态。本地模式现在在 `.scratch/<feature-slug>/issues/<NN>-<slug>.md` 下为每张工单写入一个文件，按依赖顺序，匹配本地跟踪器模板已经描述的布局。`NN` 前缀是一个真实的工单 ID，所以 `/implement 03` 可以用，而不是重新输入一个长标题。

**它在尝试读取我的规格时一直截断。**
一个非常大的规格可能会超出跟踪器问题干净地回显的内容，而且没有本地副本可以回退，因此代理随后会消耗[工具调用](https://www.aihero.dev/ai-coding-dictionary/tool-call)重新获取块，永远无法到达结尾。不要在 `/to-spec` 和 `/to-tickets` 之间[清除](https://www.aihero.dev/ai-coding-dictionary/clearing)或[压缩](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一个上下文窗口中运行它们，规格根本不需要重新获取。

**验收标准什么都没评分：有些在工作完成前就通过了。**
模板询问标准但没说它们能否失败，所以会发生这种情况。三种形状反复出现：一个在基础提交上已经为真的标准，一个只能通过另一张工单拥有的工作满足的标准，还有一个重述请求而不是从工件推导出的标准。垂直切片防止了大多数情况（一个交付不存在行为的切片在基础提交上根据构造是红色的），但手动检查是值得的。对于每个标准，命名将显示它为假的观察，并确认它在实现者开始从的提交上失败。

**工单已经发布。我实际要如何运行它们？**
该技能止步于工件，没有自动派发模式。派发是手动的：看板看一眼，数出没有未解决阻塞项的工单数量，然后开启同等数量的代理会话。每张工单对应一个全新上下文，会话之间要清理。请注意，[implement](https://aihero.dev/skills-implement) 在结束时并不总能可靠地在 GitHub 或本地 markdown 中关闭或勾选工单，因此工单状态需要你来更新。

## 如果它起作用了

* 每张工单都有对“当这个完成时我可以演示什么？”的回答，并且答案是行为，而不是一层。
* 在发布之前，返回给你的列表是带编号的，且每张工单上都有“被阻塞于”一行。
* 最上面的工单没有阻塞项，可以立即开始。
* 工单正文中不包含文件路径或行号，除了原型产生的代码片段。
* 每张工单读起来都像是全新会话能在你不在场的情况下完成的事项。
* 预重构（如果有发现）排在顺序的前面，而不是混在功能工单中。

## 它在系统中的位置

`to-tickets` 是主构建链中的一个步骤：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

上游是 [to-spec](https://aihero.dev/skills-to-spec)，它交付一个已确定的规范供其切分；将两者保持在同一个连续的上下文窗口中。下游是 [implement](https://aihero.dev/skills-implement)，它为每个新会话构建一个工单，推动 [tdd](https://aihero.dev/skills-tdd) 进行测试，并以 [code-review](https://aihero.dev/skills-code-review) 结束。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指引。
