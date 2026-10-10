## 它做什么

`triage` 会处理项目跟踪器上的问题。它通过一个小型状态机将每个问题移动到 **分类角色**（类别角色和状态角色）。每个问题最终会变成一个 agent-ready 简报、一个给报告者的具体问题，或一个带有记录原因的已关闭问题。

它**仅用于你没有创建的问题**。这意味着原始 bug 报告、传入的功能请求，或未通知而到达的外部 pull request：从外部进入跟踪器的工作，无论报告者以何种形式留下。来自 [to-tickets](https://aihero.dev/skills-to-tickets) 的 [Tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 已经是 agent-ready 的，所以如果你对它们运行 `triage`，就会浪费工作。规则很简单：`/triage` 仅用于传入的问题，不用于你自己创建的问题。

它还不同于手动打标签，因为它会推荐然后等待。它会给出类别和状态的判断及推理，以及在代码库中发现的内容，直到你告诉它应用之前，它不会应用任何东西。

## 何时使用它

你通过输入 `/triage` 然后用通俗语言描述你想要什么来调用它。[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会主动去调用它。示例："Show me anything that needs my attention"、"let's look at #42"、"move #42 to ready-for-agent"。

| 你拥有什么                                                                 | 去哪里                                                          |
| --------------------------------------------------------------------- | ------------------------------------------------------------ |
| 一个满是他人原始报告的跟踪器                                                        | `/triage`                                                    |
| 自己有一个粗略的想法，什么都没写下来                                                    | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 将一次已定型的对话转化为 [规格说明](https://www.aihero.dev/ai-coding-dictionary/spec) | [to-spec](https://aihero.dev/skills-to-spec)                 |
| 将规格说明拆分为 agent-ready 工单                                               | [to-tickets](https://aihero.dev/skills-to-tickets)           |
| 一个已确认的 bug，需要的是根本原因，而不是标签                                             | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

## 先决条件

`triage` 读取和写入你的问题跟踪器，所以 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须先配置该跟踪器及其标签词汇表。下面的角色名称是**规范的**。你跟踪器中的标签字符串可能不同，设置会提供映射。如果你的跟踪器已经完全使用规范名称，则无需映射，也无需设置。

跟踪器配置还决定外部 pull request 是否算作请求表面，以及谁算作外部。该标志默认关闭，设置不再询问它。要将 PR 纳入范围，请在 `docs/agents/issue-tracker.md` 中将其开启。

## 状态机

每个分类处理的项目最终都会有且仅有一个类别角色和一个状态角色。有两个类别：`bug`（某些东西坏了）和 `enhancement`（新功能或改进）。有五种状态：

| 状态                | 含义                                                                                     |
| ----------------- | -------------------------------------------------------------------------------------- |
| `needs-triage`    | 你需要评估它。未打标签的问题通常首先落在这里。                                                                |
| `needs-info`      | 等待报告人。他们回复后返回 `needs-triage`。                                                          |
| `ready-for-agent` | 完全明确，并附带 agent 简报。一个 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk)agent 可以接手。 |
| `ready-for-human` | 相同的简报，加上 agent 无法处理的原因：判断、外部访问、手动测试。                                                   |
| `wontfix`         | 已关闭，并记录了原因。                                                                            |

这就是全部词汇。每个项目恰好一个状态角色的规则保持了查询的简单性。状态也是 [skill](https://www.aihero.dev/ai-coding-dictionary/skill) 中最常被请求的领域。用户要求为已指定但被另一个问题阻塞的工作添加一个状态，为等待未来触发器的工作添加 `deferred` 状态，以及一个终态 `implemented` 状态。这些都没有发布。见下面的问题。

`wontfix` 有三种情况。区别很重要，因为只有其中一种会写入知识库：

| 你关闭它的原因  | 会发生什么                                                                         |
| -------- | ----------------------------------------------------------------------------- |
| 已实现      | 一条指向功能已存在位置的评论。没有任何东西进入 `.out-of-scope/`，因为它是一个已构建的功能，而不是被拒绝的功能，那里的文件会破坏去重检查。 |
| 被拒绝的 bug | 礼貌地解释，然后关闭。                                                                   |
| 被拒绝的增强   | 在 `.out-of-scope/`中创建一个文件，在关闭评论中链接它，然后关闭。                                     |

`.out-of-scope/` 每个被拒绝的**概念**存放一个 markdown 文件，而不是每个问题。每个文件是一个简短的设计文档，不是数据库行。它说明了拒绝什么、为什么拒绝，并列出了所有请求它的问题。`triage` 在评估任何东西之前会读取整个目录。它按概念匹配，而不是关键字，所以 "night theme" 会匹配 `dark-mode.md`。当它找到匹配时，会向你展示旧决定并询问你是否仍同意，而不是从头开始再次争论该请求。

## 简报前先验证

在任何 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 之前，`triage` 会检查声明是否成立。对于 bug，它会根据报告者的步骤复现。对于 PR，它会检出分支并运行相关测试。然后它报告三种结果之一：

* 已确认，附带代码路径。
* 无法复现。
* 细节不足无法尝试。这是最强的 `needs-info` 信号。

在同一遍中，它对代码库运行两项额外检查。**冗余**检查询问功能是否已实现，按领域概念搜索，而不是按报告者的措辞。**先前拒绝**检查询问 `.out-of-scope/` 是否已经说不。两项检查都很廉价，任一命中都会产生 `wontfix`。

所有这些工作只为制造一个优秀的工件：**agent 简报**。这是当问题移至 `ready-for-agent` 时 `triage` 发布的结构化评论。`triage` 发布后，简报就是契约，原始报告只是上下文。简报是**持久**而非精确的，因为问题可能在 `ready-for-agent` 中停留数周而代码在变化。所以简报命名类型、签名和行为契约，从不使用文件路径或行号。确认的复现比猜测能产生强得多的简报。

## PR 就是附带代码的 issue

如果跟踪器将外部 pull request 视为请求表面，它们会经过相同的机器，具有相同的类别、状态和转换。状态适用于 diff。`ready-for-agent` 意味着附带了简报，agent 应该在代码上采取下一步。`ready-for-human` 意味着人可以合并它。PR 上的简报描述现有 diff 剩下要做什么，而不是如何从零构建。

发现仅显示*外部* PR，因为协作者的进行中分支不是分类工作。该过滤器仅适用于发现。如果你显式指定一个 PR，`triage` 会处理它，无论谁写的。

## 常见问题

**我运行了 `/to-spec` 和 `/to-tickets`，现在那些工单就在那里没被分类。我要对它们运行 `/triage` 吗？**
不。它们已经是 agent-ready 的。`to-tickets` 在发布时应用 `ready-for-agent` 标签，所以 AFK runner 无需另一遍就能拾取它们。遇到这个问题的用户运行了 spec 流程并在输出上看到了 `needs-triage`，他们的 AFK runner 忽略了一切。`triage` 是来自外部工作的入口。spec 流程是你自己发起工作的车道。两者在 `ready-for-agent` 相遇，而不是之前。

**现在有了 `to-spec` → `to-tickets` → `implement` 流程，`triage` 还相关吗？**
只有当你有传入工作时。`triage` 比那个流程老，做不同的工作：它处理别人提交的报告。如果你跟踪器里的一切都来自你自己的规划，你很少会用到它。如果你维护任何公开项目，或你的团队向你提交 bug，那就是工作开始的地方。主要用例是接受外部贡献者问题的开源仓库。

**agent 尝试应用 `ready-for-agent` 但 `gh` 说标签不存在。**
这是一个已知的开放 bug ([#616](https://github.com/mattpocock/skills/issues/616))。`setup-matt-pocock-skills` 将标签词汇表写入 `docs/agents/triage-labels.md`，但不会在你的跟踪器中创建标签。用 `gh label create` 或跟踪器的 UI 自己创建那五个状态标签和两个类别标签一次，错误就会停止。该 issue 链接到一个未合并的社区修复分支。

**五个状态不够。那 blocked、deferred 或 implemented 呢？**
这是该 skill 上报得最多的缺口。它以三种形式出现：

* 一个已完全指定但等待另一个 issue 关闭的 issue ([#139](https://github.com/mattpocock/skills/issues/139))。报告者说 `ready-for-agent` 在那里是"技术上正确的"但具有误导性，所以代理会拾取它并卡住。
* 打算进行但等待触发器的未来工作，因此尚不可执行 ([#297](https://github.com/mattpocock/skills/issues/297))。
* 一个用于"已实现，等待验证"的终态。没有它，AFK 运行器会再次将完成的工单加入队列。

`blocked` 情况被接受为真实存在，但名称未定（`blocked` 还是 `paused`）。这些都未发布。作为变通方法，人们在类别旁添加一个仓库本地的额外标签。状态槽随后保存准确值，但技能不知道该额外标签。一个社区分支更进一步，添加了 `needs-slicing`、`tracking` 和工作量标签。这有效，但属于该分支，不属于该技能。

**这与 `/diagnosing-bugs` 有何不同？**
此处的验证步骤刻意浅显。它回答"这是真的吗，大致在哪里"，而不寻找根因。如果按报告者的步骤几分钟内无法复现 bug，请使用 `needs-info`，或使用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) 若你想立即调查。两个技能的文本目前都未提及对方。用户报告了这一空白，至今仍未解决。

**我能把它指向整个积压任务让它跑吗？**
你可以这么问，但要注意它读取什么。"显示需要关注的内容"这一遍历是用于*选择*的廉价列表。你挑选一个 issue，然后 `triage` 收集该 issue 的完整[上下文](https://www.aihero.dev/ai-coding-dictionary/context)。如果你一次跑二十个 issue，代理可能只用那份廉价列表作为唯一证据而不告诉你。列表只返回 issue 正文，不返回评论。一位用户正中此坑。三个 issue 已有评论说"已修复，建议关闭"，结果全被生成了新的代理简报。若要批量处理，请明确要求它必须阅读每个 issue 的评论。

**它适用于 Linear，或除 GitHub Issues 外的其他工具吗？**
适用。追踪器是配置项，而非硬编码假设。人们通过 `linear` CLI 在 Linear 上运行它，也在 GitLab 和 `.scratch/` 下的纯 Markdown 文件上运行。常见的分工是：Linear 用于 issue 和规划，GitHub 用于代码和 PR。提到"issue tracker"的技能映射到 Linear，提到"PR"的技能映射到 GitHub。本地 Markdown 追踪器有一个未修复的模板 bug：生成的文件可能包含两次验收标准，一次在顶层，一次在代理简报内 ([#200](https://github.com/mattpocock/skills/issues/200))。

## 如果它起作用了

* 它接触的每个项目最终都恰好拥有一个类别角色和一个状态角色，从不为零，也从不出现两个冲突的状态。
* 它给出带有推理的建议并停止。它不会重新贴标签就继续下一个。
* 它在任何东西到达 `ready-for-agent` 之前，已复现了 bug，或已检出并运行了 PR。
* 它编写的简报会指明类型和行为，且不包含文件路径和行号。
* 当六个月前拒绝的请求再次出现时，它会告知你并引用旧理由，而不是再次分诊。
* 它发布的每条评论都以 `> *This was generated by AI during triage.*` 开头。

## 它在系统中的位置

`triage` 是一个**入口坡道**，而非主链路中的一个步骤。主流程始于你的一个想法（grill、spec、tickets、implement、review）。`triage` 是处理来自他人工作的并行泳道。两条泳道终点相同：一个贴有 `ready-for-agent` 标签并附带简报的 issue。[implement](https://aihero.dev/skills-implement) 拾取它的方式，与拾取来自 [to-tickets](https://aihero.dev/skills-to-tickets) 的工单相同。当请求在 `triage` 能生成简报前需要更多细节时，`triage` 会同时运行 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling)，每轮一个问题，从而在你做决策时将其记录在 `GLOSSARY.md` 和 ADR 中。当你不确定自己处于哪条泳道时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
