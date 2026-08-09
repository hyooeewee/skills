## 功能说明

`grill-me` 拿走一个**模糊的想法**并对你进行面试，直到你能承诺它。你不需要一个完善的计划就开始——生成一个计划正是 [session](https://www.aihero.dev/ai-coding-dictionary/session) 的目的。它按**轮次**提问：每一轮都是整个**前沿**——即你已经解决了所有前提问题的每一个问题——所以你永远不会被问到依赖于它尚未听到的答案的问题。

它是\*\*[无状态的](https://www.aihero.dev/ai-coding-dictionary/stateless)\*\*。它不写入文件，也不留下工作空间。它留下的唯一东西是脑海中想法的更清晰的版本。

## 何时使用

You invoke this by typing `/grill-me` — the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own. Start it in a **fresh conversation**, not on top of a plan you already had an agent write.

一旦你有一个值得认真对待的想法——一个功能、一个产品方向、一个商业决策、一段文字——以及在你弄清楚它涉及什么之前很久，就使用它。模糊不是一个等待的理由；它是该会话吞噬的东西。如果你已经能精确地指定该事物，你就不需要对其进行 grilled。

你想选择的三种 grilled 技能中的哪一种取决于你面前是什么：

* **任何东西，任何地方** — `grill-me`。它不需要仓库也不写入文件，且主题不一定是代码。
* **一个要对齐的代码库** — [grill-with-docs](https://aihero.dev/skills-grill-with-docs)。同样的面试，但是 [有状态的](https://www.aihero.dev/ai-coding-dictionary/stateful)：它读取你的代码并将所学内容保存在 `CONTEXT.md` 和 ADRs 中。
* **对于一次会话来说太大** — [wayfinder](https://aihero.dev/skills-wayfinder)。它将工作映射为地图并在其中运行 grilled 会话。

关闭 [计划模式](https://www.aihero.dev/ai-coding-dictionary/agent-mode)。计划模式让代理倾向于急于生成一个计划，这与保持探究相反。

## 这是一场对话，而不是面试

该技能提出问题，但**你**拥有范围。这是人们容易遗漏的部分，它将把想法转化为决策的会话与产生自信废话的会话区分开来。

失败模式是**被动**——回答了四十个问题“同意，同意，同意”，然后拿着代理写的计划点头认可。它感觉很有效率，因为它很长。实际上什么都没决定，而且结果带有一种它没有赢得的确定性。

积极意味着引导。对低于你所需要精度的质问提出反对。当范围偏离时说出来。回答“我不知道”并把它当真。这项技能是旨在辅助工程师，而不是取代工程师：输出的内容追踪的是你回答的质量，而不是提问的数量。

相反的错误是真实但较少见的——在面试中停留太久以至于从未到达代码编写阶段。

## 可 grilled 和不可 grilled

有些问题可以通过交谈回答。其他的不能，无论多少 grilled 都无法让你到达那里。

“一份长表单还是三页？”和“这种交互应该感觉如何？”是**不可 grilled** 的——它们需要一些东西来反应。当你遇到一个时，停止 grilled。使用 [prototype](https://aihero.dev/skills-prototype) 构建一个一次性版本，查看它，然后回来用一行回答。

通过谈话解决一个不可 grilled 的问题是会话膨胀的地方。代理不断改写措辞，你不断猜测，范围不断增长以填补不确定性。

## 判断是否生效

* 你对某事持不同意见。如果没有来自你的反驳，这个会话是你不需要的。
* 问题在几轮中到达，而不是一次长滴答，后来的几轮显然建立在你之前所说内容的基础上。
* 你最终到达了一个你没有预料到的地方，因为一个问题揭示了一个你一直隐含做出的决定。
* 最后，你可以向当时不在场的人为每个选择进行辩护。

## 常见问题

**我应该期望多少问题，以及如何知道何时结束？**
计算轮次，而不是问题。四个轮次中包含四十六个问题是普通会话。当前沿为空时结束——访问了每个分支，没有留下被静默假设的东西。

**它问我了两百个问题。出什么问题了？**
通常范围太大了。让代理先将工作分解为更小的部分，然后 grilled 每一部分。非常长的会话也会漂移到\*\*[愚蠢区域](https://www.aihero.dev/ai-coding-dictionary/smart-zone)\*\*，其中[上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window)足够满，导致问题变得更糟。

**我可以回到一次问一个问题吗？**
是的。将此添加到你的全局 `CLAUDE.md` 中：

```
When grilling, ask one question at a time.
```

**如果我真的不知道答案怎么办？**
就这么说了。“我不知道”是一个真实的答案，一个你无法回答的问题通常是去原型化而不是去猜测的信号。

**我在写规范之前需要开始一个新会话吗？**
不。会话的价值是你刚刚建立的[上下文](https://www.aihero.dev/ai-coding-dictionary/context)。直接将相同的对话交给 [to-spec](https://aihero.dev/skills-to-spec)。

**模型重要吗？**
比大多数技能更重要。Grilling 依赖于 [模型](https://www.aihero.dev/ai-coding-dictionary/model) 本身对系统如何崩溃的感知，所以给它最好的一个。实现主要遵循上下文并容忍较便宜的模型。

## 在系统中的位置

`grill-me` 是一个**你可以随处运行，在任何东西上运行的独立工具**。无状态是使其可移植的原因：没有仓库，没有工作空间，没有设置，也没有假设想法甚至与软件有关。人们将其指向商业决策、写作、接下来做什么——任何不会停留在他们脑海中的东西。

这种可移植性是与 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 的全部区别，后者运行相同的面试但读取代码库以对齐并记录所学内容为 `CONTEXT.md` 和 ADRs。两者都建立在 [grilling](https://www.aihero.dev/skills-grilling) 原语之上；`grill-me` 是用户调用的前门，不带任何附加物。

如果你 grilled 的东西确实变成了软件，你可以将相同的对话交给 [to-spec](https://aihero.dev/skills-to-spec) 并继续进行构建流程——这是一个选项，而不是该技能的要点。当你不确定哪个流程合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
