## 功能说明

`codebase-design` 修正你在设计模块时使用的词汇：**模块**、**接口**、**深度**、**缝合点**、**适配器**、**杠杆效应**、**局部性**。它精确地定义了每一个词，禁止松散的替代词（"组件"、"服务"、"API"、"边界"），并陈述了由此产生的少数几条原则。

它是一个参考，而不是一个流程。没有要运行的循环，没有它产生的工件，没有问你问题的检查点。其他所有涉及设计的技能都借用它的词汇；它独自为你提供语言，然后停止。在调用它之前要知道这一点，因为一个没有流程和停止规则的技能，如果你将 [会话](https://www.aihero.dev/ai-coding-dictionary/session) 指向它并说“开始”，它会即兴创作一个流程——见下文的问题。

## 何时使用

输入 `/codebase-design`，或者当设计任务适合时，代理会自动调用它。

当你已经知道要重新设计哪段代码，并且需要考虑其形状时使用它：缝合点在哪里，接口可以多小，提取是否物有所值。这也是你用来解决关于某个词的含义的争论的工具。

有几个技能与之相近。你想要哪一个取决于实际问题是：

| 问题                                        | 技能                                                                           |
| ----------------------------------------- | ---------------------------------------------------------------------------- |
| 单个模块的形状——它的接口、它的缝合点、它的深度                  | `codebase-design`                                                            |
| 领域的\*词汇\* *词汇*——"账户"意味着三件事，两个人对"取消"有不同的意思 | [domain-modeling](https://aihero.dev/skills-domain-modeling)                 |
| 你还不知道\*哪个\* *哪个*模块需要重新设计                  | [改进代码库架构](https://aihero.dev/skills-improve-codebase-architecture)——找出候选者的调查 |
| 你希望对设计进行辩论，而不仅仅是命名                        | [grill-me](https://aihero.dev/skills-grilling)                               |
| 有具体的行为要构建，并且你希望测试能经受住重构                   | [tdd](https://aihero.dev/skills-tdd)                                         |

## 词汇表

词汇表就是该技能。每个术语都是相对于其他术语定义的，并且每个术语都附有它所替换的词。

| 术语       | 它的含义                                                                            | 不要说      |
| -------- | ------------------------------------------------------------------------------- | -------- |
| **模块**   | 任何具有接口和实现的事物。有意设计为规模无关——一个函数、一个类、一个包、一个跨越层级的切片。                                 | 单元、组件、服务 |
| **接口**   | 调用者为了正确使用它必须知道的一切：类型签名，加上不变量、顺序约束、错误模式、所需配置、性能特征。                               | API、签名   |
| **深度**   | 接口处的杠杆作用——调用者或测试人员每学习一个单位的接口可以执行多少行为。 **深度**：在小的接口后面有许多行为。 **浅度**：接口几乎和实现一样复杂。  | —        |
| **缝合点**  | Michael Feathers 的术语：一个你可以在不编辑该处的情况下改变行为的地方。 *位置*接口的位置，把它放在哪里是它自己的决定，与它后面是什么无关。 | 边界       |
| **适配器**  | 在缝合点满足接口的具体事物。它命名的是一个角色，而不是一种物质——内存中的假数据和 Postgres 仓库都是适配器。                     | —        |
| **杠杆作用** | 调用者从深度中获得的东西：每学习一个单位的接口就能获得更多的能力。                                               | —        |
| **局部性**  | 维护者从深度中获得的东西：变更、错误和验证集中在同一个地方。一次修复，到处修复。                                        | —        |

深度故意不定义为实现行数与接口行数的比率，这是 Ousterhout 自己的定义。那个指标会奖励填充实现。相反使用的是“深度即杠杆作用”的定义。

## 四个原则

* **深度是接口的属性，而不是实现的属性。** 一个深度模块可以在内部由小的可交换部分构建。它们只是不会暴露给调用者。模块可以有内部缝合点供其自己的测试使用，并且在接口处有一个外部缝合点。
* **删除测试。** 想象删除该模块。如果复杂性消失了，它就是一个透传。如果它在 N 个调用者之间重新出现，它就是物有所值的。
* **接口是测试表面。** 调用者和测试穿过同一个缝合点。如果你想要测试接口*之后*的内容，该模块的形状就是错误的。
* **一个适配器意味着一个假设的缝合点。两个适配器意味着一个真实的缝合点。** 在有东西实际穿过它发生变化之前，不要切割缝合点。单适配器的缝合点只是间接引用。

两个支持文件更进一步，该技能按需读取它们而不是预先读取。[DEEPENING.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DEEPENING.md) 对候选者的依赖进行分类——进程内、本地可替换、远程但拥有、真正的外部——因为类别决定了如何跨越缝合点测试深度化的模块。[DESIGN-IT-TWICE.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DESIGN-IT-TWICE.md) 启动并行的[sub-agents](https://www.aihero.dev/ai-coding-dictionary/subagent) 为同一个模块生成三个或更多截然不同的接口，然后根据深度、局部性和缝合点位置对它们进行比较。

## 常见问题

**我实际上如何在 TypeScript 中构建一个深度模块？**

这是关于该技能最常被问到的问题，而该技能并没有回答它。它定义了深度模块*是什么*；它对如何阻止一个随机的导入越过接口到达另一端没有任何说明。[Issue #458](https://github.com/mattpocock/skills/issues/458) 把这一点说得很清楚：“假设我们对接口很满意，它隐藏了细节等等。但我们如何强制执行它？我认为如果没有 linting 或清晰的护栏，人类和 LLM 都会随着时间的推移开始把它弄得一团糟。” Matt 在那个线程中的回答有三个选项：将其包装在类或 IIFE 中并接受类变得极其庞大；在 monorepo 中将其设为一个包并接受 monorepo 工具；或者使用像 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 这样的 linter 来禁止绕过接口的导入。他单独将 Effect 称为最佳机制，将 dependency-cruiser 称为第二好的。仓库的 `in-progress/` 桶中有一个 `setup-ts-deep-modules` 技能，它制定了 `src/packages/<name>/index.ts` 约定，但它是一个 beta 渠道技能，没有文档页面，并且没有附带任何 lint 规则。

**我将一个会话指向它，它烧掉了 10 万 [token](https://www.aihero.dev/ai-coding-dictionary/token)重新设计了我从未问过的事情。**

已知，并作为 [issue #449](https://github.com/mattpocock/skills/issues/449) 提交。该技能是由模型调用的，并将其描述为词汇表，但其中没有任何内容能硬性阻止代理将其视为可运行的流程。被指示“在 /codebase-design 中恢复并推动未完成的决策”时，代理找到了它所能找到的最具行动导向的内容——即 `DESIGN-IT-TWICE.md` 中的并行子代理——重新探索了前一个会话已经映射过的代码，并在提出任何问题之前运行了很长时间。驱动技能的所有护栏（检查点、一次一个问题、不自动前进）在这里都不存在，因为参考技能没有任何护栏。变通方法是命名一个驱动技能并让这个技能位于它之下：使用 `codebase-design` 作为词汇表的 `/grill-with-docs`、`/improve-codebase-architecture` 或 `/tdd`。该问题尚未解决。

**哪里去了 `design-an-interface`\`design-an-interface\` `/interface-design`去了？并且有一个吗**

`design-an-interface` 被移除并合并到这个技能中。没有什么丢失：它的“设计两次”技术——即由并行子代理生成截然不同的设计，源自 Ousterhout——作为 `DESIGN-IT-TWICE.md` 在这里发布。此外，几个人要求有一个专门的 `/interface-design` 技能用于深度模块/薄接口哲学；该哲学已经存在于这里，并且没有计划单独的技能。如果你来寻找这两个名字中的任何一个，这就是这个页面。

**这不是一个文件结构约定——文件夹、桶文件、功能切片吗？**

不，该技能在多次回击中一直坚持这一立场。[Issue #95](https://github.com/mattpocock/skills/issues/95) 提议了一种形式化的分形树文件结构作为深度模块的具体实现；回复是两者是正交的——"深度模块是关于接口的设计以及通过严格的接口访问，无论文件系统看起来如何。使用这种方法似乎完全可以拥有浅模块。" #458 中也出现了同样的情况："我认为你可能将模块的概念与文件系统联系得太紧密。文件系统当然可以作为模块形状的有用提示，但在构建深度模块时没有必要使用文件系统。" 词汇表特意将 **模块** 定义为规模无关。

**\`tdd\` 会取代 \`/implement\`，还是课程的 \`/do-work\`？ `tdd`实际上在使用这个词汇吗？**

现在确实如此。很长一段时间并非如此。以前位于 `tdd` 内部的内联深度模块注释在 v1.0 中被删除，取而代之的是这个共享技能，但替换它们的指针从未被添加——所以 `tdd` 为自己定义了"seam"（缝合点）并且没有引用任何内容。差距已填补：指针现在在技能中，当接口的形状是开放问题而不是测试时被调用。`tdd` 仍然拥有"seam"作为你*测试*的边界；这个技能拥有其背后的模块形状。

**"design-it-twice" 模式在 Claude Code 之外有效吗？**

并不整洁。`DESIGN-IT-TWICE.md` 说"使用 Agent 工具并行生成 3 个以上子代理"，这是 Claude Code 的 [tool](https://www.aihero.dev/ai-coding-dictionary/tool)（工具）。该仓库为其他 [harnesses](https://www.aihero.dev/ai-coding-dictionary/harness)（启动器）发送元数据，包括 Codex，并且这些可能不会在该名称下暴露任何内容——因此并行设计阶段不如技能的元数据所暗示的那样可移植。在 [issue #564](https://github.com/mattpocock/skills/issues/564) 中追踪，未解决。

**我可以把自己的概念添加到词汇表中吗——connascence（相依性）、module secrets（模块秘密），\[progressive disclosure]\(https\://www\.aihero.dev/ai-coding-dictionary/progressive-disclosure)？ [渐进式披露](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)\`/do-work\`？**

人们提出了正是这些。[Issue #180](https://github.com/mattpocock/skills/issues/180) 添加了 Parnas 的模块秘密和 Page-Jones 的相依性作为跨越缝合点泄漏的*什么*的命名层，并附上了工作补丁；[issue #303](https://github.com/mattpocock/skills/issues/303) 提议在实现内部进行渐进式披露，因此在其公共接口处深度模块在其下面不是一个未区分的厚板。两者都是开放且未合并的。作为已发布的词汇表故意很小，它保持很小的原因是该技能本身所述：一致的语言是重点所在，而一个没有人一致使用的术语比没有术语更糟。

## 判断是否生效

* 设计对话不再产出 "component"（组件）、"service"（服务）和 "boundary"（边界）等词汇，而是开始产出 "module"（模块）、"interface"（接口）和 "seam"（缝合点）。
* 任何人都可以指出一个提议的提取项，并说明它是否通过删除测试，而无需含糊其辞。
* 提议的缝合点会附带一个第二个适配器，而不仅仅是第一个。
* 对接口的讨论涵盖了不变量、顺序和错误模式——而不仅仅是类型签名。
* 调用它不会启动会话。如果代理开始读取文件并仅基于 `/codebase-design` 提出重构，它就误将参考作为了驱动。

## 在系统中的位置

`codebase-design` 是一个**随时可用的独立技能**，位于工程技能之下的词汇层，而不是任何链条中的一步。它最近的邻居是 [domain-modeling](https://aihero.dev/skills-domain-modeling)，它是*问题域*的词汇的并行参考，而不是模块的形状——两者通常一起需要，因为要很好地命名深度模块两者都需要。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是另一个：它调查代码库以寻找深化候选者，并将它们中的每一个写入此词汇表，因此它找到模块，而这个技能是你设计它的平台。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指引。
