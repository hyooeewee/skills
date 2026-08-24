## 它做什么

它是 **[无状态](https://www.aihero.dev/ai-coding-dictionary/stateless)** 的。它不写任何文件，也不留下工作区。它唯一留下的是你脑海中更清晰的想法版本。

它是 **[无状态](https://www.aihero.dev/ai-coding-dictionary/stateless)** 的。它不写任何文件，也不留下工作区。它唯一留下的是你脑海中更清晰的想法版本。

## 何时使用它

你通过输入 `/grill-me` 来调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自己调用它。在**新的对话**中启动它，而不是在你已经有代理编写好计划的基础上。

一旦你有了一个值得认真对待的想法（功能、产品方向、业务决策、写作内容），就尽快使用它，远在你弄清楚它涉及什么之前。含糊不是等待的理由；正是 session 会将其消化的事物。如果你已经能精确描述它，就不需要 grill 它了。

你想要三种 grilling 技能中的哪一种，取决于你面前的东西：

* “任何事，任何地”：`grill-me`。它不需要代码库，也不写文件，且主题不必是代码。
* “用于对照的代码库”：[grill-with-docs](https://aihero.dev/skills-grill-with-docs)。同样的访谈，但是 [有状态](https://www.aihero.dev/ai-coding-dictionary/stateful)：它会读取你的代码，并将学到的内容保存在 `CONTEXT.md` 和 ADRs 中。
* “单次会话无法处理”：[wayfinder](https://aihero.dev/skills-wayfinder)。它将工作量绘制成地图，并在其中运行 grilling 会话。

关闭 [计划模式](https://www.aihero.dev/ai-coding-dictionary/agent-mode)。计划模式会让代理急于产出计划，这与保持探究恰恰相反。

## 这是对话，不是访谈

这项技能负责提问，但**你**掌控范围。这是人们忽略的部分，它区分了把一个想法变成决策的会话和产生自信废话的会话。

失败模式是**被动性**：回答“同意，同意，同意”四十个问题，最后得到一个你点头过的代理写的计划。它感觉很高效，因为它很长。实际上并没有做出任何决定，而且结果带有它尚未赢得的确信。

主动意味着引导。当某个问题低于你所需的精度时，要反驳。当范围偏离时要说出来。回答“我不知道”并且是认真的。这项技能是为辅助工程师而构建的，而不是取代工程师：产出取决于你答案的质量，而不是提问的数量。

相反的错误虽然真实但较少见：在访谈中停留太久，以至于永远无法触及代码。

## 可 grill 与不可 grill

有些问题可以通过交谈来回答。有些则不能，再多的 grilling 也无法让你到达那里。

“一份长表单还是三页？”和“这个交互应该感觉如何？”是 **不可 grill 的**：它们需要某种东西来反应。当你遇到一个时，停止 grilling。使用 [prototype](https://aihero.dev/skills-prototype) 构建一个临时版本，看看它，然后回来用一句话回答。

试图用交谈来通过一个不可 grill 的问题，正是会话膨胀的地方。代理不断换一种说法，你不断猜测，范围不断增长以填满不确定性。

## 如果它起作用了

* 你对某些事情表示不同意。一个没有你反驳的会话，是你并不需要的会话。
* 问题分几轮到达，而不是一条长长的细流，并且后面的轮次明显建立在你之前所说的内容之上。
* 你最终到达了一个你未曾预料的地方，因为一个问题揭示了你一直在隐式做出的决定。
* 在最后，你能向一个不在场的人为每个选择辩护。

## 常见问题

“我应该预期多少个问题，以及如何知道何时结束？”数轮数，不是问题数。四轮共四十六个问题是一个普通的会话。当前沿为空时结束：每个分支都已访问，没有剩下隐式假设的内容。

**它问了我两百个问题。哪里出了问题？**
通常是范围太大。先让代理把工作分解成更小的部分，然后分别 grill 每一部分。非常长的会话也会漂移到 **[傻区](https://www.aihero.dev/ai-coding-dictionary/smart-zone)**，此时 [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window) 已经足够满，问题会变得更糟。

**我可以回到一次只问一个问题吗？**
可以。将此添加到你的全局 `CLAUDE.md`：

```
When grilling, ask one question at a time.
```

**如果我确实不知道答案怎么办？**
直接说出来。“我不知道”是一个真实的答案，而一个你无法回答的问题通常是一个应该原型化而不是猜测的信号。

**在编写规格说明之前，我需要开启一个新的会话吗？**
不需要。会话的价值在于你刚刚构建的 [上下文](https://www.aihero.dev/ai-coding-dictionary/context)。将同一个对话直接交给 [to-spec](https://aihero.dev/skills-to-spec)。

**模型有关系吗？**
比大多数技能更重要。Grilling 依赖于 [模型](https://www.aihero.dev/ai-coding-dictionary/model) 自身对系统如何崩溃的感知，所以给它你最好的模型。实现主要是跟随上下文，可以容忍较便宜的模型。

## 它在系统中的位置

`grill-me` 是一个 **你可以随时在任何东西上运行的独立工具**。无状态正是使其可移植的原因：没有代码库，没有工作区，没有设置，也没有关于想法甚至与软件无关的假设。人们把它指向业务决策、写作、接下来做什么：任何不会停留在他们脑海中的事物。

这种可移植性正是它与 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 的全部区别，后者运行同样的访谈，但会读取一个代码库来对齐，并将其学到的内容记录为 `CONTEXT.md` 和 ADR。两者都基于 [grilling](https://aihero.dev/skills-grilling) 原语；`grill-me` 是用户调用的、不携带任何东西的入口。

如果你 grill 的东西确实结果是软件，你可以将同一个对话交给 [to-spec](https://aihero.dev/skills-to-spec) 并继续进入构建流程（这是一个选项，不是技能的重点）。当你不确定哪个流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指引。
