## 它做什么

`implement` 负责落实已经决定好的工作。你把它指向一个[工单](https://www.aihero.dev/ai-coding-dictionary/ticket)、一份[规格](https://www.aihero.dev/ai-coding-dictionary/spec)，或你在对话中刚商定的计划，它就会编写代码，在接缝处驱动 [tdd](https://aihero.dev/skills-tdd)，边做边做类型检查，最后运行 [code-review](https://aihero.dev/skills-code-review)，并提交到当前分支。

它永远不会重启规划。没有面试，没有澄清轮，没有提出不同方法的建议。上游已定下来的内容即为输入，该技能的全部工作就是把它变成一次提交。这正是它与在一个全新的 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 面前输入“构建这个”的区别——后者往往在构建过程中重新设计工作。

## 何时使用它

你需要自己输入 `/implement` 来调用它，代理不会主动使用它。它自带 `disable-model-invocation: true`，因此其他技能也无法调用它。凡是 [ask-matt](https://aihero.dev/skills-ask-matt) 或 [to-tickets](https://aihero.dev/skills-to-tickets) 说“然后按工单 `/implement`”，那是给你的指令，而不是代理会自动做的事。

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

同一会话的情况值得单独说明，因为技能自身的第一行描述并未涵盖它。`SKILL.md` 说的是“规格或工单”，这会推动 [model](https://www.aihero.dev/ai-coding-dictionary/model) 去寻找一个并不存在的文件。如果计划只存在于对话线程中，请在调用时说明。

## 先决条件

`implement` 会提交到你当前所在的分支。它不会创建分支，也不会询问。开始之前，请确认你已位于想要开展工作的分支上。

如果工单来自 [to-tickets](https://aihero.dev/skills-to-tickets)，[setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 已配置它们所在的跟踪器。`code-review` 读取同一配置，在结束时找到源规格。

## 单次运行会做什么

一次运行包含五个步骤，按顺序：

1. 阅读工单或规格，并厘清接缝。
2. 在预先商定的接缝处驱动 [tdd](https://aihero.dev/skills-tdd)，一次一个红-绿切片。
3. 经常进行类型检查，并在过程中运行单个测试文件。
4. 在最后运行一次完整的测试套件。
5. 运行 [code-review](https://aihero.dev/skills-code-review)，然后提交到当前分支。

一次运行覆盖一个工单。[to-tickets](https://aihero.dev/skills-to-tickets) 生成的工单是示踪弹式的垂直切片，大小适配单个全新的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)，因此预期的节奏是：清除上下文，实现一个工单，提交，再清除。每个工单都是自包含的，所以你可以丢弃前一个工单的上下文。

## 预先商定的接缝

该技能的核心概念是**接缝**，即你在不深入内部的情况下观察行为的公共边界。测试生于接缝。当接缝在代码诞生前就已商定，测试就能经久不衰，你可以在不改动测试的前提下重写其下的实现。

“预先商定”这部分很重要，也是该技能最薄弱的环节。`implement` 内部没有任何机制来商定接缝。`tdd` 才是负责询问的技能，它拒绝在未确认的接缝处编写测试。因此在实践中，商定要么发生在上游的规格里，要么发生在本次运行的首轮对话中。如果哪里都没发生，运行就会退化为“只写代码”，且没有任何警告。在规格中命名接缝，正是为了防止这种情况。

## 常见问题

**它完成了，但我的工单仍然打开，验收标准仍未勾选。**

正确，且符合预期。`implement` 没有完成步骤。它在提交处结束，绝不触及工作项。GitHub Issues 和本地 markdown 跟踪器都是如此，所以这不是跟踪器集成问题。它也不会根据 `code-review` 的发现采取行动，也不会勾选源 issue 上的 `- [ ]` 框。请自行关闭工单并核对验收标准。这在依赖链上最为关键，因为 `to-tickets` 将前沿定义为所有阻塞者均已关闭的工单。如果没有工单被关闭，就永远不会有工单明显地变为解除阻塞。

**我能让它一次处理我所有的工单，或者并行运行多个吗？**

用 `/implement` 做不到：一次调用，一个工单。若想在一次运行中完成整个规格，请使用 [implement-spec](https://aihero.dev/skills-implement-spec)，它会将前沿就绪的每个工单交给各自工作树中的一个 [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent)，随后将结果合并到同一个集成分支。在同一个检出中并行运行多个 `/implement` 会话比不支持更糟糕。一份现场报告描述：一次会话的 `git commit --amend` 落在了另一个会话的提交上，一个 stash 从 `refs/stash` 消失，提交落在了错误分支上，所有这些都在一个下午、三个 issue 中发生。这些会话共享同一个工作目录、同一个索引和同一个 HEAD。用户用 git worktrees 规避此问题，但 `refs/stash` 在 worktrees 间也是共享的，所以仅用 worktrees 无法解决 stash 问题。

**它能创建一个拉取请求来代替提交吗？**

内置不支持。它直接提交到当前分支。许多人觉得这太激进，因为代码在未经验证前就已落地。没有配置标志，也没有 PR 模式。人们通过在调用时覆盖（“提交到分支并打开 PR”）或编辑本地技能副本来绕过。当代理确实撰写 PR 时，[pr](https://aihero.dev/skills-pr) 负责生成其正文。

**`code-review`说它看不到我的改动。**

`code-review` 审查的是 `git diff <fixed-point>...HEAD`，这排除了已暂存和工作区中的更改。`implement` 在提交之前运行它，所以除非已经有一个中间提交，否则该 diff 中没有任何内容可审查。已有多人报告此问题，且两侧都未修复。先提交，再针对你分支出去的那个点进行审查。

此外，一些人根本不想在运行中进行审查，因为由刚写完代码的代理来审查，会偏向自己的方案。在一个全新会话中、针对一个固定点运行 [code-review](https://aihero.dev/skills-code-review) 是一个有效的替代方案。同一个偏见也是为什么该技能在独立的子代理中运行其两个维度。

**一个工单烧掉了 150k tokens。是我用错了吗？**

大概率不是。更可能是工单太大。一次运行包含代码库探索、每个接缝一个红绿循环、完整测试套件和一次审查，所以非平凡工单超过 100k [tokens](https://www.aihero.dev/ai-coding-dictionary/token) 属于正常，而非出错信号。修复在上游。在 [to-tickets](https://aihero.dev/skills-to-tickets) 中把工单切到合适大小，使每个都能装进一个全新窗口。如果单个工单持续超标，请拆分它，而不是提高 [effort](https://www.aihero.dev/ai-coding-dictionary/effort) 等级。

**`/implement #2`在一个全新会话中处理了完全无关的事情。**

代理在上下文中把 `#2` 解析到了另一个编号列表（如 todo 文件或清单），而不是配置的跟踪器。`implement` 现在会从 issue 跟踪器获取传入的引用，在开始前陈述其标题，并在引用模糊时询问。请核对该标题是否与你意指的工单一致；传入 issue URL 或 `owner/repo#2` 可彻底消除歧义。

## 如果它起作用了

* 会话开始时先阅读工单或规格说明，并重述将要构建的内容，而不是问你构建什么。
* 你能在追踪中看到真实的 `tdd` Skill 工具调用，而不仅仅是 diff 中出现的测试。
* 类型检查和单个测试文件在运行过程中会反复执行，而完整测试套件在接近尾声时运行一次。
* 运行会在当前分支上达到一次提交，而无需你提示它继续。
* diff 是一个工单量的变更：贯穿每一层的垂直切片，而不是几个工单混在一起。

## 它在系统中的位置

`implement` 是主链路的构建步骤：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

它的邻居有 [to-tickets](https://aihero.dev/skills-to-tickets)，它负责生成 `implement` 所消费的工单，并声明决定其顺序的阻塞边；[tdd](https://aihero.dev/skills-tdd)，它在每个接缝处内部驱动；以及 [code-review](https://aihero.dev/skills-code-review)，它在提交前运行。它位于规划技能的下游，并信任它们。它不会重新验证交给它的内容形态，因此一个结构糟糕的地图或一个水平分层的工单会按原样被构建。

这种信任正是为什么 [wayfinder](https://aihero.dev/skills-wayfinder) 在 [to-spec](https://aihero.dev/skills-to-spec) 处汇入链路，而不是把它的地图直接环入 `implement`。仅当工作量被证实很小才从地图直达 `implement`。

[ask-matt](https://aihero.dev/skills-ask-matt) 是你不确定自己处于哪个流程时，对整个流程集合的路由器。
