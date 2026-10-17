## 它做什么

当你听不懂一条消息时，输入 `wait-what`。[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 随后会重新表述它刚才说的内容。它会补充你缺失的上下文，用通俗英语书写，并使用你项目 `GLOSSARY.md` 中的词汇。

该技能仅三行长。这是设计使然，而非未完成的草稿。对抗冗长的技能通常会失败，因为它们会变得太长。一条四百行的简练技能仍会让 [model](https://www.aihero.dev/ai-coding-dictionary/model) 啰嗦，因为模型模仿的是技能的长度，而非其请求。这个技能只有一个精确的首词，别无其他。

## 何时使用它

你通过输入 `/wait-what` 来调用它。agent 不会自行使用它，也不应该自行使用。只有你知道自己什么时候跟不上了。

一旦发现自己在略读，立刻使用它。例如，agent 开始使用它自造的行话，在一句话里塞进五个缩写，或解释一个你从未见过前提的决定。它修复的是你正在进行的对话。要从根源上杜绝行话，请使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它能在前期建立共享语言。

## 名称就是机制

首词是 **wait**。“要简练”是对 agent 输出的指令，模型通过删减字词来服从，反而让你更跟不上。**Wait** 关乎*你的*状态。它表示你在这里停止理解了。听到“简短点”的 agent 会写出残缩片段。听到“等等，你把我弄丢了”的 agent 会退回去重新解释。

这个差别就是整个技能。所有流行的冗长修复方案都命名*输出*：`/tldr`、`/no-fluff`、`/talk-normal`。模型过度修正成更短但不更清晰的残缩片段。命名*听众*能同时要求两方面：更少的字词**和**你缺失的上下文。

该技能说的是重新说明**那件事**，而不是“那最后一条消息”。让你跟不上的通常比一个段落更大，因此由 agent 决定要回溯多远。

## 它与你已有的语言相衔接

技能正文复用了你全局 `CLAUDE.md` 和项目 `GLOSSARY.md` 中已有的首词。ASD-STE100 简化技术英语设定了语域。通用语言提供了名词。技能、`CLAUDE.md` 和 `GLOSSARY.md` 使用相同的 [tokens](https://www.aihero.dev/ai-coding-dictionary/token)，因此调用它不是新指令。它只是提醒 agent 遵守它已同意的一条指令。

如果你没有 `GLOSSARY.md`（也没有指向当前上下文词表的 `GLOSSARY-MAP.md`），该技能依然有效。你仅损失领域词汇那一半。

## 如果它起作用了

* 重新说明**更简短且更清晰**，而不是更简短却更生硬。
* 它补充了你缺失的前提，而不只是删掉词语。
* 项目名词取代自造名词。你 `GLOSSARY.md` 中的术语回归。
* 你可以连续使用它两次，而它不会退化成为简略生硬的表达。

## 它在系统中的位置

你可以在任何时刻、任何对话、任何其他技能中使用 `wait-what`。它事后修复一条消息。真正的修复是预先约定的共享语言，那就是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)：一次 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 会话，在进行中运行 [domain-modeling](https://aihero.dev/skills-domain-modeling)，从而将你们共同使用的词汇记录进 `GLOSSARY.md`。如果不确定哪个技能适合当下，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你导航。
