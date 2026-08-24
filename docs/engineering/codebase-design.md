## 它做什么

`codebase-design` 界定了你设计模块时所用的词汇：**module**（模块）、**interface**（接口）、**depth**（深度）、**seam**（接缝）、**adapter**（适配器）、**leverage**（杠杆）、**locality**（局部性）。它精确地定义每一个词，禁用松散的替代词（“component”、“service”、“API”、“boundary”），并阐明由它们得出的那几条原则。

It is a reference, not a process. There is no loop to run, no artifact it produces, no checkpoint where it asks you a question. Every other skill that touches design borrows its vocabulary; on its own it gives you the language and stops. That is the thing to know before you invoke it, because a skill with no process and no stopping rule will improvise one if you point a [会话](https://www.aihero.dev/ai-coding-dictionary/session) at it and say "go." See the questions below for what that looks like in practice.

## 何时使用它

输入 `/codebase-design`，或者当某个设计任务适用时，agent 会自动调用它。

当你已经知道要重新设计哪些代码，并且需要思考它的形态时，就使用它：接缝放在哪里、接口能有多小、一次抽取是否值得保留。要解决关于某个词含义的争论时，你也会用到它。

有几个技能和它靠得很近。你想要哪一个取决于实际的问题是什么：

| 问题                                                 | 技能                                                                                                   |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| 一个模块的形状：它的接口、接缝和深度                                 | `codebase-design`                                                                                    |
| 该 *领域的词汇*：“account”意味着三件事，两个人对“cancellation”有不同的理解 | [domain-modeling](https://aihero.dev/skills-domain-modeling)                                         |
| 你还不知道 *哪*个模块需要重新设计                                 | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture)（用于发现候选者的调查） |
| 你希望设计接受质疑，而不只是被命名                                  | [追问](https://aihero.dev/skills-grilling)                                                             |
| 有一个具体的行为要构建，并且你希望测试能在重构中幸存下来                       | [tdd](https://aihero.dev/skills-tdd)                                                                 |

## 词汇表

词汇表就是这个技能。每个术语都相对于其他术语来定义，而且每个都带有它所替代的词。

| 术语                | 含义                                                                                     | 不要使用                             |
| ----------------- | -------------------------------------------------------------------------------------- | -------------------------------- |
| **模块（Module）**    | 任何拥有接口和实现的东西。刻意保持与规模无关：一个函数、一个类、一个包、跨越多个层级的切片。                                         | unit、component、service（单元、组件、服务） |
| **接口（Interface）** | 调用者为了正确使用它而必须知道的一切：类型签名，外加不变量、顺序约束、错误模式、必需的配置、性能特征。                                    | API、signature（签名）                |
| **深度（Depth）**     | 接口处的杠杆：调用者或测试每学习一单位接口，能执行多少行为。 **深（Deep）**：大量行为隐藏在一个小接口背后。 **浅（Shallow）**：接口几乎与实现一样复杂。 | 无                                |
| **接缝（Seam）**      | Michael Feathers 的术语：一个无需在该处编辑就能改变行为的地方。它是接口的 *位置*，而把它放在哪里本身就是独立的决策，与它背后放什么分开考虑。       | boundary（边界）                     |
| **适配器（Adapter）**  | 在接缝处满足接口的具体事物。它命名的是一个角色，而非实质：一个内存中的假对象和一个 Postgres 仓库都是适配器。                            | 无                                |
| **杠杆（Leverage）**  | 调用者从深度中获得的东西：每学习一单位接口，就获得更多能力。                                                         | 无                                |
| **局部性（Locality）** | 维护者从深度中获得的东西：变更、缺陷和验证集中在一处。修复一次，处处生效。                                                  | 无                                |

深度特意*不*被定义为实现代码行数与接口代码行数之比——尽管那是 Ousterhout 本人的定义。那个指标会奖励给实现注水的行为。取而代之的是“深度即杠杆”（depth-as-leverage）的定义。

## 四条原则

* \*\*深度是接口的属性，不是实现的属性。\*\*一个深模块内部可以由可替换的小部件构建而成。它们只是不会向调用者显现。一个模块可以有供自身测试使用的内部接缝，以及一个位于其接口上的外部接缝。
* \*\*删除测试。\*\*想象删除这个模块。如果复杂性随之消失，那它就是个透传。如果复杂性重新出现在 N 个调用者身上，那它就是在体现自己的价值。
* \*\*接口就是测试表面。\*\*调用者和测试穿过同一个接缝。如果你想测试*越过*接口的东西，那么这个模块的形状就是错的。
* \*\*一个适配器意味着假设的接缝；两个适配器意味着真实的接缝。\*\*在确实有东西跨越它变化之前，不要切出接缝。单一适配器的接缝只是间接层。

两个支持文件进一步提供了细节，技能按需读取它们而不是提前读取。[DEEPENING.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DEEPENING.md) 将候选者的依赖归类为四个类别（in-process、local-substitutable、remote-but-owned、true-external），因为类别决定了经过接缝测试的深化模块是如何工作的。[DESIGN-IT-TWICE.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DESIGN-IT-TWICE.md) 启动并行的[sub-agents](https://www.aihero.dev/ai-coding-dictionary/subagent)来为同一个模块生成三个或更多根本不同的接口，然后根据深度、局部性和接缝位置进行比较。

## 常见问题

**我到底如何在 TypeScript 中构建一个深模块？**

这是关于这个技能被问得最多的问题，而这个技能并没有回答它。它定义了深模块*是什么*，却完全没有提到如何阻止游离的导入越过接口。[Issue #458](https://github.com/mattpocock/skills/issues/458) 说得直白：“假设我们对接口感到满意，它隐藏了细节，等等。但我们如何强制实施它？我认为如果没有 lint 或清晰的护栏，人类和 LLM 都会随着时间推移开始把它弄得一团糟。”Matt 在那个讨论串中的回答是三个选项：把它包在一个 class 或 IIFE 里，并接受这个类会变得巨大；把它做成 monorepo 中的一个包，并接受 monorepo 的工具链；或者使用像 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 这样的 linter，禁止绕过接口的导入。他另外还分别称 Effect 是最好的机制，dependency-cruiser 是第二好的。仓库的 `in-progress/` 目录中有一个 `setup-ts-deep-modules` 技能，它定下了一种 `src/packages/<name>/index.ts` 约定，但它是一个没有文档页面的测试版技能，而且没有随附任何 lint 规则。

**我把一个 session 指向它，结果它烧掉了 10 万 [tokens](https://www.aihero.dev/ai-coding-dictionary/token)重新设计我从未要求过的东西。**

已知，并已作为 [issue #449](https://github.com/mattpocock/skills/issues/449) 提交。该技能是模型触发的，且将其自身描述为词汇，但其中没有任何硬性规定阻止 agent 将其视为可运行的进程。被要求“在 /codebase-design 中恢复并推动开放决策”时，agent 找到了它所能找到的最具行动导向的内容：`DESIGN-IT-TWICE.md` 中的并行子代理。它重新探索了前一个 session 已经映射过的代码，并且在提出任何问题之前运行了很久。这里不存在驱动技能的所有护栏（检查点、一次一个问题、无自动推进），因为参考文档没有这些。变通方法是命名一个驱动技能，让这个技能位于它下面：使用 `codebase-design` 作为词汇表的 `/grill-with-docs`、`/improve-codebase-architecture` 或 `/tdd`。该问题仍然开放。

**把 `design-an-interface`移到哪去了？还有 `/interface-design`这个技能吗？**

`design-an-interface` 已被移除并吸收进此技能中。没有丢失任何东西：它的“两次设计”技术（Ousterhout 提出的并行子代理生成根本不同设计）作为 `DESIGN-IT-TWICE.md` 跟随此处。此外，几个人曾请求为 deep-module/thin-interface 哲学建立一个专门的 `/interface-design` 技能；该哲学已经存在于这里，也没有计划单独的技能。如果你是在找这两个名字中的任何一个，这就是你要找的页面。

**这难道不是一种文件结构约定，比如文件夹、barrel 文件、feature slices 吗？**

不是，而且该技能在反复的阻力下一直坚持这个立场。[Issue #95](https://github.com/mattpocock/skills/issues/95) 提出了一种形式化的分形树文件结构作为深模块的具体实现；回复是两者是正交的：‘深模块关乎接口的设计以及通过严格接口的访问，无论文件系统长什么样。使用这种方法似乎完全可以拥有浅模块。’同样的事情也出现在 #458 中：‘我认为你把模块的概念与文件系统系得太紧了。文件系统当然可以是对模块形状的一个有用提示，但在构建深模块时没有必要使用文件系统。’词汇表刻意将 **module** 定义为与规模无关。

**&#x20;`tdd`真的使用这套词汇吗？**

现在可以了。很长一段时间它并没有。以前存在于 `tdd` 内部的内联深模块笔记在 v1.0 中被移除，取而代之的是这个共享技能，但替换它们的指针从未被添加，所以 `tdd` 为自己定义了“seam”并引用了其他内容。这个空白已经填补：指针现在在这个技能中，当接口的形状是开放问题而不是测试问题时，会调用它。`tdd` 仍然拥有“seam”作为你*测试*的边界；这个技能拥有它背后的模块形状。

**design-it-twice 模式在 Claude Code 之外也能工作吗？**

并不干净利落。`DESIGN-IT-TWICE.md` 说‘使用 Agent 工具并行生成 3+ 个子代理’，这是 Claude Code 的 [tool](https://www.aihero.dev/ai-coding-dictionary/tool)（按 Claude Code 的名字命名的）。该仓库为其他 [harnesses](https://www.aihero.dev/ai-coding-dictionary/harness)（包括 Codex）提供元数据，而这些可能不会在该名称下暴露任何内容，因此并行设计阶段的可移植性不如技能的元数据所暗示的那样强。已在 [issue #564](https://github.com/mattpocock/skills/issues/564) 中跟踪，处于开放状态。

**我可以把我的概念添加到词汇表中吗，比如 connascence（共变）、module secrets（模块秘密）， [progressive disclosure（渐进式披露）](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)？**

人们确实提议过这些。[Issue #180](https://github.com/mattpocock/skills/issues/180) 将 Parnas 的 module secrets（模块秘密）和 Page-Jones 的 connascence（共变）作为命名层加入，用来说明*什么*正在穿过接缝泄漏，并附带了可用的 diff；[issue #303](https://github.com/mattpocock/skills/issues/303) 提议在实现内部采用 progressive disclosure（渐进式披露），这样在公共接口上是深模块的模块，其内部就不会是一整块无从区分的平板。两者都处于开放状态，尚未合并。随附的词汇表刻意保持很小，而它保持很小的原因在技能本身中就有说明：一致的语言才是全部意义所在，一个没人能一致使用的术语比没有术语更糟。

## 如果它起作用了

* 设计对话不再产出 "component"、"service" 和 "boundary" 这些词，而是开始产出 "module"、"interface" 和 "seam"。
* 有人能指着提议的抽取方案，毫不犹豫地说出它是否通过删除测试。
* 提议的接缝要附带第二个已命名的适配器，而不只是第一个。
* 对接口的讨论涵盖了不变量、顺序和错误模式，而不仅仅是类型签名。
* 调用它不会启动会话。如果 agent 单凭 `/codebase-design` 就开始读取文件并提出重构，那就是把参考文档误当成了驱动者。

## 它在系统中的位置

`codebase-design` 是一个 **随时可用的独立技能**，它是位于工程技能之下的词汇层，而不是任何流程中的一个步骤。它最近的邻居是 [domain-modeling](https://aihero.dev/skills-domain-modeling)，这是 *问题领域* 的词汇而非模块结构的并行参考。这两者通常都需要，因为给深层模块起个好名字既需要词汇也需要结构。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是另一个：它会扫描代码库以寻找深度化候选，并将它们全部写入此术语表中，因此它能找到模块，而这个技能就是你设计该模块的基准。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指引方向。
