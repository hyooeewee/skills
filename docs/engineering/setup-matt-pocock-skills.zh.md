Quickstart:

```bash
npx skills add mattpocock/skills --skill=setup-matt-pocock-skills
```

```bash
npx skills update setup-matt-pocock-skills
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/setup-matt-pocock-skills)

## 功能说明

`setup-matt-pocock-skills` 教导一个仓库如何在工程技能中表现 —— 也就是问题存放在哪里，分类标签叫什么，领域文档位于何处 —— 并将这些答案记录为其他技能读取的 **配置**。

它写入配置，而不是硬编码行为。工程链假设 `docs/agents/` 下存在三个文件；这个技能是一次性的引导程序，用于生成这些文件。它会从你实际的仓库（`git remote`、现有标签、现有的 `CONTEXT.md`）中发现这些信息，并与你确认，而不是凭空猜测。它是基于提示驱动的 —— 探索、展示发现的内容、确认、然后写入 —— 而不是确定性的脚手架。

## 何时使用

你通过输入 `/setup-matt-pocock-skills` 来调用它 —— 代理不会自行调用它。

在使用任何其他工程技能之前，**每个仓库只调用一次**。如果 [triage](https://aihero.dev/skills-triage)、[to-spec](https://aihero.dev/skills-to-spec) 或 [to-tickets](https://aihero.dev/skills-to-tickets) 开始猜测你的问题位置或应用不存在的标签，说明它们还没有在这里设置好。只有在切换问题跟踪器或重新开始时才重新运行它 —— 日常调整只需编辑 `docs/agents/*.md`。

## 三个决策

它会为每个问题提出一个你可以用一个词接受的推荐答案，并跳过所有它已经可以推断出的内容 —— 所以大多数运行只是一两次快速确认：

* **问题跟踪器** —— 工作被追踪的地方，因此 `triage`/`to-spec`/`to-tickets` 知道是调用 `gh`、`glab`、在 `.scratch/` 下写入 markdown，还是遵循你描述的工作流程。GitHub、GitLab、本地 markdown 或其他。（它提议与你 `git remote` 匹配的那个。）
* **分类标签** —— 只有在安装了 `triage` 技能时才会询问，然后只是问：保留默认标签（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`）吗？只有当你的跟踪器已经使用其他名称时才说不，这样 `triage` 就会应用真实的标签，而不是创建重复项。
* **领域文档** —— 假设为单上下文（根目录下一个 `CONTEXT.md` + `docs/adr/`），这几乎适合每个仓库；只有当它发现单体仓库信号时，才会生成多上下文映射。

输出是 `docs/agents/` 下的一组文件 —— 安装 `triage` 时为 `issue-tracker.md`、`domain.md` 和 `triage-labels.md` —— 以及一个 `## Agent skills` 块，指向你在仓库已经使用的 `CLAUDE.md` / `AGENTS.md` 中的任何一个。这些文件是工具包其余部分所依赖的共享基础。

## 判断是否生效

* `issue-tracker.md` 和 `domain.md` 落在 `docs/agents/` 下（安装 `triage` 时加上 `triage-labels.md`），并且你的 `CLAUDE.md` 或 `AGENTS.md` 中出现了一个 `## Agent skills` 部分。
* 它建议的跟踪器与你的真实 `git remote` 匹配，并且标签与你仓库中已存在的字符串匹配。
* 之后，`triage` 和 `to-tickets` 在正确的位置使用正确的标签进行操作，而不是询问或猜测。

## 在系统中的位置

`setup-matt-pocock-skills` 是一个**运行一次的设置** —— 整个工程集合所依赖的基础，而不是你重复的步骤。它的邻接技能是读取它写入内容的技术：[triage](https://aihero.dev/skills-triage)，因为它应用在这里配置的标签词汇；以及 [to-spec](https://aihero.dev/skills-to-spec) / [to-tickets](https://aihero.dev/skills-to-tickets)，因为它们发布到在这里配置的问题跟踪器。先运行它；下游的一切都假定它存在。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指引。
