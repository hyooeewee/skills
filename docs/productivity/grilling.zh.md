## 功能说明

`grilling` 是一个访谈循环，它在任何人采取行动之前对计划、决策或想法进行压力测试。它将主体映射为一个**设计树**——每个决策都分叉出附属于它的决策——并逐个分支地对你进行访谈，直到没有剩余的假设是默默存在的。

它不会一次问一个问题，也不会一次性问完所有问题。每个**轮次**会询问整个**前沿**：那些前提条件已经解决的问题，除此之外别无其他。如果一个问题依赖于另一个问题，它们永远不会出现在同一个轮次中——一个取决于尚未回答的答案的问题属于后续的轮次。你的答案解决问题，前沿向外推移，下一个轮次会询问什么被解除了阻碍。通常，13个问题会被安排在约3个轮次中，而不是13个。

## 何时使用

输入 `/grilling`，或者当任务符合条件时，[代理](https://www.aihero.dev/ai-coding-dictionary/agent)会自行调用它。它是 grilling 系列中唯一由模型触发的[技能](https://www.aihero.dev/ai-coding-dictionary/skill)，这也是你很少手动输入它的原因：通常是你*确实*输入的那个技能正在为你运行它。

直接输入 `/grilling` 会得到纯访谈，除此之外什么也没有。当你想要更多的时候：

| 你拥有什么                       | 使用                                                                                                                              |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| 你不在工作目录中工作                  | [grill-me](https://aihero.dev/skills-grill-me)— 同样的 [会话](https://www.aihero.dev/ai-coding-dictionary/session)，使用一个代理永远不会自行触发的名称 |
| 你在某个工作目录中                   | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)— 同样的会话，并且它会写入 `CONTEXT.md`以及 ADRs（架构决策记录），并在过程中写入                  |
| 一个在一次会话中无法容纳的庞大任务           | [wayfinder](https://aihero.dev/skills-wayfinder)— 它绘制地图并在决策工单中运行 grilling                                                       |
| 一个通过交谈无法确定的问题——某物应该看起来或感觉如何 | [prototype](https://aihero.dev/skills-prototype)— 构建原型版本，然后回来                                                                   |
| 你自己的需要访谈的技能                 | 调用 `/grilling`，而不是编写另一个访谈                                                                                                       |

## 轮次、前沿以及决策者

三个理念支撑着整个技能。

**设计树**是主体的模型：带有附属决策的决策。**前沿**是那些前提条件都已解决的问题的集合——是唯一可以诚实提出的问题。一个**轮次**是完整提出并完整回答的前沿。

在一个轮次中，每个问题都以固定的形式出现：在 `❓` 后面编号并标注标题，然后是正文，最后是代理的推荐答案单独出现在 `➡️` 行上。这就是为什么轮次可以通过数字回答——“1 是，2 第二个选项，3 否，理由如下”——而不是通过引用问题回答。该格式有一个已知的缺陷：推荐答案有时会反驳问题原本的措辞，因此同意推荐答案意味着回答问题的“否”。当发生这种情况时，回答推荐答案并说明原因。

设计的另一部分是事实与决策的划分。事实是技能自己的工作：当前沿问题需要 [环境](https://www.aihero.dev/ai-coding-dictionary/environment) 可以解决的问题时，它会派遣 [子代理](https://www.aihero.dev/ai-coding-dictionary/subagent) 去查找，而不是问你。它不会阻塞于此——只有正在进行的探索下游的问题会等待。决策是你的，它必须等待它们。运行 `grilling` 的代理回答了自己的决策，这是破坏了技能，而不是自由解读。当前沿为空时会话结束，在确认你已达成共同理解之前，它不会根据你同意的内容采取行动。

诚实的限制在于：前沿是代理的判断，而不是计算图。它可能会在一个轮次中放入两个问题，然后才在之后发现一个答案本应该改变另一个。除了告诉它之外，没有任何机制可以防止这种情况，这会在下一轮中重新打开受影响的分支。

## 这里包含什么，以及包装器里包含什么

本页涵盖了该机制。人们最常想要的东西被记录在上一级。

| 问题                                   | 答案所在位置                                                       |
| ------------------------------------ | ------------------------------------------------------------ |
| 树、前沿、轮次、问题格式、事实与决策                   | 这里                                                           |
| 会话应该运行多久，如何处理你无法通过交谈回答的问题，以及如何避免点头附和 | [grill-me](https://aihero.dev/skills-grill-me)               |
| 写入 `CONTEXT.md`，什么变成了 ADR            | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |

## 常见问题

**我可以回到一次问一个问题吗？**
是的，很大一部分观众也是这么做的。将以下内容添加到你的全局 `CLAUDE.md` 中：

```
When grilling, ask one question at a time.
```

基于轮次的默认设置确实存在争议。阅读缓慢、使用第二语言工作、或将顺序格式用作焦点脚手架的从业者都报告说，一次一个的节奏对他们来说更好，而且退出机制是受到支持的，而不是仅仅被容忍。

**`/batch-grill-me` 去哪了？**
进入这个技能。基于轮次的提问短暂地作为一个单独的技能发布，然后移入了 `grilling` 本身，因此所有建立在原语之上的东西——`grill-me`、`grill-with-docs`、`triage`、`wayfinder`——立刻就获得了它。没有 `batch-grill-me` 可安装，也没有单独的顺序技能；上面的 `CLAUDE.md` 行是回到一次一个问题的方法。

**一次询问整个轮次必然会失去我之前的答案本会提出的问题。不是吗？**
这是对轮次设计最常见的异议，而前沿就是对此的答案：一个轮次永远只包含不互相依赖的问题，因此轮次中的任何答案都不能使该轮次中的另一个问题无效。答案仍然会重塑下游的一切——下一轮是重新计算的，而不是预先写好的。你失去的比“一次性问所有问题”所暗示的要少，但也比没有多：见上文前沿的限制。

**它问完了问题就开始构建了。**
确实存在一个确认门控就是为了这种情况：当前沿为空时技能并没有结束，只有当你表示理解已共享时它才结束。较弱且较快的[模型](https://www.aihero.dev/ai-coding-dictionary/model)仍然会破坏它——这在低投入或非前沿模型上报告得最多，它们将“访谈直到共同理解”压缩成了几个问题和一份大纲。如果你的模型会这样，可靠的修复方法是在你自己的 `AGENTS.md` 或 `CLAUDE.md` 中加一行，告诉代理不要在未经许可的情况下实施。

**它回答了自己的问题而不是问我。**
这是运行时的一个错误，而不是预期的行为，这也是为什么技能的文本中将事实与决策分开的原因。它最常出现在另一种技能在 resolve-this-ticket 框架内运行 `grilling` 的时候，周围的阅读任务读起来像是继续移动的许可。同样的约束也是为什么没有异步模式的原因：人们曾要求一种变体来读取 GitHub 问题并发布一份综合决策备忘录，那是一个不同的技能，因为一个没人回答的 grilling 会话产生的是代理的意见，而不是你的意见。

**我可以设置问题数量上限吗？**
不，上限是有意超出范围的。有些计划需要3个问题，有些需要50个；固定的上限要么截断了难处理的案例，要么在简单的案例上显得武断。用自然语言引导是预期的控制方式——告诉它结束，或者停下来接受现有的计划。如果会话运行时间很长，原因通常是范围太大；把工作拆分并逐个进行 grilling。

**我单独安装了 `grill-me` 但什么也没发生。**
`grill-me` 是一个单行技能，其主体内容是“运行一个 `/grilling` 会话”，所以它也需要安装这个技能。`grill-with-docs` 也是一样，它还需要 [domain-modeling](https://aihero.dev/skills-domain-modeling)。安装整个套装可以避免这个问题；选择性安装意味着也要安装原语。

**`grill-with-docs` ran, but it never loaded `grilling`.**
A real and unfixed rough edge, reported across [harnesses](https://www.aihero.dev/ai-coding-dictionary/harness) and models: a skill that names another skill does not reliably cause that skill to load, and `grill-with-docs` names two. The tell is a session that asks everything at once with no recommendations attached — that is the model improvising an interview rather than running this one. Asking the agent directly whether it loaded `grilling` and `domain-modeling` usually recovers it.

## 判断是否生效

* 每一轮以编号列表形式出现，每个问题都有其推荐在单独的 `➡️` 行上，你可以通过数字回答整轮。
* 一轮中的任何问题都不需要先回答同一轮中的另一个问题。
* 后续轮次会提出第一轮无法提出的问题。
* 它会去查找事实——读取文件、派遣子代理——而不是问你本来可以通过查找就能知道的事情。
* 后台运行的研究不会阻塞当前轮次；只有依赖于它的那些问题会等待。
* 它会在结束时停止，并要求你确认双方对理解达成一致，而不是直接开始工作。
* 问题数量保持较高，而轮次数量保持较低。

## 在系统中的位置

## Where it fits`grilling` is a **primitive**, not a step you schedule: the single source of truth for the interview technique, kept in one place so every skill that needs an interview reaches for it instead of inventing one. [grill-me](https://aihero.dev/skills-grill-me) and [grill-with-docs](https://aihero.dev/skills-grill-with-docs) are its two user-invoked front doors, and `grill-with-docs` is where the main build chain begins, ahead of [to-spec](https://www.aihero.dev/skills-to-spec). [wayfinder](https://www.aihero.dev/skills-wayfinder) runs it to resolve decision tickets, [triage](https://www.aihero.dev/skills-triage) to grill a vague report into a workable one, and [improve-codebase-architecture](https://www.aihero.dev/skills-improve-codebase-architecture) to walk the tree once you have picked a candidate to deepen. When you are unsure which entry point fits, [ask-matt](https://www.aihero.dev/skills-ask-matt) routes you.
