## 它做什么

`codebase-design` 界定了你设计模块时所用的词汇：**module**（模块）、**interface**（接口）、**depth**（深度）、**seam**（接缝）、**adapter**（适配器）、**leverage**（杠杆）、**locality**（局部性）。它精确地定义每一个词，禁用松散的替代词（“component”、“service”、“API”、“boundary”），并阐明由它们得出的那几条原则。

它是一份参考资料，而不是一个流程。没有需要运行的循环，没有它产生的产物，也没有它会向你提问的检查点。每个涉及设计的其他技能都会借用它的词汇；单独使用时，它把语言交给你，然后就此打住。这是你在调用它之前需要知道的事，因为一个没有流程、没有停止规则的技能，如果你把一个 [session](https://www.aihero.dev/ai-coding-dictionary/session) 指向它并说“开始”，它会即兴编造一个流程——见下面的问题。

## 何时使用它

输入 `/codebase-design`，或者当某个设计任务适用时，agent 会自动调用它。

当你已经知道要重新设计哪些代码，并且需要思考它的形态时，就使用它：接缝放在哪里、接口能有多小、一次抽取是否值得保留。要解决关于某个词含义的争论时，你也会用到它。

有几个技能和它靠得很近。你想要哪一个取决于实际的问题是什么：

| 问题                                                | 技能                                                                                                  |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| 单个模块的形态——它的接口、它的接缝、它的深度                           | `codebase-design`                                                                                   |
| 该 *领域的词汇*——“account”有三种含义，两个人对“cancellation”的理解不同 | [domain-modeling](https://aihero.dev/skills-domain-modeling)                                        |
| 你还不知道 *哪*个模块需要重新设计                                | [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture)——找出候选模块的调研 |
| 你希望设计接受质疑，而不只是被命名                                 | [追问](https://aihero.dev/skills-grilling)                                                            |
| 有一个具体的行为要构建，并且你希望测试能在重构中幸存下来                      | [tdd](https://aihero.dev/skills-tdd)                                                                |

## 词汇表

词汇表就是这个技能。每个术语都相对于其他术语来定义，而且每个都带有它所替代的词。

| 术语                | 含义                                                                                         | 不要使用                             |
| ----------------- | ------------------------------------------------------------------------------------------ | -------------------------------- |
| **模块（Module）**    | 任何具有接口和实现的东西。刻意与规模无关——可以是一个函数、一个类、一个包、一个跨层的切片。                                             | unit、component、service（单元、组件、服务） |
| **接口（Interface）** | 调用者为了正确使用它而必须知道的一切：类型签名，外加不变量、顺序约束、错误模式、必需的配置、性能特征。                                        | API、signature（签名）                |
| **深度（Depth）**     | 接口处的杠杆——调用者或测试每学习一个单位的接口，就能驱动多少行为。 **深（Deep）**：大量行为隐藏在一个小接口背后。 **浅（Shallow）**：接口几乎与实现一样复杂。 | —                                |
| **接缝（Seam）**      | Michael Feathers 的术语：一个无需在该处编辑就能改变行为的地方。它是接口的 *位置*，而把它放在哪里本身就是独立的决策，与它背后放什么分开考虑。           | boundary（边界）                     |
| **适配器（Adapter）**  | 在接缝处满足接口的具体事物。它命名的是一个角色，而不是一种实体——内存假实现和 Postgres 仓库都是适配器。                                  | —                                |
| **杠杆（Leverage）**  | 调用者从深度中获得的东西：每学习一单位接口，就获得更多能力。                                                             | —                                |
| **局部性（Locality）** | 维护者从深度中获得的东西：变更、缺陷和验证集中在一处。修复一次，处处生效。                                                      | —                                |

深度特意*不*被定义为实现代码行数与接口代码行数之比——尽管那是 Ousterhout 本人的定义。那个指标会奖励给实现注水的行为。取而代之的是“深度即杠杆”（depth-as-leverage）的定义。

## 四条原则

* \*\*深度是接口的属性，不是实现的属性。\*\*一个深模块内部可以由可替换的小部件构建而成。它们只是不会向调用者显现。一个模块可以有供自身测试使用的内部接缝，以及一个位于其接口上的外部接缝。
* \*\*删除测试。\*\*想象删除这个模块。如果复杂性随之消失，那它就是个透传。如果复杂性重新出现在 N 个调用者身上，那它就是在体现自己的价值。
* \*\*接口就是测试表面。\*\*调用者和测试穿过同一个接缝。如果你想测试*越过*接口的东西，那么这个模块的形状就是错的。
* \*\*一个适配器意味着假设的接缝；两个适配器意味着真实的接缝。\*\*在确实有东西跨越它变化之前，不要切出接缝。单一适配器的接缝只是间接层。

两个辅助文件更进一步，技能按需读取它们，而不是预先读取。[DEEPENING.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DEEPENING.md) 对候选模块的依赖进行分类——进程内、本地可替换、远程但自有、真正外部——因为类别决定了加深后的模块如何跨越接缝进行测试。[DESIGN-IT-TWICE.md](https://github.com/mattpocock/skills/blob/main/skills/engineering/codebase-design/DESIGN-IT-TWICE.md) 会启动并行的[子智能体](https://www.aihero.dev/ai-coding-dictionary/subagent)，为同一个模块生成三个或更多截然不同的接口，然后比较它们的深度、局部性和接缝位置。

## 常见问题

**我到底如何在 TypeScript 中构建一个深模块？**

这是关于这个技能被问得最多的问题，而这个技能并没有回答它。它定义了深模块*是什么*，却完全没有提到如何阻止游离的导入越过接口。[Issue #458](https://github.com/mattpocock/skills/issues/458) 说得直白：“假设我们对接口感到满意，它隐藏了细节，等等。但我们如何强制实施它？我认为如果没有 lint 或清晰的护栏，人类和 LLM 都会随着时间推移开始把它弄得一团糟。”Matt 在那个讨论串中的回答是三个选项：把它包在一个 class 或 IIFE 里，并接受这个类会变得巨大；把它做成 monorepo 中的一个包，并接受 monorepo 的工具链；或者使用像 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) 这样的 linter，禁止绕过接口的导入。他另外还分别称 Effect 是最好的机制，dependency-cruiser 是第二好的。仓库的 `in-progress/` 目录中有一个 `setup-ts-deep-modules` 技能，它定下了一种 `src/packages/<name>/index.ts` 约定，但它是一个没有文档页面的测试版技能，而且没有随附任何 lint 规则。

**我把一个 session 指向它，结果它烧掉了 10 万 [tokens](https://www.aihero.dev/ai-coding-dictionary/token)重新设计我从未要求过的东西。**

已知问题，已作为 [issue #449](https://github.com/mattpocock/skills/issues/449) 归档。这个技能由模型调用，并将自己描述为词汇表，但其中没有任何东西能硬性阻止智能体把它当作一个可运行的流程。当被告知“在 /codebase-design 中继续，并推动尚未决定的决策”时，一个智能体抓住了它能找到的最具行动形态的内容——`DESIGN-IT-TWICE.md` 中的并行子智能体——重新探索了先前会话已经摸清过的代码，并且在问任何问题之前跑出了很远。驱动型技能所拥有的护栏（检查点、一次只问一个问题、不自动推进）在这里一个都没有，因为参考资料本来就没有护栏。变通办法是指定一个驱动技能，让这个技能作为它的底层：`/grill-with-docs`、`/improve-codebase-architecture` 或 `/tdd`，以 `codebase-design` 作为词汇表。该 issue 仍然开放。

**把 `design-an-interface`移到哪去了？还有 `/interface-design`这个技能吗？**

`design-an-interface` 已被移除并并入这个技能。没有任何损失：它的“设计两次”技术——由并行子智能体生成截然不同的设计，源自 Ousterhout——以 `DESIGN-IT-TWICE.md` 的形式包含在这里。另外，有几个人曾要求为深模块/薄接口的哲学专门做一个 `/interface-design` 技能；那种哲学已经存在于这里，而且没有计划另立技能。如果你是为了这两个名字中的任何一个而来，就是这一页。

**这难道不是一种文件结构约定吗——文件夹、桶文件、特性切片？**

不，而且该技能在反复的反对声中一直坚守这一立场。[Issue #95](https://github.com/mattpocock/skills/issues/95) 提议将形式化的分形树文件结构作为深模块的具体实现；当时的回复是，这两者是正交的——"深模块关乎接口的设计，以及通过严格接口进行访问，无论文件系统长什么样。使用这种方法似乎完全有可能出现浅模块。"同样的问题在 #458 中也出现了："我认为你可能把模块的概念与文件系统绑得太紧了。文件系统当然可以作为模块形状的有用提示，但在构建深模块时没有必要使用文件系统。"词汇表特意将 **module** 定义为规模无关（scale-agnostic）的。

**&#x20;`tdd`真的使用这套词汇吗？**

现在是了。但很长一段时间内并非如此。曾经放在 `tdd` 内部的内联深模块笔记在 v1.0 中被移除，转而使用这个共享技能，但替换它们的指针从未被添加——所以 `tdd` 为自己定义了 "seam"（接缝），并且没有引用任何东西。现在这个缺口已经补上：指针现在就放在 `tdd` 中，当开放问题是接口的形状而非测试时，就会触达这里。`tdd` 仍然拥有 "seam" 这个词，指你*进行测试*所针对的边界；而这个技能拥有其背后的模块形状。

**design-it-twice 模式在 Claude Code 之外也能工作吗？**

不能干净利落地工作。`DESIGN-IT-TWICE.md` 说"使用 Agent 工具并行生成 3 个以上的子代理"，这里的 Agent 工具是 Claude Code 的 [tool](https://www.aihero.dev/ai-coding-dictionary/tool)，用的是 Claude Code 自己的名称。该仓库为其他 [harnesses](https://www.aihero.dev/ai-coding-dictionary/harness)（包括 Codex）提供了元数据，而那些工具在该名称下可能什么都不会暴露——所以并行设计阶段的便携性不如技能元数据所暗示的那么好。这个问题已在 [issue #564](https://github.com/mattpocock/skills/issues/564) 中跟踪，尚未关闭。

**我可以把自己的概念加入词汇表吗——connascence（共变）、module secrets（模块秘密）、 [progressive disclosure（渐进式披露）](https://www.aihero.dev/ai-coding-dictionary/progressive-disclosure)？**

人们确实提议过这些。[Issue #180](https://github.com/mattpocock/skills/issues/180) 将 Parnas 的 module secrets（模块秘密）和 Page-Jones 的 connascence（共变）作为命名层加入，用来说明*什么*正在穿过接缝泄漏，并附带了可用的 diff；[issue #303](https://github.com/mattpocock/skills/issues/303) 提议在实现内部采用 progressive disclosure（渐进式披露），这样在公共接口上是深模块的模块，其内部就不会是一整块无从区分的平板。两者都处于开放状态，尚未合并。随附的词汇表刻意保持很小，而它保持很小的原因在技能本身中就有说明：一致的语言才是全部意义所在，一个没人能一致使用的术语比没有术语更糟。

## 如果它起作用了

* 设计对话不再产出 "component"、"service" 和 "boundary" 这些词，而是开始产出 "module"、"interface" 和 "seam"。
* 有人能指着提议的抽取方案，毫不犹豫地说出它是否通过删除测试。
* 提议的接缝要附带第二个已命名的适配器，而不只是第一个。
* 对接口的讨论涵盖不变量、顺序和错误模式——而不仅仅是类型签名。
* 调用它不会启动会话。如果 agent 单凭 `/codebase-design` 就开始读取文件并提出重构，那就是把参考文档误当成了驱动者。

## 它在系统中的位置

`codebase-design` 是一个**随时可用的独立技能**，是工程技能之下的词汇层，而不是任何链条中的一步。它最接近的邻居是 [domain-modeling](https://aihero.dev/skills-domain-modeling)，那是*问题域*词汇的平行参考，而非模块的形状——两者通常需要一起使用，因为要给深模块取好名字，两者都需要。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 是另一个：它会勘察代码库以寻找值得加深的候选模块，并用这套词汇表逐一写出，因此它负责找到模块，而这个技能是你设计模块时的工作台。当你不确定哪个技能或流程合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
