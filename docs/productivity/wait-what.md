## 它做什么

`wait-what` 是当一条消息没有传达到位时你输入的内容。随后 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 会重新阐述它刚才说的话。它会补充你缺失的上下文，用平实的英语写作，并使用项目 `CONTEXT.md` 中的词汇。

这个技能只有三行。这是设计如此，而不是未完成的草稿。那些与冗长作斗争的技能往往会因不断膨胀而失败：一个四百行的简洁技能仍然会让 [model](https://www.aihero.dev/ai-coding-dictionary/model) 变得冗长，因为模型读取的是篇幅，而不是请求。这个技能只带一个精确的引导词，别无其他。

## 何时使用它

你通过输入 `/wait-what` 来调用它。agent 不会自行使用它，也不应该自行使用。只有你知道自己什么时候跟不上了。

一旦你发现自己开始略读，就立刻使用它。agent 已经陷入了它自己发明的行话，堆砌了五个首字母缩写，或者解释了一个你从未见过前提的决策。它修复的是你正在进行的对话。要想从一开始就阻止行话出现，请使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它会预先建立共同语言。

## 名称就是机制

引导词是 **wait**。“要简洁”是关于 agent 输出的一条指令，模型会通过裁剪词语来服从它，结果让你更加跟不上。**wait** 关注的则是*你*的状态。它表示“这里理解失败了”。一个听到“说简短点”的 agent 会发电报式回复。一个听到“等等，你跟丢我了”的 agent 会退后一步并加以解释。

这一差别就是整个技能的核心。所有流行的冗长修复方案都命名了*输出*：`/tldr`、`/no-fluff`、`/talk-normal`。模型会过度纠正，进入一种更短却并不更清晰的穴居人式的语气。而命名*听者*则同时提出了两方面的要求：更少的词语**以及**你缺失的上下文。

该技能说的是重新说明**那件事**，而不是“那最后一条消息”。让你跟不上的通常比一个段落更大，因此由 agent 决定要回溯多远。

## 它与你已有的语言相衔接

正文复用了你的全局 `CLAUDE.md` 和项目 `CONTEXT.md` 中已有的引导词。ASD-STE100 简化技术英语设定了语域。通用语言提供了名词。该技能、`CLAUDE.md` 和 `CONTEXT.md` 使用的是相同的 [tokens](https://www.aihero.dev/ai-coding-dictionary/token)，因此调用它并不是一条新指令，而是对 agent 已经同意的一条指令的提醒。

如果你没有 `CONTEXT.md`（也没有 `CONTEXT-MAP.md` 指向当前上下文），该技能仍然有效。你只是失去了领域词汇这一半。

## 如果它起作用了

* 重新说明**更简短且更清晰**，而不是更简短却更生硬。
* 它补充了你缺失的前提，而不只是删掉词语。
* 项目中的名词取代了生造的词。你 `CONTEXT.md` 中的术语回来了。
* 你可以连续使用它两次，而它不会退化成为简略生硬的表达。

## 它在系统中的位置

你可以随时在任何对话中、任何其他技能内部使用 `wait-what`。它能在事后修复一条消息。真正的解决之道是预先达成共识的共同语言，这正是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：一场 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 会话，在过程中运行 [domain-modeling](https://aihero.dev/skills-domain-modeling)，让你们双方使用的词语进入你的 `CONTEXT.md`。如果你不确定哪个技能适合当下，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指引。
