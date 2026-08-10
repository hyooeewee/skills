## 功能说明

`implement` 构建已经确定的工作。你将其指向一个 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)、一个 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，或者你在对话中刚刚达成一致的计划，它会编写代码，在预商定的接口处驱动 [tdd](https://aihero.dev/skills-tdd)，在过程中进行类型检查，在结束时运行 [code-review](https://aihero.dev/skills-code-review)，并提交到当前分支。

它从不重新打开计划。没有面试，没有澄清轮次，没有提出不同的方法。上游达成一致的内容就是输入，该技能的全部工作就是将其转化为一个提交。这就是将其与在新的 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 上输入“build this”的区别，后者会在构建过程中愉快地重新设计工作。

## 何时使用

你通过输入 `/implement` 来调用它——代理不会自动去调用它。它随附 `disable-model-invocation: true`，因此没有任何其他技能可以调用它。无论 [ask-matt](https://www.aihero.dev/skills-ask-matt) 还是 [to-tickets](https://www.aihero.dev/skills-to-tickets) 说“然后对每个票据执行 `/implement`”，那都是给你的指令，而不是代理会自发做的事情。

工作当前所在位置决定了是否使用该技能：

| 工作内容是…              | 使用                                                                                                                                             |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 跟踪器上的票据             | `/implement #42`会话 [会话](https://www.aihero.dev/ai-coding-dictionary/session), [清理](https://www.aihero.dev/ai-coding-dictionary/clearing)票据间上下文 |
| 规范，尚未拆分，且构建跨越会话     | [to-tickets](https://aihero.dev/skills-to-tickets)首先，然后 `/implement`按票据                                                                        |
| 规范，且构建很小            | `/implement`直接针对规范                                                                                                                             |
| 仅存在于您刚刚进行的对话中，且仍然很小 | `/implement`就在那里，在同一窗口中                                                                                                                        |
| 尚未在任何地方写下           | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)， [grill-me](https://aihero.dev/skills-grill-me)如果没有代码库                            |
| 一个具体的特性，你想先测试，没有规范  | [tdd](https://aihero.dev/skills-tdd)直接                                                                                                         |
| 已经构建，你想检查它          | [code-review](https://aihero.dev/skills-code-review)直接                                                                                         |

值得一提的是同会话的情况，因为该技能自己的第一行没有涵盖这一点。`SKILL.md` 说的是“规范或票据”，这会引导 [model](https://www.aihero.dev/ai-coding-dictionary/model) 去寻找一个不存在的文件。如果计划只存在于线程中，请在调用时说明。

## 前置条件

`implement` 会提交到你所在的分支。它不会创建一个分支，也不会询问。在开始之前，请检查你是否处于你想进行工作的分支上。

如果票据来自 [to-tickets](https://www.aihero.dev/skills-to-tickets)，它们所在的跟踪器是由 [setup-matt-pocock-skills](https://www.aihero.dev/skills-setup-matt-pocock-skills) 配置的。`code-review` 会读取相同的配置，以在收尾时找到原始规范。

## 一次运行做什么

一次运行有五个步骤，按顺序：

1. 读取票据或规范并确定接口。
2. 在预商定的接口处驱动 [tdd](https://aihero.dev/skills-tdd)，一次一个红绿切片。
3. 经常类型检查，运行单个测试文件。
4. 运行完整的测试套件一次，在最后。
5. 运行 [code-review](https://aihero.dev/skills-code-review)，然后提交到当前分支。

一次运行覆盖一个票据。[to-tickets](https://www.aihero.dev/skills-to-tickets) 生成的票据是弹道式的垂直切片，大小适合单个新的 [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window)，因此预期的节奏是：清除上下文，实现一个票据，提交，再次清除。每个票据都是独立的，这就是前一个票据的上下文可以丢弃的原因。

## 预商定的接口

该技能运行的理念是 **seam**（接口）：你观察行为的公共边界，而不深入内部。测试位于接口处。在编写任何代码之前商定的接口处工作，是保持测试持久性的原因，因为底层的实现可以在不移动测试的情况下被重写。

“pre-agreed” 这个词正在发挥实际作用，它也是该技能最薄弱的环节。`implement` 内部没有任何东西会商定接口。`tdd` 是提出询问的技能，它拒绝在未确认的接口处编写测试。因此，在实践中，协议要么发生在规范的上游，要么发生在运行的第一次交换中。如果哪里都没有发生，前提条件永远不会触发，运行就会悄无声息地变成“直接写代码”。在规范中命名接口正是阻止这种情况发生的方法。

## 常见问题

**它完成了，但我的票据仍然打开，验收标准仍未勾选。**

正确，且符合预期。`implement` 没有完成步骤。它在提交时结束，永远不会触碰工作项（GitHub Issues 和本地 markdown 跟踪器上已确认），所以这不是跟踪器集成问题。它也不会对 `code-review` 的发现采取行动，也不会在原始问题上勾选 `- [ ]` 框。你自己关闭票据并协调标准。这对依赖链影响最大，因为 `to-tickets` 将前沿定义为所有阻塞都已关闭的票据。如果没有东西被关闭，就永远不会有任何东西变得明显未被阻塞。

**我可以一次性指向所有我的票据，或并行运行几个吗？**

不可以。一次调用，一个票据。批量调度跨票据队列和 [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) 扩散都被反复要求，但两者都不存在。在同一个检出目录中并排运行多个 `/implement` 会话比不支持更糟：一份现场报告描述了一个会话中的 `git commit --amend` 落在了另一个会话的提交上，一个 stash 从 `refs/stash` 中消失，以及提交落在错误的分支上，所有这些都发生在一天下午跨三个问题上。这些会话共享一个工作目录、一个索引和一个 HEAD。Git worktrees 是社区的变通方法，请注意 `refs/stash` 在 worktrees 之间也是共享的，所以仅靠 worktrees 无法修复 stash 的情况。如果你想今天实现并行化，那需要你自己组装。

**它可以打开拉取请求而不是提交吗？**

没有内置该功能。它直接提交到当前分支，这让不少人觉得太急切：代码在有机会验证其是否可用之前就已提交。没有配置标志，也没有 PR 模式。人们可以通过调用时的覆盖指令（“提交到分支并打开 PR”）来绕过它，或者通过编辑他们本地的技能副本。

**`code-review`说它看不到我的更改。**

`code-review` 会审查 `git diff <fixed-point>...HEAD`，这排除了已暂存和工作树中的更改。`implement` 在提交前运行它，因此除非已经存在中间提交，否则该差异中没有可供审查的内容。许多人报告了这个问题，且双方都未修复。先提交，然后针对你分叉的点进行审查。

另外，有些人故意不希望在运行中进行审查，因为审查自己刚写的代码的代理会偏向于自己的解决方案。在新的会话中针对固定点运行 [code-review](https://aihero.dev/skills-code-review) 是一个合理的替代方案，这也是该技能在其两个轴上使用独立子代理的原因。

**一张票据烧掉了 150k tokens。我使用错了吗？**

可能是票据太大而不是技能被误用。一次运行包括代码库探索、每个接口一个红绿循环、完整套件和审查，因此超过 100k [tokens](https://www.aihero.dev/ai-coding-dictionary/token) 的非琐碎票据是正常的，而不是出错的迹象。杠杆在于上游：在 [to-tickets](https://www.aihero.dev/skills-to-tickets) 中正确调整票据大小，使其每个都能适应一个新的窗口。如果一个票据不断超限，请拆分它，而不是提高 [effort](https://www.aihero.dev/ai-coding-dictionary/effort) 级别。

**`/implement #2`在一个新会话中处理了完全无关的东西。**

`#2` 会根据代理能看到的任意编号列表进行解析，在新的会话中，这可能是一个待办文件、检查清单或其他工作列表，而不是配置的跟踪器。这种解析是自信的而不是失败时关闭的，所以错误直到开始后才会变得明显。请传递完整的引用、问题 URL 或 `owner/repo#2`，并在开始前要求它确认标题。

## 判断是否生效

* 会话通过阅读工单或规范并重申它将构建的内容来启动，而不是问你构建什么。
* 你可以在追踪中看到实际的 `/tdd` 调用，而不仅仅是出现在差异中的测试。
* 类型检查和单个测试文件在运行期间反复执行，而完整套件在结束时运行一次。
* 运行到达你当前分支上的一个提交点，而无需你提示它继续。
* 差异是一个工单量级的变化：穿过每一层的垂直切片，而不是几个工单混在一起。

## 在系统中的位置

`implement` 是主链的构建步骤，位于倒数第二位：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它的相邻技能是 [to-tickets](https://aihero.dev/skills-to-tickets)，它生产它消耗的工单并声明决定其顺序的阻塞边；[tdd](https://aihero.dev/skills-tdd)，它在每个接缝处内部驱动它；以及 [code-review](https://aihero.dev/skills-code-review)，它在提交前运行。它位于规划技能的下游并信任它们。它不重新验证它所接收到的内容的结构，因此结构糟糕的地图或水平分层工单是按原样构建的。

这种信任就是为什么 [wayfinder](https://aihero.dev/skills-wayfinder) 在 [to-spec](https://aihero.dev/skills-to-spec) 时汇入链路，而不是直接将其地图导入 `implement`。只有当工作量结果确实很小时，才直接从地图转到 `implement`。

[ask-matt](https://aihero.dev/skills-ask-matt) 是当你不确定你在哪个流程时，整个集合的路由器。
