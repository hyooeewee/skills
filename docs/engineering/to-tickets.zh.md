## 功能说明

`to-tickets` 接收一个计划、一个 [规范](https://www.aihero.dev/ai-coding-dictionary/spec)，或你当前正在进行的对话，并将其拆分为 issue tracker 上一组 **[工单](https://www.aihero.dev/ai-coding-dictionary/ticket)**。每个工单都会声明其 **阻塞边** —— 必须在它开始之前完成的其它工单。

每个工单都是一枚 **追踪弹**：一条穿过变更每一层的狭窄但完整的路径——schema、API、UI、tests——一旦部署，就可以独立演示。正是这一约束使其行为与显而易见的拆分工作方式不同，即一次切一层并在最后集成。它还调整每个工单的大小以适应单个全新的 [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window)，因为接收工单的是从未见过你的规范的 [会话](https://www.aihero.dev/ai-coding-dictionary/session)。

## 何时使用

你通过输入 `/to-tickets` 来调用它 —— [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自行调用它。

| 所在位置                                                     | 运行什么                                                                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| 你有一个规范问题，且构建涉及多个会话                                       | `/to-tickets`， `/to-tickets #<spec_issue>`                                                                   |
| 计划仅在对话中，从未被记录下来                                          | `/to-tickets`直接读取线程 —— 不需要规范                                                                                 |
| 整个变更都适合在一个上下文窗口中                                         | [implement](https://aihero.dev/skills-implement)—— 跳过工单                                                      |
| 尚未做出任何决定                                                 | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，然后 [to-spec](https://aihero.dev/skills-to-spec) |
| 一个 [wayfinder](https://aihero.dev/skills-wayfinder)地图已清除 | [to-spec](https://aihero.dev/skills-to-spec)首先折叠地图，然后 `/to-tickets`                                          |

由 `to-tickets` 生成的工单从构造上就是 agent 就绪的。不要在它们上面运行 [分类](https://www.aihero.dev/skills-triage) —— 分类是针对来自其他人的工作的。

## 前置条件

`to-tickets` 发布到 tracker，因此 [setup-matt-pocock-skills](https://www.aihero.dev/skills-setup-matt-pocock-skills) 必须已为此仓库配置好一个，以及分类标签词汇表。两种类型都可以：真实的 tracker，如 GitHub 或 Linear，或 `.scratch/` 下的本地 markdown 文件，这是开箱即用的。

## 追踪弹，而非层

**水平** 切片发布变更的一层。在每一层都落地之前，什么都不会工作，而且每个工单的验收标准必须延伸到另一个工单拥有的工作中。**垂直** 切片 —— 追踪弹 —— 一次性穿过所有层的一条细路径，因此它可以单独验证并拥有它评分的所有内容。

这是人们最常打破的规则，后果也有详细记录。一个团队运行了一个按层切片的 26 个工单堆栈——corpus、producer、aggregator、selector——每个已关闭的工单大约得到 20 次代理运行，其中大约四分之三是返工。他们自己的事后分析将每个失败类都追溯到水平切片，而不是实现。

在发布任何内容之前，会发生两件事。`to-tickets` 寻找预重构 —— “让变更变得简单，然后让简单的变更发生” —— 并首先安排这项工作。然后它将分解展示为一个编号列表并对其进行测验：粒度是否正确，阻塞边是否真实，是否应该合并或拆分。在您批准之前，没有任何内容到达 tracker，那个测验就是回击的地方。

## 阻塞边

边是工件的关键点。它们根据 tracker 以两种方式读取：

| Tracker                      | 边位于何处                                                                                                                          | 如何处理它们                           |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | -------------------------------- |
| 本地 markdown                  | 每个工单一个文件中的文本，位于 \`.scratch/\<feature>/issues/\<NN>-\<slug>.md\`，按编号阻塞优先排序 `.scratch/<feature>/issues/<NN>-<slug>.md`，按编号阻塞优先排序 | 从上到下，手动操作                        |
| 真实的 tracker (GitHub, Linear) | 原生阻塞链接，或 tracker 提供子问题的地方                                                                                                      | 任何阻塞已完成的工单都在 **frontier**并且可以被抓取 |

无论哪种方式，边都存在于工单中。介质只决定是否可以并行对它们执行任何操作。`to-tickets` 生成工件；运行它（一次一个会话，或一组）是你的工作，而不是技能的工作。

## 宽重构例外

一种形状打破了追踪弹规则。**宽重构** 是一个单一的机械变更 —— 重命名一列，重新输入一个共享符号 —— 其 **影响范围** 扩散到整个代码库，因此一次编辑会破坏数千个调用点，没有任何垂直切片能变绿。

`to-tickets` 将其序列化为 **扩展-收缩**：

* **扩展** —— 在旧形式旁边添加新形式，这样就不会破坏任何东西。
* **迁移** —— 按影响范围（每个包、每个目录）大小分批移动调用点，每批一个工单，每个都受扩展阻塞。CI 保持绿色，因为旧形式仍然存在。
* **收缩** —— 一旦没有调用者，就在一个受每个迁移批次阻塞的工单中删除旧形式。

即使批次本身也无法保持绿色时，它们共享一个集成分支，并且所有都阻塞一个最终的集成和验证工单。绿色只在那时承诺。

## 常见问题

**它为一个三行变更生成了十二个工单。**
过度分解是此技能报告最多的摩擦，而且这在从业者中是一致的：[模型](https://www.aihero.dev/ai-coding-dictionary/model) 默认为原子单元并丢失了使它们有意义的分组。测验步骤的存在正是为了这一点 —— 要求它合并，它就会。更深层的原因是工单有下限：如果整个变更适合一个上下文窗口，你根本不需要这个技能。直接去 [implement](https://aihero.dev/skills-implement)。

**工单按层出现 —— 所有的 schema 在一个里，所有的 API 在另一个里。**
这是垂直切片规则所针对的失败，而且技能有时仍然会产生它。在测验步骤中通过每个工单问一个问题来发现它：当这个完成时我可以演示什么？没有答案的工单是一个水平切片。出于这个原因，有些人会在每个工单上添加一行“演示路径”，并报告这会引导模型进行垂直分解。

**在 GitHub 上，工单没有作为规范问题的子问题创建。**
已知且未修复。它已在十多次运行和多个模型中被报告，[大多在 issue #554 中完整报告](https://github.com/mattpocock/skills/issues/554)，而且在 Codex 上比在 Claude 上更糟糕。`gh` 从 v2.94 开始原生支持这个：`gh issue create --parent <n>`，以及运行后的 `gh issue edit <parent> --add-sub-issue <n>`。在 tracker 模板偏好这些之前，运行后自己连接父链接是可靠的举动。

**“阻塞于”被写入 issue 正文而不是真正的阻塞链接。**
同一类问题，[在 issue #513 中报告](https://github.com/mattpocock/skills/issues/513)，其中 agent 甚至断言 GitHub 根本没有原生阻塞关系。它有 —— `gh issue create --blocked-by 12,15`。因为阻塞优先发布，它们的数字在创建时间总是可用的。正文文本旨在作为没有原生边的 tracker 的后备，而不是默认值。

**本地工单去哪里？v1.1 说明中提到了根级别的 `tickets.md`。**
它们确实提到了，而且那是一个 bug —— 当并行 agent 向其写入时，单个共享文件也会发生竞争。本地模式现在在每个工单下写入一个文件，位于 `.scratch/<feature-slug>/issues/<NN>-<slug>.md`，按依赖顺序，匹配本地 tracker 模板已经描述的布局。`NN` 前缀是一个真实的工单 ID，所以 `/implement 03` 可以工作，而不必重新输入长标题。

**它尝试读取我的规范时一直被截断。**
非常大的规范可能会超出 tracker issue 干净地服务的内容，而且没有本地副本可以回退 —— 然后 agent 会消耗 [工具调用](https://www.aihero.dev/ai-coding-dictionary/tool-call) 重新获取块，并且永远不会到达终点。不要在 `/to-spec` 和 `/to-tickets` 之间 [清除](https://www.aihero.dev/ai-coding-dictionary/clearing) 或 [压缩](https://www.aihero.dev/ai-coding-dictionary/compaction)。在同一个上下文窗口中运行它们，规范根本不需要被取回。

验收标准没有评分——有些在开始工作之前就通过了。模板要求了标准但没有说明它们是否会失败，所以这种情况发生了。三种模式反复出现：一个在基础提交时就已经为真的标准，一个只能通过另一张票据拥有者完成的工作来满足的标准，以及一个重述请求而不是从产物中推导出的标准。垂直切片防止了大多数这种情况——一个在基础提交时就已经存在的未通过行为是红色的——但手动检查是值得的。对于每个标准，命名那个能证明它是错误的观察，并确认它在实现者开始的提交上失败了。

票据已发布。我实际上该如何运行它们？技能止步于产物，没有自动分发模式。分发是手动的：查看看板，统计没有开放阻碍的票据数量，并打开相应数量的代理会话。每个新上下文一张票据，它们之间清除上下文。请注意，[implement](https://aihero.dev/skills-implement) 在完成时不会可靠地关闭或在 GitHub 或本地 markdown 中勾选票据，所以票据的状态由你来更新。

## 判断是否生效

* 每张票据都有关于“完成时我能演示什么？”的答案——答案应该是行为，而不是层级。
* 列表以编号形式返回给你，每张都有一个“被...阻碍”的行，在任何东西发布之前。
* 最上面的票据没有阻碍，可以立即开始。
* 票据正文中的任何内容都不是文件路径或行号，除了原型生成的片段。
* 每张票据读起来都像是一个新会话可以在你不在房间的情况下完成的任务。
* 如果找到任何预重构，它应该在顺序的前面，而不是混合在功能票据中。

## 在系统中的位置

`to-tickets` 是主构建链中的一个步骤：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

上游是 [to-spec](https://aihero.dev/skills-to-spec)，它提供一份已确定的规范供切片使用——将两者保持在一个不间断的上下文窗口中。下游是 [implement](https://aihero.dev/skills-implement)，它为每个新会话构建一张票据，驱动 [tdd](https://aihero.dev/skills-tdd) 进行测试，并以 [code-review](https://aihero.dev/skills-code-review) 结束。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
