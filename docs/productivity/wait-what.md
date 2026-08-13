## 功能说明

`wait-what` 是当一条消息没有传达清楚时你输入的命令。[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 随后会重新阐释它刚才说的话。它会补充你遗漏的上下文，用浅显的英语书写，并使用你项目中 `CONTEXT.md` 的词汇。

该技能只有三行长。这是设计使然，而非未完成的草稿。与冗长对抗的技能往往因不断膨胀而失败：一个四百行的简洁技能仍会让 [model](https://www.aihero.dev/ai-coding-dictionary/model) 保持啰嗦，因为 model 读到的是篇幅，而不是恳求。这个技能只携带一个精确的引导词，仅此而已。

## 何时使用

你通过输入 `/wait-what` 来调用它。agent 不会自行使用它，也不应该使用。只有你自己知道你何时开始跟不上了。

一旦你注意到自己开始走马观花，就立即使用它。agent 已经滑向你从未见过的、它自己发明的行话，堆砌了五个缩写词，或者解释了一个你从未看到前提的决策。它能修复你正在进行的对话。若想彻底阻止行话出现，请使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它会预先构建共享语言。

## 名称即机制

引导词是 **wait**。"要简洁" 是针对 agent 输出的一条指令，model 通过删减词语来服从它，结果让你更加跟不上。**wait** 关乎 *你的* 状态。它表示理解在此处失败。听到"简短点"的 agent 会写出电报体。听到"等等，你跟丢了"的 agent 会回溯并解释。

这个区别就是整个技能的全部。每个流行的冗长修正方案都命名了 *输出*：`/tldr`、`/no-fluff`、`/talk-normal`。model 会过度纠正，进入一种更短却并不更清晰的原始人语域。命名 *听者* 则同时要求了两部分：更少的词语 **以及** 你遗漏的上下文。

该技能说的是重新阐释**那件事**，而不是"最后那条消息"。让你跟丢的通常不止一个段落，因此 agent 会决定回溯多远。

## 它与你已有的语言相衔接

技能主体复用了你全局 `CLAUDE.md` 和项目 `CONTEXT.md` 中已有的引导词。ASD-STE100 简化技术英语设定了语域。通用语言提供了名词。该技能、`CLAUDE.md` 和 `CONTEXT.md` 指向相同的 [tokens](https://www.aihero.dev/ai-coding-dictionary/token)，因此调用它并不是一条新指令，而是对 agent 已经同意的一条指令的提醒。

如果你没有 `CONTEXT.md`，该技能仍然有效。你只是失去了领域词汇那一半。

## 判断是否生效

* 重新阐释**更短且更清晰**，而不是更短但更生硬。
* 它补充了你遗漏的前提，而不只是删除词语。
* 项目名词取代了发明出来的名词。`CONTEXT.md` 中的术语回来了。
* 你可以连续使用两次，它不会退化成简短生硬。

## 在系统中的位置

你可以在任何时间、任何对话中、任何其他技能内部使用 `wait-what`。它能事后修复某条消息。真正的解药是事先达成一致的共享语言，那就是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：一场 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 会话，在过程中运行 [domain-modeling](https://aihero.dev/skills-domain-modeling)，让你们双方使用的词语落入 `CONTEXT.md`。如果你不确定哪个技能适合当下，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
