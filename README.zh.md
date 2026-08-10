<p>
  <a href="https://www.aihero.dev/s/skills-newsletter">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://res.cloudinary.com/total-typescript/image/upload/v1777382277/skills-repo-dark_2x.png">
      <source media="(prefers-color-scheme: light)" srcset="https://res.cloudinary.com/total-typescript/image/upload/v1777382277/skill-repo-light_2x.png">
      <img alt="Skills" src="https://res.cloudinary.com/total-typescript/image/upload/v1777382277/skill-repo-light_2x.png" width="369">
    </picture>
  </a>
</p>

# 真正工程师的技能

[![skills.sh](https://skills.sh/b/mattpocock/skills)](https://skills.sh/mattpocock/skills)

我每天使用的代理技能，用于真正的工程工作，而不是“vibe coding”。

开发真正的应用程序很难。GSD、BMAD 和 Spec-Kit 等方法试图通过掌控流程来提供帮助。但在这样做的同时，它们剥夺了你的控制权，使得流程中的错误难以解决。

这些技能设计得小巧、易于适应且可组合。它们适用于任何模型。它们基于数十年的工程经验。随意折腾它们。让它们成为你自己的。享受吧。

如果你想要跟上这些技能的更新，以及我创建的任何新技能，你可以加入我的通讯，与约 60,000 名其他开发者一起：

[订阅通讯](https://www.aihero.dev/s/skills-newsletter)

## 安装（30秒设置）

有两种方式，两种理念。**[Claude Code 插件](https://code.claude.com/docs/en/plugins)** 将整个集合安装为托管、只读的捆绑包，在我发布时更新——你订阅而不是 fork。**[skills.sh](https://skills.sh/mattpocock/skills)** 将可编辑的技能文件复制到你的项目中，这样你就可以对它们进行黑客攻击并使其成为你自己的。选一个——安装两个会让你拥有每个技能两份。

### 1. 获取技能

<details>
<summary><strong>Claude Code</strong></summary>

```bash
claude plugins install mattpocock-skills
```

或者，在会话内部：

```
/plugin install mattpocock-skills
```

它在 Claude Code 的官方市场里，所以不需要先添加任何东西，更新会自动到来。

</details>

<details>
<summary><strong>Codex, and other agents</strong></summary>

```bash
npx skills@latest add mattpocock/skills
```

选择你想要的技能，以及要在哪个编码代理上安装它们。**安装程序允许你选择要获取哪些技能——确保 `setup-matt-pocock-skills` 在其中之一。**

原生 Codex 插件已在路线图中——请参阅 [`.agents/adr/0002-ship-as-a-claude-code-plugin.md`](./.agents/adr/0002-ship-as-a-claude-code-plugin.md)。

</details>

<details>
<summary><strong>For tinkerers</strong></summary>

在任何代理上使用同一个安装程序，包括 Claude Code：

```bash
npx skills@latest add mattpocock/skills
```

它将技能写入你的仓库，作为你拥有且可以编辑的普通文件。不会在背后偷偷更新；当你想要时，使用 `npx skills update` 拉取我的最新更改。

</details>

### 2. 运行 `/setup-matt-pocock-skills`

在你的代理中，每个仓库运行一次。它将：

* 询问你想要使用哪个问题跟踪器（GitHub、Linear 或本地文件）
* 询问你在进行分类时对票据应用什么标签（`/triage` 使用标签）
* 询问你想把我们要创建的任何文档保存在哪里

### 3. Bam - 你已经准备好了。

## 为什么存在这些技能

我构建这些技能是为了解决我在 Claude Code、Codex 和其他编码代理中看到的常见失败模式。

### #1：代理没有做我想做的事

> “没人确切知道他们想要什么”
>
> David Thomas & Andrew Hunt，《实用程序员》

**问题**。软件开发中最常见的失败模式是错位。你以为开发者知道你想要什么。然后你看到他们构建的东西——然后你意识到它根本不理解你。

在 AI 时代这也是一样的。你和代理之间存在沟通差距。解决这个问题的方法是**质询环节**——让代理问你关于你在构建的细节问题。

**解决方案**是使用：

* [`/grill-me`](./skills/productivity/grill-me/SKILL.md) - 用于非代码用途
* [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md) - 与 [`/grill-me`](./skills/productivity/grill-me/SKILL.md) 相同，但添加了更多好东西（见下文）

这些是我最受欢迎的技能。它们帮助你在开始之前与代理对齐，并深入思考你正在进行的更改。每次你想进行更改时都使用它们。

### #2：代理太啰嗦了

> 有了通用语言，开发人员之间的对话和代码的表达都源于同一个领域模型。
>
> Eric Evans，《领域驱动设计》

**问题**：在项目开始时，开发人员以及他们为其构建软件的人（领域专家）通常使用不同的语言。

我对代理也有同样的感觉。代理通常被扔进一个项目中，并被要求在过程中弄清楚行话。所以他们用 20 个词，而一个词就够了。

解决这个问题的方法是共享语言。这是一份帮助代理解码项目中使用的行话的文档。

<details>
<summary>
Example
</summary>

这是我的 `course-video-manager` 仓库中的示例 [`CONTEXT.md`](https://github.com/mattpocock/course-video-manager/blob/076a5a7a182db0fe1e62971dd7a68bcadf010f1c/CONTEXT.md)。哪一个更容易阅读？

* **之前**：“当课程章节内的课程内容变为‘真实’（即赋予文件系统中的位置）时会出现一个问题”
* **之后**：“存在实例化级联问题”

这种简洁在每次会话中都大放异彩。

</details>

这已内置在 [`/grill-with-docs`](./skills/engineering/grill-with-docs/SKILL.md) 中。这是一种质询环节，但它帮助你与 AI 建立共享语言，并在 ADR 中记录难以解释的决策。

很难解释这有多强大。它可能是这个仓库中唯一最酷的技术。试试看，你就会明白。

> [!TIP]
> 共享语言除了减少冗长之外还有很多其他好处：
>
> * **变量、函数和文件使用共享语言命名一致**
> * 结果是，**代码库对代理来说更容易导航**
> * 代理在思考上也**消耗更少的 token**，因为它可以使用更简洁的语言

### #3：代码不工作

> “总是采取小而深思熟虑的步骤。反馈率是你的速度限制。永远不要承担太大的任务。”
>
> David Thomas & Andrew Hunt，《实用程序员》

**问题**：假设你和代理在要构建什么上达成了一致。当代理*仍然*产生垃圾时会发生什么？

是时候看看你的反馈循环了。如果没有关于代码实际运行情况的反馈，代理将盲目飞行。

**解决方案**：你需要通常的一套反馈循环：静态类型、浏览器访问和自动化测试。

对于自动化测试，红绿重构循环至关重要。这是代理先编写一个失败的测试，然后修复测试的地方。这有助于给代理提供一致的反馈水平，从而产生更好的代码。

我构建了一个 **[`/tdd`](./skills/engineering/tdd/SKILL.md) 技能**，可以插入任何项目。它鼓励红绿重构，并给代理关于什么构成好测试和坏测试的充分指导。

对于调试，我还构建了一个 **[`/diagnosing-bugs`](./skills/engineering/diagnosing-bugs/SKILL.md)** 技能，它将最佳调试实践封装到一个有纪律的循环中，分阶段把关。

### #4：我们建立了一个烂摊子

> “每天投入系统的设计。”
>
> Kent Beck，《极限编程解释》

> “最好的模块是深入的。它们允许通过简单接口访问大量功能。”
>
> John Ousterhout，《软件设计哲学》

**问题**：大多数使用代理构建的应用都很复杂且难以更改。由于代理可以极大地加快编码速度，它们也加速了软件熵的增加。代码库正以前所未有的速度变得越来越复杂。

**解决方案**是采用一种全新的 AI 驱动开发方法：关注代码的设计。

这是内置于这些技能的每一层：

* [`/to-spec`](./skills/engineering/to-spec/SKILL.md) 在创建规范之前，会询问你正在接触哪些模块。

而关键的是，[`/improve-codebase-architecture`](./skills/engineering/improve-codebase-architecture/SKILL.md) 会扫描代码库以寻找深化机会，并将候选方案交给你。我建议每隔几天在你的代码库上运行一次。它是一次调查，不是救援：在真正古老的代码库上，它会找到真正的候选方案，但不会为你理清烂摊子。

### 摘要

软件工程基础比以往任何时候都重要。这些技能是我将基础原理浓缩为可重复实践的最佳尝试，旨在帮助你交付职业生涯中最好的应用。请享受。

## 参考

这些根据一个轴进行划分——谁可以调用它们。**用户调用**的技能只有在你输入它们时（例如 `/grill-me`）才能访问；它们的工作是编排。**模型调用**的技能可以由你调用，或者在任务适合时由代理自动调用；它们持有可重用的纪律。用户调用的技能可以调用模型调用的技能，但绝不会调用另一个用户调用的技能。

### 工程

我日常用于代码工作的技能。

**用户调用**

* **[ask-matt](./skills/engineering/ask-matt/SKILL.md)** — 询问哪种技能或流程适合你的情况。本仓库中用户调用技能的路由器。
* **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)** — 审问会议，同时也构建你的项目的领域模型，完善术语并内联更新 `CONTEXT.md` 和 ADR。
* **[triage](./skills/engineering/triage/SKILL.md)** — 将问题在分类角色的状态机中流转。
* **[improve-codebase-architecture](./skills/engineering/improve-codebase-architecture/SKILL.md)** — 扫描代码库以寻找改进机会，以可视化的 HTML 报告呈现，然后针对你选择的任意一个进行审问。
* **[setup-matt-pocock-skills](./skills/engineering/setup-matt-pocock-skills/SKILL.md)** — 为工程技能配置此仓库（问题跟踪器、分类标签、领域文档布局）。在使用其他工程技能之前，每个仓库运行一次。
* **[to-spec](./skills/engineering/to-spec/SKILL.md)** — 将当前对话转化为规范并发布到问题跟踪器。没有面试——只是综合你已经讨论过的内容。
* **[to-tickets](./skills/engineering/to-tickets/SKILL.md)** — 将任何计划、规范或对话分解为一组追踪子弹票据，每个票据声明其阻塞边界——写成本地文件中的文本，或作为真实跟踪器上的原生阻塞链接。
* **[implement](./skills/engineering/implement/SKILL.md)** — 构建由规范或一组票据描述的工作，在预先商定的接缝处驱动 `/tdd`，并在提交前以 `/code-review` 结束。
* **[wayfinder](./skills/engineering/wayfinder/SKILL.md)** — 规划一大块工作，这是单个代理会话无法容纳的，将其作为问题跟踪器上决策票据的共享地图——逐个解决它们，直到通往目的地的路径清晰。

**模型调用**

* **[prototype](./skills/engineering/prototype/SKILL.md)** — 构建一个一次性原型来回答设计问题——一个用于状态/逻辑问题的可共享 HTML 文件，或者多个可以从一个路由切换的截然不同的 UI 变体。
* **[diagnosing-bugs](./skills/engineering/diagnosing-bugs/SKILL.md)** — 针对疑难 Bug 和性能回归的严格诊断循环：构建一个针对此 Bug 变红的反馈循环 → 最小化 → 假设 → 插入监控 → 修复 → 回归测试。
* **[research](./skills/engineering/research/SKILL.md)** — 针对高信任度的原始来源调查一个问题，并将发现作为仓库中的引用 Markdown 文件记录下来，作为后台代理运行。
* **[tdd](./skills/engineering/tdd/SKILL.md)** — 带有红绿重构循环的测试驱动开发。一次构建一个垂直切片的功能或修复一个 Bug。
* **[domain-modeling](./skills/engineering/domain-modeling/SKILL.md)** — 主动构建并完善项目的领域模型——对照术语表验证术语，使用边缘情况场景进行压力测试，并内联更新 `CONTEXT.md` 和 ADR。
* **[codebase-design](./skills/engineering/codebase-design/SKILL.md)** — 用于设计深层模块的共享纪律和词汇：在小接口背后封装大量行为，放置在干净的接缝处，并通过该接口进行测试。
* **[code-review](./skills/engineering/code-review/SKILL.md)** — 对自固定点以来的差异进行双轴审查：**标准**（它是否遵循仓库的编码标准，加上一个 Fowler 气味基准？）和**规范**（它是否忠实地实现原始问题/规范？），作为并行子代理运行，这样都不会污染对方。
* **[resolving-merge-conflicts](./skills/engineering/resolving-merge-conflicts/SKILL.md)** — 逐个解决进行中的 git 合并或变基冲突块，通过追溯到每一方原始来源的意图来解决，然后完成操作——决不 `--abort`。
* **[wizard](./skills/engineering/wizard/SKILL.md)** — 生成一个交互式 bash 向导，引导人类完成只有他们能执行的步骤：配置基础设施，设置凭据或 CI 密钥，浏览不熟悉的三方仪表板，或运行一次性迁移或切换。

### 生产力

通用工作流工具，不针对特定代码。

**用户调用**

* 不断接受关于计划或设计的采访，直到设计树的所有分支都得到解决。
* **[handoff](./skills/productivity/handoff/SKILL.md)** — 将当前对话精简为移交文档，以便另一个代理可以继续工作。
* **[teach](./skills/productivity/teach/SKILL.md)** — 在多个会话中教用户一个新技能或概念，使用当前目录作为有状态的教学生态工作空间。
* 将你无法独自回答的决定转化为一个 Markdown 问卷，给那个能回答的人——可以异步填写，也可以在会议中一起填写。它会追问你关于发送细节（是给谁、你需要什么），而不是主题本身。
* 一旦消息没有投递成功，立即触发此技能。代理会利用你缺失的上下文，用你的 `CONTEXT.md` 词汇，用通俗易懂的英语重新推介该消息。

**模型调用**

* 不间断地采访用户关于计划、决定或想法，直到设计树的所有分支都得到解决。这是 `grill-me`、`grill-with-docs`、`triage`、`wayfinder` 和 `improve-codebase-architecture` 背后的可重用采访原语。
* 为代理撰写文档：技能、AGENTS.md/CLAUDE.md 以及代理通过指针指向的任何文档。
