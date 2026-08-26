## 它做什么

`setup-matt-pocock-skills` 回答了关于单个仓库的三个问题：问题存放在哪里、Triage 标签叫什么、以及领域文档放在哪里。它将这些答案记录为 `docs/agents/` 下的 markdown 文件。

这些文件是唯一在不同仓库之间有所差异的东西。技能本身在任何地方都是相同的；它们在运行时读取 `docs/agents/issue-tracker.md` 并按照其中的内容行事。这就是为什么这套技能不绑定于 GitHub，也是为什么没有任何技能文件需要修改来指向其他地方。用“将技能链接到自定义问题跟踪器”的方式来调用它，可以适用于任何你能以编程方式连接的东西，且技能本身零更改。

这是一个提示驱动的技能，而不是一个确定性的脚本。它会读取你的 `git remote`、现有的 `CLAUDE.md`、现有的 `CONTEXT.md`，提出它发现的内容，并在写入任何内容之前等待你确认。

## 何时使用它

你通过输入 `/setup-matt-pocock-skills` 来调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自行调用它。它被特意标记为不可调用，因此没有其他技能可以为你触发它。

每个仓库使用一次，在任何其他工程技能首次使用之前。如果 [triage](https://aihero.dev/skills-triage)、[to-spec](https://aihero.dev/skills-to-spec)、[to-tickets](https://aihero.dev/skills-to-tickets) 或 [wayfinder](https://aihero.dev/skills-wayfinder) 开始猜测你的问题该放到哪里，或者应用你的跟踪器中不存在的标签，那说明这里还没有完成设置。一个已经在项目中期阶段的仓库也是一个运行它的好地方；技能会读取已有的内容，之前的工作不会白费。

## 先决条件

它会写入你运行它的仓库：

| 它写入                   | 写入位置                                 |
| --------------------- | ------------------------------------ |
| `issue-tracker.md`    | `docs/agents/`                       |
| `domain.md`           | `docs/agents/`                       |
| `triage-labels.md`    | `docs/agents/`，仅当 `triage`技能已安装时     |
| 一个 `## Agent skills`块 | 两者中 `CLAUDE.md` / `AGENTS.md`已存在的那一个 |

所有这些都是提交的 markdown。没有用户级或全局模式：配置存在于仓库中，所以每个仓库都有自己的副本。

## 三个决策

它会在每个部分开头给出推荐答案，并跳过已经完成的探索。大多数情况下，只需两次确认即可完成。

| 决策            | 它建议什么                                                                                   | 它何时真正询问                                        |
| ------------- | --------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **问题跟踪器**     | 匹配你的 `git remote`                                                                       | always: this is the one real choice            |
| **Triage 标签** | 保留五个规范名称（`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`) | 仅当 `triage`技能已安装时                              |
| **领域文档**      | single-context: one `CONTEXT.md`加上 `docs/adr/`位于根目录                                     | 仅当它发现 monorepo 信号时，才会提供一个多上下文 `CONTEXT-MAP.md` |

跟踪器选项：

| 选项              | 问题存放位置                         | 需要                        |
| --------------- | ------------------------------ | ------------------------- |
| **GitHub**      | 该仓库的 GitHub Issues             | 需要 `gh`CLI                |
| **GitLab**      | 该仓库的 GitLab Issues             | 需要 `glab`CLI              |
| **本地 markdown** | 本仓库中 `.scratch/<feature>/`下的文件 | nothing: no remote at all |
| **其他**          | 你指定的任何地方                       | 你提供一段描述工作流程的文字            |

前三种作为模板内置在技能中，开箱即用。本地 markdown 是一级选项，而不是退而求其次的方案：没有远程仓库的个人项目完全受支持。有一点值得重申：如果你正在使用 GitHub，就不要使用本地 markdown。两者是替代关系，不是叠加关系。

