# 编写文档页面

`engineering/` 和 `productivity/` 中的每个技能都在 `docs/<bucket>/<skill-name>.md` 处有一个面向人类的 **docs 页面** —— 文档树在 `skills/` 下镜像了这两个桶文件夹。它在 `https://aihero.dev/skills-<skill-name>` 上发布；URL 始终是 `skills-<skill-name>`，无论桶如何，因此文档路径仅与仓库组织有关。该页面不是技能，也不是 `SKILL.md` 的副本。只有这两个桶被推广；其余的（`misc/`、`personal/`、`in-progress/`、`deprecated/`）没有文档页面。

这些技能大多是 **用户触发** 的：代理绝不会为你触发它们，因此 *你* 是必须记住它们存在以及何时使用它们的索引。这种记忆是 **认知负荷**。文档页面的工作是减轻它 —— 将一名读者围绕一个技能进行定位，以便他们能在脑海中掌握它，知道何时使用它，并了解它在系统中的位置。这些页面共同构成了一个分布式路由器；每个都是一个节点。

每当推广的技能被添加、重命名或行为发生更改时，请立即采取行动：创建或重新同步其文档页面。重命名也会移动文件（`docs/<bucket>/<old>.md` → `docs/<bucket>/<new>.md`），因为发布的 URL 会跟踪名称；在 `engineering/` 和 `productivity/` 之间移动的技能会将文档文件移动到匹配的文件夹。`misc/`、`personal/`、`in-progress/` 和 `deprecated/` 中的技能没有页面 —— 这些桶中的任何一个都没有被推广。从其中之一移动到 `engineering/` 或 `productivity/` 的技能会获得一个页面；反向移动的技能会失去它。

由于这些页面发布在 `aihero.dev` 上，**每个链接都是绝对路径** —— 永远不是仓库相对路径。指向另一个技能的链接指向 `https://aihero.dev/skills-<name>`；指向仓库内部的链接指向其完整的 `https://github.com/mattpocock/skills/...` URL。在仓库中有效的相对链接一旦发布就会失效。

没有 H1 —— 发布的页面会使用 slug 作为标题。

## 页面结构

填写下方的模板。**固定框架**（Quickstart 块、源码链接、`## What it does`、`## When to reach for it`、`## Where it fits`）出现在每个页面上。**可调整的中间部分** —— `## Prerequisites` 和自由形式的实质性章节 —— 仅包含该特定技能的内容；删除其余部分。

<page-template>

Quickstart:

```bash
npx skills add mattpocock/skills --skill=<name>
```

```bash
npx skills update <name>
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/<bucket>/<name>)

## 功能说明

一两段通俗易懂的文字。以技能的一句话工作职责开头，然后陈述 **定义约束** —— 使该技能与明显默认行为不同的唯一事实（对于 `to-spec`：它不会再次采访用户，它综合已有的已知信息）。将其写成普通的陈述句 —— 绝不要写成带有标签的旁注，如 "The defining constraint:" 或 "The key thing:"；这种公式读起来像填充内容。这一行是页面上最有价值的内容；绝不要省略它。

## 何时使用

你如何以及何时使用该技能 —— 两个关键点，实际上两者总是同时存在：

* **调用模式**。说明你是输入它还是代理触发它。用户触发的技能："你通过输入 `/<name>` 来调用它 —— 代理不会主动使用它。" 模型触发的技能："输入 `/<name>`，或者当任务匹配时代理会自动使用它。"
* **触发边界**。索引条目："当……时使用这个"。当技能与兄弟技能容易混淆时，添加另一半 —— "对于 <X>，请使用 [<兄弟技能>](https://aihero.dev/skills-<sibling>)。"

## 前置条件

可选 —— 仅在技能需要某些东西处于就绪状态才能运行时包含；否则完全省略标题。涵盖：**它写入的工作区**（有状态技能如 `grill-with-docs` 会写入 `CONTEXT.md` 和 ADR；`teach` 会构建整个目录 —— 说明它写入什么以及在哪里），**先前设置**（`triage`/`to-spec`/`to-tickets` 需要已配置问题跟踪器的 `setup-matt-pocock-skills`），或 **仓库特定工具**。在任何地方运行的无状态技能没有前置条件 —— 删除该部分。

## <free-form middle>

一到三个简短的章节，使用技能的 *自有词汇*，使其易于理解 —— 选择适合技能的任何标题：它运行的循环、它生成的工件、它做出的分支、它消除的一种反模式。没有固定的标题；技能种类太多，无法统一。

唯一不可协商的规则：**突出技能的核心概念 / 定义性想法** —— `tight` 反馈循环、`deep module`、throwaway-code-answers-a-question、红绿。这有两重好处：读者学习了技能 *是* 什么，并学习了他们稍后用来 *使用* 它的词汇。

## 判断是否生效

可选。一份简短的、可检查的列表，列出表明技能实际上正在工作的可观察信号 —— 当它触发时他们应该看到什么，以及当它没有触发时通过缺失情况显示什么。当技能有清晰的指示时包含它（例如 `to-spec` 不会重新采访你就写入；核心概念在追踪中重新出现）；当信号模糊时省略标题。仅几条要点。

## 在系统中的位置

始终存在。用一两句话将技能定位在系统中：

* **角色**。给它命名：**链式步骤**（`grill-with-docs → to-spec → to-tickets → implement → code-review`）、**一次性设置**（`setup-matt-pocock-skills`）、**定期维护**（`improve-codebase-architecture`、"每隔几天"），或 **随时可用的独立技能**（`diagnosing-bugs`、`prototype`、`handoff`）。独立技能的地图是一句诚实的陈述 —— 远比省略该部分要好。
* **邻居**。一个或两个重要的兄弟技能，每个都带有原因从句，绝对链接。
* **地图**。指向 [ask-matt](https://aihero.dev/skills-ask-matt)，即整个集合的路由器，以便此页面保持为一个节点，无需重绘图形。

</page-template>

## 约定

* 解释 **为什么**，而不是过程。页面用于定位和安置技能；它从不复制 `SKILL.md` 的步骤或模板转储 —— 选择工具的人类不需要运行手册。
* 使用技能的 **核心词汇**（*seam*、*deep module*、*tracer bullet*），使页面和技能使用同一种语言。
* 保持页面本身低负载。它是关于低认知负荷技能的文档；家具（多余的标题、重述的链接）正是它所反对的内容。

## 完成标准

* 页面存在于 `docs/<bucket>/<name>.md`，并且没有过时的页面在重命名或桶移动后幸存。
* 快速入门块和源码链接指明了正确的桶和技能；更新行也指明了技能名称。
* `## What it does` 将定义约束陈述为通俗的散文，而不是带标签的旁注。
* `## When to reach for it` 陈述调用模式和触发边界。
* `## Where it fits` 命名角色并链接到 `ask-matt`。
* 在存在前置条件（工作区、先前设置、工具）的地方说明，而在不存在的地方省略该部分。
* 中间部分突出了核心概念。
* 每个链接都是绝对路径，且都能正常解析。
