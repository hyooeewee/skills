## 它做什么

`implement` 负责落实已经决定好的工作。你把它指向一个[工单](https://www.aihero.dev/ai-coding-dictionary/ticket)、一份[规格](https://www.aihero.dev/ai-coding-dictionary/spec)，或你在对话中刚商定的计划，它就会编写代码，在接缝处驱动 [tdd](https://aihero.dev/skills-tdd)，边做边做类型检查，最后运行 [code-review](https://aihero.dev/skills-code-review)，并提交到当前分支。

它绝不会重新打开计划。没有访谈，没有澄清环节，也不会提出不同的方法。上游已经确定的内容就是输入，这个技能的全部工作就是把它变成一个提交。这正是它与在全新的[代理](https://www.aihero.dev/ai-coding-dictionary/agent)中输入"构建这个"的区别——后者会在构建工作的同时乐此不疲地重新设计工作。

## 何时使用它

你通过输入 `/implement` 来调用它——代理不会自己主动使用它。它自带 `disable-model-invocation: true`，所以其他技能也无法调用它。无论 [ask-matt](https://aihero.dev/skills-ask-matt) 还是 [to-tickets](https://aihero.dev/skills-to-tickets) 在哪里说"然后按工单 `/implement`"，那都是给你的指令，而不是代理会未经提示就去做的事。

工作目前所处的位置决定了是否该选用这个技能：

| 工作目前是…                     | 选择                                                                                                                                                      |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 跟踪器上的一个工单                  | `/implement #42`，一个工单对应一个 [会话](https://www.aihero.dev/ai-coding-dictionary/session), [清除](https://www.aihero.dev/ai-coding-dictionary/clearing)工单之间的上下文 |
| 一份尚未拆分的规格，并且构建会跨越多个会话      | [to-tickets](https://aihero.dev/skills-to-tickets)先做，然后 `/implement`按工单                                                                                 |
| 一份规格，而且构建规模很小              | `/implement`直接针对该规格                                                                                                                                     |
| 只存在于你刚才的对话中，而且规模还很小        | `/implement`就在那里，在同一个窗口中                                                                                                                                |
| 还没有写在任何地方                  | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，或 [grill-me](https://aihero.dev/skills-grill-me)如果没有代码库                                    |
| 一个具体的、你想以测试优先方式实现的行为，且没有规格 | [tdd](https://aihero.dev/skills-tdd)直接                                                                                                                  |
| 已经构建完成，而你想要检查它             | [code-review](https://aihero.dev/skills-code-review)直接                                                                                                  |

同一个会话的情况值得一提，因为这个技能自己的第一行并没有覆盖它。`SKILL.md` 写的是"规格或工单"，这会引导[模型](https://www.aihero.dev/ai-coding-dictionary/model)去寻找一个并不存在的文件。如果计划只存在于对话线程中，请在调用时说明这一点。

## 先决条件

`implement` 会提交到你当前所在的分支。它不会创建分支，也不会询问。开始之前，请确认你已位于想要开展工作的分支上。

如果这些工单来自 [to-tickets](https://aihero.dev/skills-to-tickets)，那么它们所在的跟踪器是由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 配置的。`code-review` 在收尾时会读取相同的配置，以找到最初的规格。

## 单次运行会做什么

一次运行按顺序包含五个节拍：

1. 阅读工单或规格，并厘清接缝。
2. 在预先商定的接缝处驱动 [tdd](https://aihero.dev/skills-tdd)，一次一个红-绿切片。
3. 经常进行类型检查，并在过程中运行单个测试文件。
4. 在最后运行一次完整的测试套件。
5. 运行 [code-review](https://aihero.dev/skills-code-review)，然后提交到当前分支。

一次运行覆盖一个工单。[to-tickets](https://aihero.dev/skills-to-tickets) 生成的工单是曳光弹式的垂直切片，大小适合单个全新的[上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window)，因此预期的节奏是：清除上下文，实现一个工单，提交，再次清除。每个工单都是自包含的，这正是上一个工单的上下文可以被丢弃的原因。

## 预先商定的接缝

这个技能所依赖的核心概念是**接缝**：你在不深入内部的情况下观察行为的公开边界。测试位于接缝处。在编写任何代码之前先商定接缝，并在该接缝处工作，这能保证测试的持久性，因为底层的实现可以被重写，而测试无需变动。

"预先商定"这个词承担着实际工作，同时也是这个技能最薄弱的环节。`implement` 内部没有任何东西会商定接缝。`tdd` 是那个会提出询问的技能，它拒绝在未经确认的接缝处编写测试。因此，在实际操作中，商定要么发生在上游的规格中，要么发生在运行的第一次交流中。如果哪里都没有发生，前置条件就永远不会触发，运行就会悄悄变成"只管写代码"。在规格中明确命名接缝，正是阻止这种情况发生的办法。

## 常见问题

**它完成了，但我的工单仍然打开，验收标准仍未勾选。**

正确，而且这是预期行为。`implement` 没有完成步骤。它止步于提交，绝不会触碰工作项——这一点在 GitHub Issues 和本地 markdown 跟踪器上均已确认，所以这不是跟踪器集成问题。它也不会对 `code-review` 产出的发现采取行动，更不会勾选原始 issue 上的 `- [ ]` 复选框。你需要自己关闭工单并核对标准。在依赖链上，这一点最令人头疼，因为 `to-tickets` 将前沿定义为所有阻塞项都已关闭的工单。如果没有任何工单被关闭，就永远不会有工单看起来被解除阻塞。

**我能让它一次处理我所有的工单，或者并行运行多个吗？**

不行。一次调用只处理一个工单。跨工单队列的批量调度和 [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) 扇出都被反复要求过，但两者都不存在。在同一个检出中并排运行多个 `/implement` 会话比不受支持还要糟糕：一份现场报告描述了在三个 issue 上，一个下午之内，一个会话中的 `git commit --amend` 落在了另一个会话的提交上，一个 stash 从 `refs/stash` 中消失，还有提交落在了错误的分支上。这些会话共享同一个工作目录、同一个索引和同一个 HEAD。Git worktree 是社区的变通方案，但请注意 `refs/stash` 在不同 worktree 之间也是共享的，所以仅靠 worktree 并不能解决 stash 的问题。如果你今天想要并行，就得自己组装。

**它能创建一个拉取请求来代替提交吗？**

没有内置此功能。它会直接提交到当前分支，这让一些人觉得太过急切：代码在他们有机会验证其能否工作之前就已经落地。没有配置标志，也没有 PR 模式。人们会在调用时覆盖这一行为（"提交到分支并打开 PR"），或者通过编辑本地技能副本。

**`code-review`说它看不到我的改动。**

`code-review` 审查的是 `git diff <fixed-point>...HEAD`，这排除了已暂存和工作区中的更改。`implement` 在提交之前运行它，所以除非已经有一个中间提交，否则该 diff 中没有任何内容可审查。已有多人报告此问题，且两侧都未修复。先提交，再针对你分支出去的那个点进行审查。

另外，有些人刻意根本不希望在运行中包含审查，因为代理审查自己刚写的代码时，会偏向自己的解决方案。在一个全新的会话中，针对一个固定点运行 [code-review](https://aihero.dev/skills-code-review)，是一种合理的替代方案，也正是该技能在独立的子代理中运行其两个轴的原因。

**一个工单烧掉了 150k tokens。是我用错了吗？**

很可能是工单太大，而不是技能被误用。一次运行会进行代码库探索、每个接缝一个红绿循环、完整测试套件和一次审查，所以一个不小的工单超过 100k [tokens](https://www.aihero.dev/ai-coding-dictionary/token) 是正常的，而不是出了问题的迹象。杠杆在上游：在 [to-tickets](https://aihero.dev/skills-to-tickets) 中把工单调整到合适的大小，让每个工单都能放进一个全新的窗口。如果某个工单总是爆掉，就拆分它，而不是提高 [effort](https://www.aihero.dev/ai-coding-dictionary/effort) 等级。

**`/implement #2`在一个全新会话中处理了完全无关的事情。**

`#2` 是相对于智能体所能看到的任何编号列表来解析的，在一个新会话中，这个列表可能是待办文件、清单或其他工作列表，而不是已配置的跟踪器。这种解析是自信式的，而不是失败关闭式的，因此错误直到开始后才会变得明显。请传递完整引用、issue URL 或 `owner/repo#2`，并在开始前让它确认标题无误。

## 如果它起作用了

* 会话开始时先阅读工单或规格说明，并重述将要构建的内容，而不是问你构建什么。
* 你可以在跟踪日志中看到实际的 `/tdd` 调用，而不仅仅是在 diff 中出现测试。
* 类型检查和单个测试文件在运行过程中会反复执行，而完整测试套件在接近尾声时运行一次。
* 运行会在当前分支上达到一次提交，而无需你提示它继续。
* diff 是一个工单量的变更：贯穿每一层的垂直切片，而不是几个工单混在一起。

## 它在系统中的位置

`implement` 是主链的构建步骤，倒数第二个：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它的邻居有 [to-tickets](https://aihero.dev/skills-to-tickets)，它负责生成 `implement` 所消费的工单，并声明决定其顺序的阻塞边；[tdd](https://aihero.dev/skills-tdd)，它在每个接缝处内部驱动；以及 [code-review](https://aihero.dev/skills-code-review)，它在提交前运行。它位于规划技能的下游，并信任它们。它不会重新验证交给它的内容形态，因此一个结构糟糕的地图或一个水平分层的工单会按原样被构建。

正是这种信任，使得 [wayfinder](https://aihero.dev/skills-wayfinder) 在 [to-spec](https://aihero.dev/skills-to-spec) 处接入主链，而不是将其地图直接循环进 `implement`。只有当工作量确实很小的时候，才从地图直接进入 `implement`。

[ask-matt](https://aihero.dev/skills-ask-matt) 是你不确定自己处于哪个流程时，对整个流程集合的路由器。
