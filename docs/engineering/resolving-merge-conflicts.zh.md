## 功能说明

`resolving-merge-conflicts` 逐个处理进行中的 git 合并或变基，然后运行项目的自身检查，最后通过提交完成操作。

它拒绝将冲突视为纯文本问题。在处理代码块之前，它会追溯每一方回溯到其\*\*[主要来源](https://www.aihero.dev/ai-coding-dictionary/primary-source)\*\*——即提交信息、PR、原始问题——因此它是在选择两个意图，而不是两个文本块，并在兼容的地方保留两者。如果它们确实不兼容，它会选择与合并既定目标匹配的一方并命名这种权衡。它不会为了掩盖冲突而编造新行为，并且它没有 `--abort` 选项：合并始终会推进到一个已完成的提交。

## 何时使用

类型`/resolving-merge-conflicts`, or the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) reaches for it automatically when a task fits.

当 git 已经在它无法自行解决的冲突处停止时，使用此技能。它的作用范围仅限于你面前的冲突，而不是它两侧的任何内容：

| 你的情况                     | 技能                                                  |
| ------------------------ | --------------------------------------------------- |
| 合并或变基中途，树中有冲突标记          | 这个技能                                                |
| 合并已完成，有某样东西现在行为异常，原因你看不见 | [诊断 bug](https://aihero.dev/skills-diagnosing-bugs) |
| 规划如何切分工作以减少分支碰撞          | 都不需要 —— 请参阅下方的并行工作问题                                |

## 优先使用主要来源而非 `ours` 和 `theirs`

该技能旨在消除的失败模式是通过标志解决：使用 `--ours`、`--theirs`，或手动删除看起来不那么重要的代码块，以便标记消失且构建通过。这种解决方式在语法上可能是完美的，但仍然会默默丢弃某人有意做出的更改。

你无法保留你没有读过的意图。因此工作始于历史记录——提交、PR、[工单](https://www.aihero.dev/ai-coding-dictionary/ticket)——然后才转移到差异（diff）。循环中还有另一个步骤存在是出于同样的原因：该技能会找到仓库自身的[自动化检查](https://www.aihero.dev/ai-coding-dictionary/automated-check)并在提交前运行它们，因为合并是 git 中生成同时满足两个分支并通过两个分支测试的代码的最简单位置。

## 常见问题

**Claude Code 本身已经能很好地解决冲突。为什么这个还需要一个技能？**

增加的价值在于“查找主要来源”和“运行反馈循环”这两个步骤，否则每次都必须手动提示。未提示的代理通常只会从 diff 出发生成一个看似合理的解决方案并就此止步。该技能的价值在于它不会让代理跳过的两个步骤——读取每一方存在的原因，以及在之后运行检查。这在优秀的 [模型](https://www.aihero.dev/ai-coding-dictionary/model) 之上是一个很小的优势，而且本意也是如此：至少有一位读者预测这是一个完整的技能，随着模型的改进，它将变成一个无操作。

**我应该让并行代理避开相同的文件以从一开始就避免冲突吗？**

大多是“不”。在并行任务之间划分文件区域的成本高于节省的成本，因为代理在合并冲突方面已经足够好，以至于权衡看起来并不那么严苛。唯一值得保留的纪律是先进行大型重构。在十个分支分叉出来之后才进行的大型重命名是唯一保持昂贵的案例。

来自关于并行工作树的用户报告的一个注意事项：当兄弟 [会话](https://www.aihero.dev/ai-coding-dictionary/session) 在各自的树中构建工单时，反向合并最好由编写该更改的会话完成，因为它是已经知道意图的一方。最后将所有人的冲突批量处理到一个代理上，会完全丢弃该技能第 2 步必须去重构的 [上下文](https://www.aihero.dev/ai-coding-dictionary/context)。

**为什么从不使用 \`--abort\`？ `--abort`\`/do-work\`？**

Aborting throws away the resolution work and returns you to the same conflict, unchanged, the next time you try. The skill is written for the case where the merge is going to happen. If you have decided it should not happen, that is a decision to make before invoking, not a branch inside the loop.

## 判断是否生效

* 代理在解决冲突时会引用提交信息、PR 或问题，而不仅仅是 diff 代码块。
* 每个 hunk 最终都包含两边的代码行为，或者带有明确说明丢弃了什么以及原因的注释。
* 结果中不出现既不属于 A 分支也不属于 B 分支的内容。
* 类型检查、测试和格式化是在提交*之前*定位并运行通过的，而不是在你发现某样东西坏了之后。
* 操作完成后，你结束在一个干净的树上——包括多提交变基中的每一个剩余提交。

## 在系统中的位置

这是一个随时可用的独立技能，不依赖任何其他技能：它始于 git 停滞，止于树是干净且已提交的。它唯一的真正邻接技能是 [诊断 bug](https://aihero.dev/skills-diagnosing-bugs)，它在合并干净解决但合并后的代码行为异常时接管——这是一个诊断问题，而不是冲突问题。它完全位于主要想法到发布流程之外，因此 [ask-matt](https://aihero.dev/skills-ask-matt) 是在其之前和之后运行的路线图。
