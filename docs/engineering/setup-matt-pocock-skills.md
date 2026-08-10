## 功能说明

`setup-matt-pocock-skills` 询问一个仓库的三个问题——问题存放在哪里、优先级标签叫什么名字、以及领域文档位于何处——并将答案记录为 `docs/agents/` 下的 markdown 文件。

这些文件是唯一在仓库间变化的东西。技能本身到处都是一样的；它们在运行时读取 `docs/agents/issue-tracker.md` 并照其所述执行。这就是为什么该集合不绑定于 GitHub，为什么任何技能文件都不需要编辑来指向别处。通过“将技能链接到自定义问题追踪器”来调用它，可以适用于任何你可以编程连接的东西，且无需对技能进行任何更改。

它是一个基于提示的技能，而不是确定性脚本。它读取你的 `git remote`、你现有的 `CLAUDE.md`、你现有的 `CONTEXT.md`，提出它发现的内容，并在写入任何内容之前等待你确认。

## 何时使用

你通过输入 `/setup-matt-pocock-skills` 来调用它——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会自动去调用它。它被故意标记为不可调用，因此其他技能无法为你触发它。

每个仓库调用一次，在任何其他工程技能使用之前。如果 [triage](https://aihero.dev/skills-triage)、[to-spec](https://aihero.dev/skills-to-spec)、[to-tickets](https://aihero.dev/skills-to-tickets) 或 [wayfinder](https://aihero.dev/skills-wayfinder) 开始猜测你的问题去哪里，或者应用你 tracker 中没有的标签，说明它们还没有在这里设置好。一个已经进行到项目一半的仓库是运行它的好地方；该技能会读取已有的内容，且不会浪费之前的工作。

## 前置条件

它写入你运行它的仓库中：

| 它写入                                         | 位置                                                                  |
| ------------------------------------------- | ------------------------------------------------------------------- |
| `issue-tracker.md`                          | `docs/agents/`                                                      |
| `domain.md`                                 | `docs/agents/`                                                      |
| `triage-labels.md`                          | `docs/agents/`仅当安装了 \`triage\` 技能时 `triage`技能已安装                    |
| 一个 \`## Agent skills\` 块 `## Agent skills`块 | 任意一个 \`CLAUDE.md\` / \`AGENTS.md\` 已存在 `CLAUDE.md` / `AGENTS.md`已存在 |

全部都是提交的 Markdown。没有用户级或全局模式：配置存在于仓库中，因此每个仓库都有自己的副本。

## 三个决策

它在每个部分以推荐答案开头，并跳过已经确定的任何探索。大多数运行只需要两次确认即可完成。

| 决策        | 它建议的内容                                                                                                                                                                           | 它实际提问的时候                                 |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| **问题追踪器** | 与你的 \`git remote\` 匹配的那个 `git remote`                                                                                                                                            | 总是 — 这是唯一真实的选择                           |
| **优先级标签** | 保留五个标准名称 (\`needs-triage\`, \`needs-info\`, \`ready-for-agent\`, \`ready-for-human\`, \`wontfix\`)`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`) | 仅当安装了 \`triage\` 技能时 `triage`技能已安装       |
| **领域文档**  | single-context: one `CONTEXT.md`加上 `docs/adr/`在根目录                                                                                                                               | 多上下文 \`CONTEXT-MAP.md\` `CONTEXT-MAP.md` |

追踪器选项：

| 选项              | 问题存放位置                                                   | 需求                     |
| --------------- | -------------------------------------------------------- | ---------------------- |
| **GitHub**      | 仓库的 GitHub Issues                                        | \`glab\` CLI `gh`CLI   |
| **GitLab**      | 仓库的 GitLab Issues                                        | \`glab\` CLI `glab`CLI |
| **本地 markdown** | \`.scratch/\<feature>/\` 下的文件 `.scratch/<feature>/`在此仓库中 | 无需任何东西 —— 完全没有远程仓库     |
| **其他**          | 随你指定                                                     | 你描述工作流程的一段话            |

前三个作为技能中的模板附带并提供开箱即用的功能。本地 markdown 是一等选项，而不是备选方案：没有远程仓库的独立项目得到完全支持。一个值得重复的注意事项：如果你使用 GitHub，就不要使用本地 markdown。它们是替代方案，而不是分层。

"Other" 也不是一个占位符。这是 Jira、Linear、Azure DevOps 和 Beads 都能工作的原因：你描述工作流程，技能将你的文字记录在 `docs/agents/issue-tracker.md` 中，下游技能遵循这些文字。社区已经这样做了——一个基于 [MCP](https://www.aihero.dev/ai-coding-dictionary/mcp) 的 Jira 变体，一个形状像 `gh` 的 Gitea CLI，一个手工构建的本地仪表板。

## 常见问题

**我必须使用 GitHub 吗？**

不。GitHub、GitLab 和 `.scratch/` 下的本地 markdown 都作为现成的模板附带，其他任何内容都通过“其他”路径工作。这是记录中重复最多的问题，大致用这些话表达：“*硬锁定到 GitHub*”、“*我可以使用 GitLab / Jira 吗*”、“*那 Azure DevOps 呢*”。每次的答案都是：tracker 是一个设置答案，而不是技能属性。

**更新技能后，我需要重新运行它吗？**

它写入到了 `CLAUDE.md`，但我使用的是 Codex。

**它写入了 \`CLAUDE.md\`，但我使用的是 Codex。 `CLAUDE.md`但我使用的是 Codex。**

已知缺口，仍在处理中。文件选择规则是“如果存在则编辑 `CLAUDE.md`，否则编辑 `AGENTS.md`”——它检查哪个文件存在，而不是检查哪个 [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 正在运行。带有来自 Claude Code 的 `CLAUDE.md` 的仓库会在 Codex 从不读取的某个地方获得其 `## Agent skills` 块。两种变通方法正在流通：手动将块移动到 `AGENTS.md`，或者保持 `AGENTS.md` 的权威性，并使 `CLAUDE.md` 成为其的一行指针。如果两个文件都不存在，技能会询问你创建哪个而不是选择，这让期望它只是决定的人感到困惑。

**它没有创建我的优先级标签。**

它不会。`docs/agents/triage-labels.md` 是一个 *映射*——它告诉 `/triage` 你的 tracker 中的哪些字符串对应五个标准角色。它不会运行 `gh label create`。在一个全新的 GitHub 仓库中，标签确实还不存在，这已经被多次作为错误提交。两个后续事项：

* 如果你的 tracker 已经使用了标准名称，映射就是一个恒等表，不需要配置。这是预期的常见情况，而不是缺失的步骤。
* [wayfinder](https://aihero.dev/skills-wayfinder) 的 `wayfinder:map` 和 `wayfinder:<type>` 标签也不会在这里创建，并且 `gh issue create --label <missing>` 会直接失败而不是创建标签。在 GitHub 仓库上的第一次 wayfinder 运行之前，请手动创建它们。

**我能否在这里配置其他技能的行为 —— \[grill-me]\(...) 频率、问题格式、语调？ [grill-me](https://www.aihero.dev/ai-coding-dictionary/grilling)频率、问题格式、语调？**

我能否在这里配置其他技能的行为 —— \[grilling] 的频率、问题格式、语调？

**我可以将配置保存在 \~/.claude 中，而不是提交到每个仓库吗？ `~/.claude`而不是提交到每个仓库吗？**

我可以将配置保存在 `~/.claude` 中，而不是提交到每个仓库吗？

**有一个配置其他技能的技能，这难道不奇怪吗？**

一个长期存在的抱怨说是的，措辞如下：“*对我来说，有一个设置其他技能的技能感觉不对——这意味着 LLM 正在配置它自己的技能。*”这种权衡是真实且被承认的：设置步骤的替代方案是将 tracker 指令复制到每个接触问题的技能中。输出是可检查、可编辑的 markdown，这是缓解措施——你可以阅读它写的每个文件并手动更改它，日常调整正是如此，而不是另一次运行。

## 判断是否生效

* `docs/agents/issue-tracker.md` 和 `docs/agents/domain.md` 存在，此外如果已安装 `triage`，还会有 `triage-labels.md`。
* 在你实际使用的框架指令文件中会出现一个 `## Agent skills` 部分，其中包含一行摘要，指向这些文件中的每一个。
* 它建议的 tracker 与你实际使用的远程仓库匹配，且标签字符串与你 tracker 中实际存在的标签相匹配。
* 之后，`/to-tickets` 会发布而不询问问题存放在哪里，而 `/triage` 会应用标签而不是生成标签。
* skill 文件本身没有任何更改。如果 setup 编辑了 `SKILL.md`，说明出错了。

## 在系统中的位置

`setup-matt-pocock-skills` 是工程流程的 **run-once setup**，它是其他操作的前提条件，而非流程链中的一个步骤。它的邻居即是它的读取者：[triage](https://aihero.dev/skills-triage)，它应用此处定义的标签词汇；[to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets)，它们发布到此处命名的 tracker 中；以及 [wayfinder](https://aihero.dev/skills-wayfinder)，它读取同一个 tracker 文件的“Wayfinding operations”部分，以了解地图和子 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 是如何存储的。它记录的 domain-doc 布局是 [domain-modeling](https://aihero.dev/skills-domain-modeling) 稍后填充的内容——它会在术语或决策实际解决时才惰性地创建 `CONTEXT.md` 和 ADRs，因此 setup 后仓库为空是预期状态。至于下一个应该使用哪个 skill，[ask-matt](https://aihero.dev/skills-ask-matt) 会负责路由整个集合。
