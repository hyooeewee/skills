## 它做什么

`grill-me` 接受一个**模糊的想法**并对你进行访谈，直到你能对其做出承诺。你不需要一个完善的计划就能开始，因为 [session](https://www.aihero.dev/ai-coding-dictionary/session) 的存在就是为了产生一个计划。它以**轮次**进行提问。每一轮都是完整的**前沿**，即所有前置条件你已经确定的问题。因此它永远不会问你依赖于它尚未听到的答案的问题。

它是\*\*[无状态的](https://www.aihero.dev/ai-coding-dictionary/stateless)\*\*。它不写入文件，也不留下任何工作区。唯一的结果是你脑海中更清晰的想法版本。

## 何时使用它

你通过输入 `/grill-me` 来调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自己调用它。在**新的对话**中启动它，而不是在你已经有代理编写好计划的基础上。

只要你有一个值得认真对待的想法（一个功能、一个产品方向、一个商业决策、一篇写作），就立即使用它，且要在你弄清楚它包含什么之前很久就开始。模糊不是等待的理由，因为会话的存在就是为了消除模糊。如果你已经能精确指定这件事，就不需要 grill 它。

你想要三种 grilling 技能中的哪一种，取决于你面前的东西：

* **任何事物，任何地方。** 使用 `grill-me`。它不需要代码仓库，不写入文件，主题也不必是代码。
* **有一个代码库需要对齐。** 使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)。这是同样的访谈，但是[有状态的](https://www.aihero.dev/ai-coding-dictionary/stateful)：它读取你的代码，并将所学保存在 `GLOSSARY.md` 和 ADRs 中。
* **太大，无法在一个会话中完成。** 使用 [wayfinder](https://aihero.dev/skills-wayfinder)。它将工作量绘制为地图，并在其中运行 grilling 会话。

关闭 [plan mode](https://www.aihero.dev/ai-coding-dictionary/agent-mode)。Plan mode 会让代理急于产出计划，而你希望它继续提问。

## 这是对话，不是访谈

技能提出问题，但**你**拥有范围的所有权。人们经常忽略这一点。这正是将想法转化为决策的会话，与产生自信胡扯的会话之间的区别。

失败模式是**被动**：对四十个问题回答"同意、同意、同意"，最后得到一个代理写的、你点头赞同的计划。感觉很有成效，因为过程很长。但你什么也没决定，结果看起来比实际更确定。

积极参与意味着掌舵。对细节程度不够的问题推回去。指出范围偏移时说出来。回答"我不知道"并当真。这个技能帮助工程师，不取代工程师。结果的质量取决于你答案的质量，而非问题的数量。

相反的错误虽然真实但较少见：在访谈中停留太久，以至于永远无法触及代码。

## 可 grill 与不可 grill

有些问题可以通过交谈来回答。有些则不能，再多的 grilling 也无法让你到达那里。

"一个长表单还是三个页面？"和"这个交互该怎么感觉？"是**不可 grill 的**。你需要有东西可供反应，才能回答它们。遇到这种情况，停止 grilling。用 [prototype](https://aihero.dev/skills-prototype) 构建一个一次性版本，看一看，然后回来用一句话回答。

当你试图通过谈话来解决一个不可 grill 的问题时，会话会变得太长。代理不断重述，你不断猜测，范围随不确定性增长。

## 如果它起作用了

* 你对某些事情表示不同意。一个没有你反驳的会话，是你并不需要的会话。
* 问题分几轮到来，而不是长时间一个个来，且后续轮次清晰地建立在你早先所说的基础上。
* 你到达了一个意想不到的地方，因为一个问题揭示了你一直在不知不觉中做的决定。
* 在最后，你能向一个不在场的人为每个选择辩护。

## 常见问题

**我应该预期多少问题，怎么知道何时结束？**
数轮次，别数问题。四轮四十六个问题很普通。当前沿为空时结束：它访问了每个分支，没有任何东西作为未陈述的假设留下。

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

**模型重要吗？**
比大多数技能都重要。Grilling 依赖 [model](https://www.aihero.dev/ai-coding-dictionary/model) 自身关于系统如何崩溃的知识，所以用你最好的模型。实现主要跟随上下文，所以那里用便宜模型也行。

## 它在系统中的位置

`grill-me` 是一个**可在任何地方、对任何事物运行的独立工具**。因为它是无状态的，所以可移植。它不需要代码仓库、工作区或设置，也不假设想法关乎软件。人们用它做商业决策、写作、决定下一步：任何他们自己无法清晰思考的事。

可移植性是与 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 的唯一区别。该技能运行同样的访谈，但读取代码库以对齐，并将所学记录为 `GLOSSARY.md` 和 ADRs。两者底层都使用 [grilling](https://aihero.dev/skills-grilling) 技能。`grill-me` 是用户调用的入口点，不保留状态。

如果你 grill 的东西确实结果是软件，你可以将同一个对话交给 [to-spec](https://aihero.dev/skills-to-spec) 并继续进入构建流程（这是一个选项，不是技能的重点）。当你不确定哪个流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指引。
