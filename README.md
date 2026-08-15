<p>
  <a href="https://www.aihero.dev/s/skills-newsletter">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://res.cloudinary.com/total-typescript/image/upload/v1777382277/skills-repo-dark_2x.png">
      <source media="(prefers-color-scheme: light)" srcset="https://res.cloudinary.com/total-typescript/image/upload/v1777382277/skill-repo-light_2x.png">
      <img alt="Skills" src="https://res.cloudinary.com/total-typescript/image/upload/v1777382277/skill-repo-light_2x.png" width="369">
    </picture>
  </a>
</p>

# 面向真实工程师的技能

[![skills.sh](https://skills.sh/b/mattpocock/skills)](https://skills.sh/mattpocock/skills)

我每天用来做真实工程——而不是“氛围编程”——的智能体技能。

开发真实应用是困难的。GSD、BMAD 和 Spec-Kit 等方法试图通过掌控流程来提供帮助。但这样做会剥夺你的控制权，并使流程中的缺陷难以解决。

这些技能被设计得小巧、易于调整且可组合。它们适用于任何模型。它们基于数十年的工程经验。随意改造它们，让它们成为你自己的。尽情享受。

如果你想随时了解这些技能的更新以及我创建的任何新技能，可以在我的通讯中与约 6 万名开发者一起订阅：

[订阅通讯](https://www.aihero.dev/s/skills-newsletter)

## 安装（30 秒设置）

两种方式，两种理念。**[Claude Code 插件](https://code.claude.com/docs/en/plugins)** 将整套技能作为受管理的只读捆绑包安装，在我发布时自动更新——你订阅而不是分叉。**[skills.sh](https://skills.sh/mattpocock/skills)** 会将可编辑的技能文件复制到你的项目中，因此你可以对它们进行改造并据为己有。选一个——如果两个都装，每个技能都会出现两次。

### 1. 获取技能

<details>
<summary><strong>Claude Code</strong></summary>

```bash
claude plugins install mattpocock-skills
```

或者在会话内部：

```
/plugin install mattpocock-skills
```

它位于 Claude Code 的官方市场中，因此无需预先添加任何内容，更新会自动到达。

</details>

<details>
<summary><strong>Codex, and other agents</strong></summary>

```bash
npx skills@latest add mattpocock/skills
```

选择你想要的技能，以及要将它们安装到哪些编码智能体上。**安装程序允许你选择要安装哪些技能——请确保 `setup-matt-pocock-skills` 是其中之一。**

原生 Codex 插件已在路线图上——请参阅 [`.agents/adr/0002-ship-as-a-claude-code-plugin.md`](./.agents/adr/0002-ship-as-a-claude-code-plugin.md)。

</details>

<details>
<summary><strong>For tinkerers</strong></summary>

在任何智能体上使用相同的安装程序——包括 Claude Code：

```bash
npx skills@latest add mattpocock/skills
```

它会将技能以普通文件的形式写入你的仓库，这些文件归你所有且可编辑。不会有任何东西在你不知情的情况下更新；当你需要时，可以使用 `npx skills update` 拉取我的最新更改。

</details>

### 2. 运行 `/setup-matt-pocock-skills`

在你的智能体中，对每个仓库运行一次。它将：

* 询问你想使用哪个问题追踪器（GitHub、Linear 或本地文件）
* 询问你在对工单进行分类时应用哪些标签（`/triage` 使用标签）
* 询问你想将我们创建的任何文档保存到哪里

### 3. 搞定——你就准备好了。

## 为什么会有这些技能

我构建这些技能是为了修复我在 Claude Code、Codex 和其他编码智能体上看到的常见失败模式。

### #1：智能体没有按照我的意愿行事

> “没有人确切地知道自己想要什么”
>
> 戴维·托马斯与安德鲁·亨特，[《程序员修炼之道》](https://www.amazon.co.uk/Pragmatic-Programmer-Anniversary-Journey-Mastery/dp/B0833F1T3V)

**问题所在**。软件开发中最常见的失败模式是目标不一致。你以为开发者知道你想要什么。然后你看到他们构建的东西——才意识到它完全没有理解你。

在 AI 时代也是如此。你和智能体之间存在着沟通鸿沟。解决这个问题的方法是**盘问环节**——让智能体就你正在构建的内容向你提出详细的问题。

**解决方案**是使用：

* [`/grill-me`](./skills/productivity/grill-me/SKILL.md) - 用于非代码用途
* [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md) - 与 [`/grill-me`](./skills/productivity/grill-me/SKILL.md) 相同，但增加了更多好处（见下文）

这些是我最受欢迎的技能。它们帮助你在开始之前与智能体对齐，并深入思考你要做出的更改。每当你想要进行更改时，都要使用它们。

### #2：智能体过于啰嗦

> 借助一种通用语言，开发者之间的对话以及代码的表达都源自同一个领域模型。
>
> 埃里克·埃文斯，[《领域驱动设计》](https://www.amazon.co.uk/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215)

**问题所在**：在项目开始时，开发者与他们为其构建软件的人（领域专家）通常说着不同的语言。

我对我的智能体也有同样的感受。智能体通常被扔进一个项目，并被要求边走边弄懂行话。所以它们会用一个词就能说清的地方用 20 个词。

**解决方案**是建立一种共享语言。这是一份帮助智能体解读项目中使用的行话的文档。

<details>
<summary>
Example
</summary>

这是我 `course-video-manager` 仓库中的一个 [`CONTEXT.md`](https://github.com/mattpocock/course-video-manager/blob/076a5a7a182db0fe1e62971dd7a68bcadf010f1c/CONTEXT.md) 示例。哪一个更容易阅读？

* **之前**：“当课程某个部分中的一课被‘实体化’（即在文件系统中获得一个位置）时，就会出现问题。”
* **之后**：“物化级联存在问题。”

这种简洁性在一次又一次的会话中都会得到回报。

</details>

这已内置于 [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md) 中。它是一次盘问环节，但能帮助你与 AI 建立共享语言，并将难以解释的决策记录在 ADR 中。

这很难解释它有多强大。它可能是这个仓库中最酷的一项技术。试试看吧。

> [!TIP]
> 共享语言除了减少啰嗦之外，还有许多其他好处：
>
> * **变量、函数和文件的命名保持一致**，使用共享语言
> * 因此，对智能体来说，**代码库更容易导航**
> * 智能体也会**花费更少的 token 来思考**，因为它可以使用更简洁的语言

### #3：代码无法工作

> “始终采取小而慎重的步骤。反馈的速度就是你的速度上限。永远不要承担过大的任务。”
>
> 戴维·托马斯与安德鲁·亨特，[《程序员修炼之道》](https://www.amazon.co.uk/Pragmatic-Programmer-Anniversary-Journey-Mastery/dp/B0833F1T3V)

**问题所在**：假设你和智能体在要构建什么上已经达成一致。当智能体*仍然*产出糟糕的东西时会发生什么？

是时候审视你的反馈循环了。如果对智能体生成的代码实际运行情况没有反馈，智能体就像在黑暗中飞行。

**解决方案**：你需要常规的那一套反馈循环：静态类型、浏览器访问和自动化测试。

对于自动化测试，红-绿-重构循环至关重要。这是智能体先编写一个失败的测试，然后修复该测试的过程。这有助于为智能体提供一致的反馈水平，从而产出更好的代码。

我构建了一个 **[`/tdd`](./skills/engineering/tdd/SKILL.md) 技能**，你可以将其插入任何项目。它鼓励红-绿-重构，并为智能体提供大量关于什么构成好测试和坏测试的指导。

对于调试，我还构建了一个 **[`/diagnosing-bugs`](./skills/engineering/diagnosing-bugs/SKILL.md)** 技能，它将最佳调试实践封装到一个纪律严明的循环中，并逐阶段进行门控。

### #4：我们构建了一个泥球

> “*每天*都投资于系统的设计。”
>
> 肯特·贝克，[《解析极限编程》](https://www.amazon.co.uk/Extreme-Programming-Explained-Embrace-Change/dp/0321278658)

> “最好的模块是深层的。它们允许通过一个简单的接口访问大量功能。”
>
> 约翰·奥斯特豪特，[《软件设计哲学》](https://www.amazon.co.uk/Philosophy-Software-Design-2nd/dp/173210221X)

**问题**：大多数用智能体构建的应用都很复杂且难以修改。因为智能体能从根本上加速编码，它们也加速了软件熵。代码库以空前的速度变得更加复杂。

**解决方案**是一种激进的全新 AI 驱动开发方法：关注代码的设计。

这一点内置于这些技能的每一个层面：

* [`/to-spec`](./skills/engineering/to-spec/SKILL.md) 在创建规格之前，会询问你正在接触哪些模块

关键是，[`/improve-codebase-architecture`](./skills/engineering/improve-codebase-architecture/SKILL.md) 会扫描代码库寻找深化机会，并把候选清单交给你。我建议每隔几天就在你的代码库上运行一次。它是一份调查，而不是拯救：在一个真正老旧的代码库上，它会找到真正的候选对象，但它不会帮你理清那团乱麻。

### 摘要

软件工程基础比以往任何时候都更重要。这些技能是我将这类基础提炼为可重复实践的最大努力，旨在帮助你交付职业生涯中最好的应用。祝愉快。

## 参考

这些技能按一个维度划分——谁能调用它们。**用户调用**的技能只有在你输入它们时才能使用（例如 `/grill-me`）；它们的工作是编排。**模型调用**的技能可以由你调用，*或者*当任务匹配时由智能体自动获取；它们承载着可复用的纪律。用户调用技能可以调用模型调用技能，但绝不能调用另一个用户调用技能。

### 工程

我日常处理代码工作所用的技能。

**用户调用**

* **[ask-matt](./skills/engineering/ask-matt/SKILL.md)** — 询问哪个技能或流程适合你的情况。本仓库中用户调用技能的路由器。
* **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)** — 盘问环节，同时构建你项目的领域模型，锐化术语，并内联更新 `CONTEXT.md` 和 ADR。
* **[triage](./skills/engineering/triage/SKILL.md)** — 通过一系列分诊角色的状态机来推进问题。
* **[improve-codebase-architecture](./skills/engineering/improve-codebase-architecture/SKILL.md)** — 扫描代码库中的深化机会，将其呈现为可视化的 HTML 报告，然后针对你挑选的任何一项进行盘问。
* **[setup-matt-pocock-skills](./skills/engineering/setup-matt-pocock-skills/SKILL.md)** — 为工程技能配置本仓库（问题跟踪器、分诊标签、领域文档布局）。在开始使用其他工程技能之前，每个仓库运行一次。
* **[to-spec](./skills/engineering/to-spec/SKILL.md)** — 将当前对话转换为规格，并发布到问题跟踪器。无需访谈——只需综合你们已经讨论过的内容。
* **[to-tickets](./skills/engineering/to-tickets/SKILL.md)** — 将任何计划、规格或对话拆分为一组曳光弹式票证，每张票证都声明其阻塞边界——写入本地文件中的文本，或作为真实跟踪器上的原生阻塞链接。
* **[implement](./skills/engineering/implement/SKILL.md)** — 构建由规格或一组票证描述的工作，在预先商定的接缝处驱动 `/tdd`，并在提交前以 `/code-review` 收尾。
* **[wayfinder](./skills/engineering/wayfinder/SKILL.md)** — 规划一项庞大的工作，超过单个智能体会话所能容纳的范围，作为问题跟踪器上决策票证的共享地图——逐个解决，直到通往目的地的道路清晰。

**模型调用**

* **[prototype](./skills/engineering/prototype/SKILL.md)** — 构建一个一次性原型来回答设计问题——对于状态/逻辑问题，是一个可共享的 HTML 文件；或者多个截然不同的 UI 变体，可从一条路由切换。
* **[diagnosing-bugs](./skills/engineering/diagnosing-bugs/SKILL.md)** — 针对棘手的 bug 和性能回归的有纪律的诊断循环：构建一个反馈循环，让这个 bug 变红 → 最小化 → 假设 → 插桩 → 修复 → 回归测试。
* **[research](./skills/engineering/research/SKILL.md)** — 针对高可信度的一手来源调查一个问题，并将发现捕获为仓库中带有引用的 Markdown 文件，作为后台智能体运行。
* **[tdd](./skills/engineering/tdd/SKILL.md)** — 带有红-绿-重构循环的测试驱动开发。一次构建一个垂直切片的功能或修复 bug。
* **[domain-modeling](./skills/engineering/domain-modeling/SKILL.md)** — 主动构建并锐化项目的领域模型——对照术语表质疑术语，用边界场景进行压力测试，并内联更新 `CONTEXT.md` 和 ADR。
* **[codebase-design](./skills/engineering/codebase-design/SKILL.md)** — 设计深模块的共享纪律和词汇：在小型接口背后承载大量行为，放置在干净的接缝处，并可透过该接口进行测试。
* **[code-review](./skills/engineering/code-review/SKILL.md)** — 对固定点以来的差异进行双轴评审：**标准**（它是否符合仓库的编码标准，加上 Fowler 坏味道基线？）和**规格**（它是否忠实地实现了来源问题/规格？），并作为并行子智能体运行，以免相互污染。
* **[resolving-merge-conflicts](./skills/engineering/resolving-merge-conflicts/SKILL.md)** — 逐块处理进行中的 git merge 或 rebase 冲突，通过追溯到每一侧的一手来源的意图来解决，然后完成操作——绝不 `--abort`。
* **[wizard](./skills/engineering/wizard/SKILL.md)** — 生成一个交互式 bash 向导，引导人类完成只有他们才能执行的步骤：配置基础设施、设置凭据或 CI 机密、浏览不熟悉的第三方仪表盘，或运行一次性迁移或切换。

### 生产力

通用工作流工具，不特定于代码。

**用户调用**

* **[grill-me](./skills/productivity/grill-me/SKILL.md)** — 持续接受关于计划或设计的盘问，直到设计树的每个分支都被解决。
* **[handoff](./skills/productivity/handoff/SKILL.md)** — 将当前对话压缩成一份交接文档，以便另一个智能体可以继续这项工作。
* **[teach](./skills/productivity/teach/SKILL.md)** — 在多次会话中教用户一项新技能或概念，使用当前目录作为有状态的教学工作区。
* **[to-questionnaire](./skills/productivity/to-questionnaire/SKILL.md)** — 将你无法独自回答的决策转化为一份 Markdown 问卷，交给唯一能回答的人——异步填写，或在一场会议中共同完成。它会盘问你关于发送对象（给谁、你需要什么反馈），而不是主题。
* **[wait-what](./skills/productivity/wait-what/SKILL.md)** — 一旦某条消息没被理解，立即触发。智能体会用你缺失的上下文，以你 `CONTEXT.md` 词汇表中的平实语言重新阐述。

**模型调用**

* **[grilling](./skills/productivity/grilling/SKILL.md)** — 持续盘问用户关于计划、决策或想法，直到设计树的每个分支都被解决。这是 `grill-me`、`grill-with-docs`、`triage`、`wayfinder` 和 `improve-codebase-architecture` 背后可复用的访谈原语。
* **[writing-for-agents](./skills/productivity/writing-for-agents/SKILL.md)** — 为智能体编写文档：技能、AGENTS.md/CLAUDE.md，以及智能体通过指针触达的任何文档。
