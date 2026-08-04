Quickstart:

```bash
npx skills add mattpocock/skills --skill=domain-modeling
```

```bash
npx skills update domain-modeling
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/domain-modeling)

## 功能说明

`domain-modeling` 在设计时构建并打磨项目的**通用语言**——挑战模糊的术语，用具体场景压力测试关系，并在概念形成时立即记录下术语表和决策。

这是**主动**的实践，而非被动的。仅仅阅读 `CONTEXT.md` 来借用其词汇是任何技能都能做到的一行代码习惯；这个技能是为了当你*改变*模型时使用——创造规范术语，捕捉代码与你刚才所说之间的矛盾，记录难以回溯的决定。此外，它还能保持术语表的整洁：`CONTEXT.md` 就是术语表，除此之外别无他物——没有实现细节，没有规范，没有草稿本。

## 何时使用

输入 `/domain-modeling`，或者当任务匹配时代理会自动调用它——当你需要锁定术语、解决一词多义或记录架构决策时。

当*词语*成为问题时使用它：两个人对“取消”有着不同的理解，“账户”承担了三个职责，或者设计对话一直纠缠于一个从未被精确命名的概念。如果相反，模块的*形状*是问题所在——接口在哪里，接口有多深——请使用 [codebase-design](https://aihero.dev/skills-codebase-design)。如果你想在构建之前质询计划本身，请使用 [grilling](https://aihero.dev/skills-grilling)。

## 前置条件

该技能写入两个位置，都是按需创建的——只有当有东西需要记录时才会创建。解析后的术语进入根目录下的 `CONTEXT.md`（或者，在由 `CONTEXT-MAP.md` 标记的多上下文仓库中，进入每个上下文的 `CONTEXT.md`）。决策进入 `docs/adr/`。不需要预先存在任何内容；第一个解析后的术语会创建术语表，第一个真正的权衡会创建 ADR。

## 术语表 vs. 架构决策记录

两个产物，两把不同的标准：

* **术语表** (`CONTEXT.md`) 捕获语言。每次将模糊术语规范化时，都会内联记录——而不是批量记录——以便共享词汇与对话保持同步。它无情地剔除实现细节。
* **ADR** 捕获决策，且标准很高：仅当选择是**难以回溯**的、**脱离上下文令人惊讶**的，并且是**真正权衡的结果**时才提供。错过这三点中的任何一个就没有 ADR。这就是为什么 `docs/adr/` 是有后果的分支记录，而不是日记。

让它生效的举动：当你陈述某事物如何运作时，技能会交叉引用代码并揭示矛盾——“你的代码取消了整个订单，但你刚刚说部分取消是可能的——哪个是对的？”语言和代码被迫达成一致。

## 专门提取

`domain-modeling` 是构建项目通用语言的**单一事实来源**，被拆分为自己的模型触发技能，以便任何其他技能都可以调用它。[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 依赖它来记录术语和决策，[triage](https://aihero.dev/skills-triage) 使用它以项目自己的语言保持工单清晰，而 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在工作时也会调用它。

保持其独立性意味着你也可以直接调用它——作为 sharpen a model（打磨模型）的**参考**——而不必承诺任何这些技能要求的步骤。语言存在于一个地方，所有需要它的东西都指向那里。

## 在系统中的位置

`domain-modeling` 是一个**随时可用的独立技能**，它运行在*底层*其他技能中，就像在固定步骤中一样。它最近的邻居是 [codebase-design](https://aihero.dev/skills-codebase-design)，因为共享语言正是让你能够精确命名深层模块及其接口的原因；在下游， settled 术语表正是 [to-spec](https://aihero.dev/skills-to-spec) 合成为项目自己的语言编写的规范的内容。当你不确定哪个技能或流程合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指引。
