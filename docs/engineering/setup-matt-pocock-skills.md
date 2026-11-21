## 它做什么

`setup-matt-pocock-skills` 回答了关于单个仓库的三个问题：问题存放在哪里、Triage 标签叫什么、以及领域文档放在哪里。它将这些答案记录为 `docs/agents/` 下的 markdown 文件。

这些文件是不同仓库之间唯一会变化的部分。技能本身在任何地方都是完全相同的。它们在运行时读取 `docs/agents/issue-tracker.md` 并按其说明执行。这就是为什么这套技能不绑定 GitHub，也为什么你永远不需要编辑技能文件来指向另一个跟踪器。使用“将技能链接到自定义问题跟踪器”调用它，适用于任何你能以编程方式连接的系统，且无需更改技能。

它是一个提示驱动的技能，而非确定性脚本。它会读取你的 `git remote`、`CLAUDE.md` 和 `GLOSSARY.md`，提出它发现的内容，并在写入任何东西前等待你的确认。

## 何时使用它

你通过输入 `/setup-matt-pocock-skills` 来调用它；[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自动调用它。其元数据刻意标记为不可调用，因此没有其他技能能替你触发它。

每个仓库只需运行一次，且应在首次使用任何其他工程技能之前运行。如果 [triage](https://aihero.dev/skills-triage)、[to-spec](https://aihero.dev/skills-to-spec)、[to-tickets](https://aihero.dev/skills-to-tickets) 或 [wayfinder](https://aihero.dev/skills-wayfinder) 开始猜测你的 issue 该去哪里，或应用了你跟踪器中不存在的标签，说明该仓库尚未设置。你也可以在项目进行中途运行它。该技能会读取现有内容，因此之前的工作不会丢失。

## 先决条件

它会写入你运行它的仓库：

| 它写入                   | 写入位置                                |
| --------------------- | ----------------------------------- |
| `issue-tracker.md`    | `docs/agents/`                      |
| `domain.md`           | `docs/agents/`                      |
| `triage-labels.md`    | `docs/agents/`，仅当 `triage`技能已安装时    |
| 一个 `## Agent skills`块 | 两者中 `CLAUDE.md`或 `AGENTS.md`已存在的那一个 |

所有内容都以 markdown 形式提交。不存在用户级或全局模式。配置驻留在仓库中，因此每个仓库都有自己的副本。

## 三个决策

它在每个章节开头给出推荐答案，并跳过其探索已解答的任何问题。大多数运行仅需两次确认。

| 决策            | 它建议什么                                                                                   | 何时询问                                            |
| ------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **问题跟踪器**     | 匹配你的 `git remote`                                                                       | 总是询问，因为这是唯一真正的选择                                |
| **Triage 标签** | 保留五个规范名称（`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`) | 仅当 `triage`技能已安装时                               |
| **领域文档**      | single-context: one `GLOSSARY.md`加上 `docs/adr/`位于根目录                                    | 仅当它发现 monorepo 信号时，才会提供一个多上下文 `GLOSSARY-MAP.md` |

跟踪器选项：

| 选项              | 问题存放位置                         | 需要               |
| --------------- | ------------------------------ | ---------------- |
| **GitHub**      | 该仓库的 GitHub Issues             | 需要 `gh`CLI       |
| **GitLab**      | 该仓库的 GitLab Issues             | 需要 `glab`CLI     |
| **本地 markdown** | 本仓库中 `.scratch/<feature>/`下的文件 | 无需任何东西，甚至不需要远程仓库 |
| **其他**          | 你指定的任何地方                       | 你提供一段描述工作流程的文字   |

前三个作为模板内置在技能中，开箱即用。本地 markdown 是一个完整的选项，而非后备方案。该技能支持没有远程仓库的单人项目。有一个注意事项：如果你使用 GitHub，请勿使用本地 markdown。它们是备选方案，二选一即可。

“其他”也是一个完整选项。Jira、Linear、Azure DevOps 和 Beads 都是通过这种方式工作的。你描述工作流程，技能将你的描述记录在 `docs/agents/issue-tracker.md` 中，下游技能遵循该描述执行。用户已经构建了这样的集成：基于 [MCP](https://www.aihero.dev/ai-coding-dictionary/mcp) 的 Jira 变体、形似 `gh` 的 Gitea CLI、手工搭建的本地仪表盘。

## 常见问题

**我必须使用 GitHub 吗？**

不必。GitHub、GitLab 以及 `.scratch/` 下的本地 markdown 都作为现成模板内置，其他任何系统都可通过“其他”路径接入。这是被问得最多的问题，大致措辞如下：“*硬绑定 GitHub*”、“*能用 GitLab / Jira 吗*”、“*Azure DevOps 怎样*”。答案永远一样：是 setup 选择跟踪器，而非技能。

**更新技能后，我需要重新运行它吗？**

v1.1 后的直接回答是肯定的。技能自身的结束语则较柔和：它建议仅在切换跟踪器或重新开始时重跑。两种说法都站得住脚。种子模板会随版本变化，因此旧版本生成的 `docs/agents/issue-tracker.md` 可能与当前读取它的技能不匹配。如果下游技能的行为与文档描述不符，请重跑 setup。成本很低。

**它写入了 `CLAUDE.md`，但我用的是 Codex。**

这是一个已知缺口，目前仍未解决。文件选择规则是“若 `CLAUDE.md` 存在则编辑它，否则编辑 `AGENTS.md`”。它检查的是文件是否存在，而非正在运行哪个 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)。在遗留有 `CLAUDE.md` 的仓库中，技能会将 `## Agent skills` 块写入 Codex 从不读取的文件。用户有两种变通方法：手动将该块移至 `AGENTS.md`，或以 `AGENTS.md` 为准并让 `CLAUDE.md` 仅作单行指向。若两个文件都不存在，技能会询问你创建哪一个，而不是自行决定。这让期望它自动决定的用户感到困惑。

**它没有创建我的 triage 标签。**

它不会创建。`docs/agents/triage-labels.md` 是一个*映射*：它告诉 `/triage` 你的跟踪器中哪些字符串对应那五个规范角色。它不运行 `gh label create`。在全新的 GitHub 仓库中标签尚不存在，用户已多次将其作为 bug 报告。两个后果：

* 如果你的跟踪器已经使用这些规范名称，那么映射就是一张恒等表，无需进行任何配置。这是预期的常见情况，而不是缺失的步骤。
* 此技能也不创建 [wayfinder](https://aihero.dev/skills-wayfinder) 的 `wayfinder:map` 和 `wayfinder:<type>` 标签，且在 GitHub 仓库上 `gh issue create --label <缺失标签>` 会失败而非创建标签。请在首次运行 wayfinder 前手动创建它们。

**我可以在这里配置其他技能的行为吗[追问](https://www.aihero.dev/ai-coding-dictionary/grilling)频率、问题格式、语气**

不能。它只配置三件事：跟踪器、标签、文档布局。用户曾请求将其作为用户偏好设置的集中地。回答是：技能保持固有观点，不接受用户级配置。偏好应以纯文本指令形式放在你的 `CLAUDE.md` 中，所有技能读取该文件时都会生效。

**我可以把配置放在 `~/.claude`而不是提交到每个仓库吗？**

目前不行。有跨多仓库运行技能的用户提交过此需求，但尚无用户级模式。每个仓库都自带自己的 `docs/agents/`。

**有一个技能来配置其他技能，这不是很奇怪吗？**

一项长期抱怨认为是的，原话是：“*有一个技能来设置其他技能让我觉得不对劲：这意味着 LLM 在配置它自己的技能。*” 权衡确实存在。若无设置步骤，每个涉及 issue 的技能都需要自带一份跟踪器说明副本。输出是可读可编辑的 markdown，这限制了风险。你可以阅读它写的每个文件并手动修改。日常变更请走这条路，而非再跑一遍 setup。

## 如果它起作用了

* `docs/agents/issue-tracker.md` 和 `docs/agents/domain.md` 存在，如果安装了 `triage`，还需有 `triage-labels.md`。
* 你使用的 harness 所读取的指令文件中会出现 `## Agent skills` 章节，其中包含指向上述各文件的单行摘要。
* 它提出的跟踪器与你使用的远程仓库一致，且标签字符串与你跟踪器中现有的标签匹配。
* 之后，`/to-tickets` 发布时不会再询问你 issue 存放在哪里，`/triage` 也会应用已有标签，而不是凭空创造它们。
* 技能文件本身没有任何改动。如果 setup 修改了 `SKILL.md`，那就出问题了。

## 它在系统中的位置

`setup-matt-pocock-skills` 是工程流程的**一次性设置**，是其他一切所假设的前置条件，而非链路中的一个步骤。它的邻居是它的读取者：[triage](https://aihero.dev/skills-triage) 应用此处写入的标签词汇；[to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets) 向此处指定的跟踪器发布内容；[wayfinder](https://aihero.dev/skills-wayfinder) 读取同一跟踪器文件的“Wayfinding operations”章节，以了解如何存储地图和子[ticket](https://www.aihero.dev/ai-coding-dictionary/ticket)。[domain-modeling](https://aihero.dev/skills-domain-modeling) 稍后会填充 setup 记录的领域文档布局。它仅在你确认某个术语或决策时创建 `GLOSSARY.md` 和 ADR，因此 setup 后没有领域文档是正常的。关于下一步该用哪个技能，[ask-matt](https://aihero.dev/skills-ask-matt) 会为整套技能导航。
