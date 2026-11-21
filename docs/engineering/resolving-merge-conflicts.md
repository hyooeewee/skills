> **Archived.** This skill was removed from the plugin in v1.3.0 and is no longer maintained. Nothing replaces it: the agent works through a merge or rebase conflict without a dedicated skill. The page stays up for reference.

## 它做什么

`resolving-merge-conflicts` 会逐 hunk 处理进行中的 git merge 或 rebase，然后运行项目自身的检查，并以一次提交收尾。

它拒绝将冲突视为文本问题。在处理某个 hunk 之前，它会将每一方追溯到其\*\*【主要来源】(<https://www.aihero.dev/ai-coding-dictionary/primary-source)**（提交信息、PR、原始> issue），因此它是在两个意图之间做选择，而不是在两段文本之间做选择，并且在兼容的地方保留双方的内容。在不兼容的地方，它会选择符合合并既定目标的一方，并点明权衡。它不会发明新行为来掩盖冲突，也永远不会使用 `--abort`。它总是将合并推进到完成的提交。

## 何时使用它

输入 `/resolving-merge-conflicts`，或者当任务匹配时，[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 会自动使用它。

当 git 已经停在它自己无法解决的冲突上时，使用这个技能。它的范围仅限于你眼前的冲突，不涉及冲突两侧的任何其他东西：

| 你的情况                      | 技能                                                           |
| ------------------------- | ------------------------------------------------------------ |
| 合并或变基进行到一半，树中有冲突标记        | 本技能                                                          |
| 合并已完成，但某些东西因你无法理解的原因而表现异常 | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |
| 规划如何拆分工作，让分支间的冲突减少        | Neither: see the parallel-work question below                |

## 主要来源优先于 `ours` 和 `theirs`

该技能的存在是为了防止通过标志位来解决冲突：`--ours`、`--theirs`，或是手动删除看起来不那么重要的代码块，以便标记消失且构建能通过。这种解决方式在语法上可能完美无缺，却仍会无声地丢弃某人有意做出的更改。

你无法保留一个你未曾读过的意图。因此工作从历史（提交、PR、[工单](https://www.aihero.dev/ai-coding-dictionary/ticket)）开始，随后才转到 diff。检查步骤存在的理由相同。该技能会找到仓库自己的[自动化检查](https://www.aihero.dev/ai-coding-dictionary/automated-check)并在提交前运行它们，因为合并是 git 中最容易产生同时满足两个分支却又都无法通过各自测试的代码的地方。

## 常见问题

**Claude Code 自己已经能很好地解决冲突。为什么还需要一个技能？**

额外的价值在于“寻找主要来源”和“运行反馈循环”这两个步骤，否则你每次都得手动提示。未经提示的 agent 通常只会根据 diff 给出一个看似合理的解决方案就停下来。该技能的价值在于它绝不允许 agent 跳过的两个步骤：读取每一方存在的原因，以及事后运行检查。这相较于一个优秀的[模型](https://www.aihero.dev/ai-coding-dictionary/model)只是微小的增益，且这是有意为之。至少有一位读者预言，随着模型改进，这整个技能会变成无操作。

**我是否应该让并行 agent 避开相同文件，从源头上避免冲突？**

多数情况下不用。在并行任务之间把文件分区隔离，代价大于收益，因为 agent 处理合并冲突的能力足够强，权衡并不像看起来那么严峻。值得保留的一条纪律是先做大重构。一个大型重命名在十个分支已经基于它分叉之后才落地，那就会一直代价高昂。

根据用户关于并行 worktree 的报告还有一个注意事项：当同级[会话](https://www.aihero.dev/ai-coding-dictionary/session)各自在自己的树中构建一个工单时，最好由编写变更的会话来执行合并回主线，因为它已经知晓意图。如果由一个 agent 在最后解决所有人的冲突，它就会丢失该技能第 2 步必须重建的[上下文](https://www.aihero.dev/ai-coding-dictionary/context)。

**为什么从不用 `--abort`？**

中止操作会丢弃所有解决工作，下次尝试时你会回到同一个冲突，没有任何变化。这个技能是为“合并将要发生”的情况而写的。如果你决定它不应该发生，那是在调用之前就该做的决定，而不是循环内部的一个分支。

## 如果它起作用了

* 解决过程中，agent 会向你引用提交信息、PR 或 issue，而不仅仅是 diff hunk。
* 每个 hunk 最终都会保留双方的行为，或者附有一条明确的说明，指出丢弃了什么以及为什么。
* 结果中不会出现任何原本不在任一分支上的内容。
* Agent 找到了类型检查、测试和格式化并让它们在提交*前*全部通过，而不是等到你发现有东西坏了之后。
* 你最终会处于一个干净的树结构上，操作已完成，包括多提交 rebase 中剩余的每一个提交。

## 它在系统中的位置

一个随时可用的独立技能，不依赖任何其他技能：它在 git 卡住时开始，在树干净且已提交时结束。它唯一真正的邻居是[diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)，后者在合并已干净解决但合并后的代码表现异常时接管：那是一个诊断问题，而非冲突问题。它位于主流程（idea-to-ship）之外，因此[ask-matt](https://aihero.dev/skills-ask-matt) 是其前后运行内容的地图。
