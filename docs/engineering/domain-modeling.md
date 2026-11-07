## 它做什么

`domain-modeling` 在你设计时构建并打磨项目的**通用语言**。它会质疑与词汇表冲突的术语，在你用模糊词汇的地方要求用精确词汇，并针对具体场景测试关系，直到边界精确。

这是**主动**的规范，而非被动规范。任何技能都能读取 `GLOSSARY.md` 来借用其词汇。这个技能用于当你在*更改*模型时，这就是为什么它会打断你。它在你确定术语的那一刻将其写入 `GLOSSARY.md`，就在对话中间，而不是在最后生成一个整洁的词汇表。写在最后的词汇表是一次 [session](https://www.aihero.dev/ai-coding-dictionary/session) 的总结；内联版本才是会话的产出。

## 何时使用它

输入 `/domain-modeling`，或当任务匹配时由代理自动调用。实际上，自动调用是该技能最薄弱的环节。当 `grill-with-docs` 或 `wayfinder` 要求加载它时，[models](https://www.aihero.dev/ai-coding-dictionary/model) 往往加载 `grilling` 而跳过这个技能。如果一次 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 会话运行结束时 `GLOSSARY.md` 未变，那就是这种情况。请在其他技能旁显式调用它的名称。

当*词语*成为问题时，就使用它：

| 情况                          | 做法                                                                       |
| --------------------------- | ------------------------------------------------------------------------ |
| 两个人对 "cancellation" 的含义理解不同 | `domain-modeling`选定规范术语，将其余的列在 \`\_Avoid\_\` 下 `_Avoid_`                 |
| "Account" 在三个文件中承担了三种职责     | `domain-modeling`将其拆分为 Customer 和 User                                   |
| 你刚刚做出了一个难以逆转的架构选择           | `domain-modeling`它会提供 ADR，如果该选择通过门槛                                      |
| 模块的 *形状*是问题所在：接缝在哪里，接口有多深   | [codebase-design](https://aihero.dev/skills-codebase-design)             |
| 你希望在构建之前对整体计划进行审查           | [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它会在底层驱动这个技能 |
| 你想要查找一个术语，而不是更改它            | 什么都不做。阅读 `GLOSSARY.md`。它只是一个文件。                                          |

## 先决条件

一开始什么都不需要。该技能会写入两个位置，并且都是惰性创建的：

* 位于仓库根目录的 **`GLOSSARY.md`**。当你确定第一个术语时，技能会创建它。在根目录有 `GLOSSARY-MAP.md` 的仓库中，术语会进入映射指向的各上下文 `GLOSSARY.md` 中。
* **`docs/adr/`**。技能随通过门槛的第一个 ADR 创建它。

开始前无需任何准备，技能也不会预先创建任何东西。

## 两种产物，两道标准

词汇表和 ADR 有不同的门槛。该技能的大多数麻烦源于混淆它们。

|      | `GLOSSARY.md`                                                              | `docs/adr/NNNN-slug.md`          |
| ---- | -------------------------------------------------------------------------- | -------------------------------- |
| 保存   | 术语。某事物 **是什么**，用一两句话说明，并在 `_Avoid_`                                        | 一个决策，用一到三句话说明：背景、选择、理由           |
| 写入门槛 | 一个含糊的术语成为规范术语                                                              | **全部三项**：难以逆转、脱离背景会令人意外、是真正权衡的结果 |
| 写入方式 | 内联，在术语确定的那一刻                                                               | 主动提供，而非假设生成                      |
| 绝不保存 | 实现细节、 [规格说明](https://www.aihero.dev/ai-coding-dictionary/spec)、草稿纸、一般的编程概念 | 本次会话中每个选择的流水账                    |

如果 ADR 的三项测试中有一项不满足，就不写 ADR。一个容易逆转的决策只会被逆转；一个不令人意外的决策无人会质疑；一个没有真正替代方案的决策，记录的只是你做了显而易见的事。

`GLOSSARY.md` 规则最重要，因为它是在实践中最容易被打破的。**它只是一个词汇表，别无他物。** 如果不加控制，模型会把"写入 `GLOSSARY.md`"视为持久化你给出的每个答案的许可，导致文件变成一份持续更新的规格说明。这是跨多个模型报告最多的问题。

## 交叉引用，以及它的边界在哪里

当你描述某事如何工作时，技能会检查代码并指出任何矛盾。*"你的代码取消整个 Order，但你刚说支持部分取消，到底哪个对？"* 在更改语言或代码之前，技能会让它们公开达成一致。

它交叉引用**代码**和已提交的 `GLOSSARY.md` 与 ADR，仅此而已。它不搜索你的问题追踪器。如果你的团队几个月前在一个已关闭的 issue 中争论并解决了命名冲突，技能会像新问题一样再次提出。有一个 [open request](https://github.com/mattpocock/skills/issues/717) 来修复此问题。在此之前，变通方法是将指令放在你自己的 `docs/agents/domain.md` 中，技能已经会读取它。

## 常见问题

**我的 `GLOSSARY.md` 有 500 行。1,000 行。3,000 行。怎么办？** 规模是症状。原因是文件吸收了从未属于词汇表的实现细节和决策。修复方法是直接指令：`/grill-with-docs make my GLOSSARY.md more concise and remove any implementation details from it`。对膨胀的文件运行它，大部分内容会被删减。只有当文件精简后仍覆盖读者不愿同时持有的两个领域时，才用 `GLOSSARY-MAP.md` 拆分。拆分膨胀文件只会得到多个膨胀文件。技能在这方面的指导还不足以防止初期膨胀，追踪该问题的 issue 仍是开放的。

**为什么是 `GLOSSARY.md` 而不是 `CONTEXT.md`？** 直到 1.3.0 版本它还是 `CONTEXT.md`（和 `CONTEXT-MAP.md`）。文件是词汇表，别无他物，所以 `GLOSSARY.md` 直呼其名，正如一位读者所言："with ai agents everything is [context](https://www.aihero.dev/ai-coding-dictionary/context)"。如果你的仓库仍有 `CONTEXT.md`，重新运行 `/setup-matt-pocock-skills`：它会提议 `git mv` 该文件并更新 `docs/agents/domain.md`。套件中的每个技能都查找 `GLOSSARY.md`，保留旧名意味着要修补所有技能。

**`/ubiquitous-language` 去哪了？** 它被移除了，但未被废弃。它的职责移入了 `domain-modeling`，后者持续维护整个模型，而不是从一次对话中倾倒出一个词汇表。词汇强制现在更重要，而非更不重要。它在 grilling、triage 和 mapping 底层运行，而不是作为你必须记得运行的单独步骤。

**如何为没有词汇表的代码库获取一个？** 显式请求，而不是等待它积累。`/grill-with-docs help me scaffold my existing repo with a GLOSSARY.md` 是文档化的路径。预期会有漫长的审问。一位用户报告文件成型前被问了 50 多个问题。在棕地仓库中，偶发使用构建词汇表的速度太慢。

我可以保留领域模型并使用我自己的 ADR 格式吗？今天还无法干净利落地做到。词汇表一半和 ADR 一半封装在一个技能中，因此拥有既定 ADR 约定的团队（不同的模板、不同的位置、不同的命名）会得到与其内部风格冲突的指令。当前选项是本地复制技能并编辑它，或者覆盖你仓库中自身代理文档里的 ADR 约定。将两者分开是 [一个开放请求](https://github.com/mattpocock/skills/issues/557)。

**词汇表真的值得维护吗？它是一个额外的审查产物，且可能过时。** 有时确实不值得。DDD 越接近实现越没用。收益在上游，在于命名和概念对齐，不在于聚合和分层仪式。同义词控制在命名边界很重要：模块名、表名、状态枚举、issue 标题、CLI 命令。在普通散文中则不那么重要。还有观点认为，领域术语压缩了*已共享术语的人类之间*的沟通，而代理对纯英语描述的反应相同。按此理解，词汇表的价值是让你和评审者与代理的行为保持一致，而非让代理变得更好。一天构建的项目，跳过它。而且未经审查的代理撰写词汇表比没有更糟：它充满自信的断言，后续会话会将其视为真理。

**它能把我的模糊提示转成领域语言吗？** 不能，也没有计划开发这样的技能。你自己都不理解的领域语言，写下来也毫无意义。这个技能在你已有理解时强制精确；它不会发明你没有的词汇。相关陷阱是没做建模就用领域词汇。错误概念之上的正确名词，产出读起来正确实则不然。

## 如果它起作用了

* 它会在你话说到一半时停下来，问你想表达的是两者中的哪一个，而不是随便选一个继续。
* `GLOSSARY.md` 在对话**期间**变更，而不是在最后集中爆发。
* 它拒绝为明天可以撤销的事情写 ADR，并说明三项测试中哪一项失败了。
* 新条目用一两句话定义某事物*是什么*，并在`_Avoid_`下列出你要放弃的词语。
* 当你的代码和你的表述不一致时，它会把你的代码原样引述给你看。
* `GLOSSARY.md` 变短的频率与变长的频率一样高。

## 它在系统中的位置

`domain-modeling` 是一个**模型调用的引用**，它在其他技能之下运行的频率比单独运行更高。[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 通过 grilling 会话驱动它，[wayfinder](https://aihero.dev/skills-wayfinder) 在绘制地图时加载它，[triage](https://aihero.dev/skills-triage) 用它让 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 保持项目自己的措辞，[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在决策落定时调用它。它最近的兄弟是 [codebase-design](https://aihero.dev/skills-codebase-design)。它们共同构成了所有事物下的词汇层，这个管*领域*，那个管模块的*形状*。也可直接调用，当你想要这种规范但不想承担正常加载它的技能的步骤时。不确定哪个技能合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你导向。