“Other” 也不是一个占位符。Jira、Linear、Azure DevOps 和 Beads 都能工作的原因就是：你描述工作流程，技能将你的文字记录在 `docs/agents/issue-tracker.md` 中，下游技能会遵循这些文字。社区已经实现了这一点：一个基于 [MCP](https://www.aihero.dev/ai-coding-dictionary/mcp) 的 Jira 变体，一个像 `gh` 一样的 Gitea CLI，以及一个手工构建的本地仪表板。

## 常见问题

**我必须使用 GitHub 吗？**

不。GitHub、GitLab 和 `.scratch/` 下的本地 markdown 都作为现成模板提供，其他一切都可以通过“其他”路径工作。这是记录中被问得最多的问题，大致措辞如下：*“hard locked to github”*、*“can I use GitLab / Jira”*、*“what about Azure DevOps”*。每一次的回答都是：跟踪器是设置答案，而不是技能属性。

**更新技能后，我需要重新运行它吗？**

v1.1 之后，Matt 被直接问到这个问题时说是的。技能自身的结束语更温和：它告诉你重新运行只是为了切换跟踪器或重新开始。两者都有道理，这种差距的原因是真实的：种子模板在不同版本之间会发生变化，因此旧版本编写的 `docs/agents/issue-tracker.md` 可能会与现在读取它的技能变得过时。如果下游技能开始做一些文档描述不同的事情，重新运行是廉价的修复方法。

**它写入了 `CLAUDE.md`，但我用的是 Codex。**

已知存在这个缺口，尚未关闭。文件选择规则是“如果存在则编辑 `CLAUDE.md`，否则编辑 `AGENTS.md`”：它检查的是哪个文件存在，而不是哪个 [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 正在运行。一个带有 Claude Code 留下的 `CLAUDE.md` 的仓库会在 Codex 永远不会读取的地方获得它的 `## Agent skills` 块。目前有两种变通方案在流传：手动将块移到 `AGENTS.md`，或者保持 `AGENTS.md` 的权威地位，并将 `CLAUDE.md` 设为指向它的单行指针。如果两个文件都不存在，技能会问你创建哪一个而不是替你选择，这让一些期望它直接决定的人感到困惑。

**它没有创建我的 triage 标签。**

它不会。`docs/agents/triage-labels.md` 是一个 *映射*：它告诉 `/triage` 你跟踪器中的哪些字符串对应五个规范角色。它不会运行 `gh label create`。在一个全新的 GitHub 仓库上，标签确实还不存在，这已经被多次提交为 bug。两个后续步骤：

* 如果你的跟踪器已经使用这些规范名称，那么映射就是一张恒等表，无需进行任何配置。这是预期的常见情况，而不是缺失的步骤。
* [wayfinder](https://aihero.dev/skills-wayfinder) 的 `wayfinder:map` 和 `wayfinder:<type>` 标签也不会在这里创建，而 `gh issue create --label <missing>` 会直接失败，而不是创建标签。在 GitHub 仓库上首次运行 wayfinder 之前，请手动创建它们。

**我可以在这里配置其他技能的行为吗[追问](https://www.aihero.dev/ai-coding-dictionary/grilling)频率、问题格式、语气**

不。它只配置三件事：跟踪器、标签、文档布局。有人直接要求让它成为存放每个用户偏好的地方，但一贯的答案是：技能保持有主见：*“配置即死亡。”* 偏好应该以普通指令的形式放在你的 `CLAUDE.md` 中，每个技能本来就会读取它。

**我可以把配置放在 `~/.claude`而不是提交到每个仓库吗？**

目前还不行。有一个尚未关闭的请求，正是来自一位跨多个仓库使用这些技能的人，但目前还不存在用户级模式。每个仓库都自带自己的 `docs/agents/`。

**有一个技能来配置其他技能，这不是很奇怪吗？**

一个长期存在的抱怨说“是”，措辞如下：“拥有一个设置其他技能的技能让我感觉不对劲：这意味着 LLM 正在配置它自己的技能。”这种权衡是真实且被承认的：设置步骤的替代方案是将跟踪器指令复制到每个涉及问题的技能中。输出是可检查、可编辑的 markdown，这就是缓解措施：你可以阅读它写的每一个文件并手动更改，日常的微调正是如此，而不是再次运行。

## 如果它起作用了

* `docs/agents/issue-tracker.md` 和 `docs/agents/domain.md` 存在，如果安装了 `triage`，还需有 `triage-labels.md`。
* 在你的 harness 实际读取的指令文件中出现一个 `## Agent skills` 部分，其中包含一行摘要，分别指向这些文件中的每一个。
* 它建议的 tracker 与你实际使用的 remote 匹配，且标签字符串与你的 tracker 中真实存在的标签匹配。
* 之后，`/to-tickets` 发布时不会再询问你 issue 存放在哪里，`/triage` 也会应用已有标签，而不是凭空创造它们。
* 技能文件本身没有任何改动。如果 setup 修改了 `SKILL.md`，那就出问题了。

## 它在系统中的位置

`setup-matt-pocock-skills` 是工程流的**一次性设置**，是其他所有内容所假设的前提条件，而不是链中的一个步骤。它的邻接者是它的读取者：[triage](https://aihero.dev/skills-triage)，它应用这里写下的标签词汇；[to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets)，它们发布到此处命名的跟踪器；以及 [wayfinder](https://aihero.dev/skills-wayfinder)，它读取同一跟踪器文件的“Wayfinding operations”部分，以了解地图和子 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 是如何存储的。它记录的领域文档布局是后来 [domain-modeling](https://aihero.dev/skills-domain-modeling) 填补的内容：它会在术语或决策真正解决时才延迟创建 `CONTEXT.md` 和 ADR，因此设置后的空仓库是预期状态。至于下一步应该使用哪个技能，[ask-matt](https://aihero.dev/skills-ask-matt) 会路由整个集合。
