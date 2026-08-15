## 它做什么

`triage` 会遍历你项目跟踪器上的 issue，让每一个经过一个由 **triage 角色** 组成的小型状态机——一个类别角色和一个状态角色——并留下一个可供 agent 直接使用的简报、一个向报告人提出的具体问题，或一个带着已记录原因的已关闭 issue。

它只针对**你没有创建**的 issue。原始的 bug 报告、新来的功能请求、一个未经预告就到达的外部拉取请求——这些从外部进入跟踪器的工作，无论报告人以何种形式留下它。[Tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 由 [to-tickets](https://aihero.dev/skills-to-tickets) 生成，天生就是 agent-ready 的，对它们运行 `triage` 最多只是浪费工作。规则很简单：`/triage` 只用于传入的 issue，不用于你自己创建的 issue。

它区别于手工打标签的第二点是：它会给出建议并等待。它会告诉你它对类别和状态的判断及理由，以及它在代码库中的发现，而在你给出指示之前，它不会应用任何更改。

## 何时使用它

你通过输入 `/triage` 然后用自然语言描述你想要什么来调用它——[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会主动使用它。“给我看任何需要我注意的东西”、“我们看看 #42”、“把 #42 移到 ready-for-agent”。

| 你拥有什么                                                                 | 去哪里                                                          |
| --------------------------------------------------------------------- | ------------------------------------------------------------ |
| 一个满是他人原始报告的跟踪器                                                        | `/triage`                                                    |
| 自己有一个粗略的想法，什么都没写下来                                                    | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 将一次已定型的对话转化为 [规格说明](https://www.aihero.dev/ai-coding-dictionary/spec) | [to-spec](https://aihero.dev/skills-to-spec)                 |
| 将规格说明拆分为 agent-ready 工单                                               | [to-tickets](https://aihero.dev/skills-to-tickets)           |
| 一个已确认的 bug，需要的是根本原因，而不是标签                                             | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

## 先决条件

`triage` 会读取并写入你的 issue 跟踪器，因此 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须首先配置好该跟踪器及其标签词汇表。下面的角色名称是**规范**的；你跟踪器中的标签字符串可能不同，映射关系由 setup 提供。如果你的跟踪器已经精确使用了规范名称，那就没有什么需要映射，也没有什么需要设置。

跟踪器配置还决定外部拉取请求是否算作请求面，以及谁算作外部。“该标志默认为关闭，不再是一个设置问题——如果你希望将 PR 纳入范围，请在 `docs/agents/issue-tracker.md` 中启用它。

## 状态机

每个经过 triage 的项目最终都恰好携带一个类别角色和一个状态角色。两个类别：`bug`（某样东西坏了）和 `enhancement`（新功能或改进）。五个状态：

| 状态                | 含义                                                                                     |
| ----------------- | -------------------------------------------------------------------------------------- |
| `needs-triage`    | 你需要评估它。未标记的 issue 通常首先落在这里。                                                            |
| `needs-info`      | 等待报告人。他们回复后返回 `needs-triage`。                                                          |
| `ready-for-agent` | 完全明确，并附带 agent 简报。一个 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk)agent 可以接手。 |
| `ready-for-human` | 同样的简报，外加为什么这不能委派——判断力、外部访问权限、手动测试。                                                     |
| `wontfix`         | 已关闭，并记录了原因。                                                                            |

这就是全部的词汇表，"恰好一个状态角色"的不变约束是保持查询简单的关键。这也是该 [skill](https://www.aihero.dev/ai-coding-dictionary/skill) 中被要求最多的领域：用户要求为那些已经明确但被另一个 issue 阻塞的工作增加第六个状态，为受未来触发条件门控的 `deferred` 工作，以及一个终态的 `implemented` 状态。这些都没有推出。请看下面的问题。

`wontfix` 有三种情况，区别很重要，因为只有其中一种会写入知识库：

| 你关闭它的原因  | 会发生什么                                                                           |
| -------- | ------------------------------------------------------------------------------- |
| 已实现      | 一条评论指向它已经存在的位置。不会向 `.out-of-scope/`写入任何内容——这是一个已构建的功能，而不是被拒绝的功能，将其归档到那里会污染去重检查。 |
| 被拒绝的 bug | 礼貌地解释，然后关闭。                                                                     |
| 被拒绝的增强   | 在 `.out-of-scope/`中创建一个文件，在关闭评论中链接它，然后关闭。                                       |

`.out-of-scope/` 是每个被拒绝的**概念**对应一个 markdown 文件，而不是每个 issue 一个，以简短的设计文档而非数据库行的形式编写：被拒绝的内容、原因，以及所有提出过该请求的 issue。`triage` 在评估任何内容之前会读取整个目录，并按概念而非关键词进行匹配——“夜间主题”会匹配 `dark-mode.md`。当它找到匹配项时，会呈现旧的决策并询问你是否仍然持相同看法，而不是从头重新争辩这个请求。

## 简报前先验证

在进行任何 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 之前，`triage` 会检查该声明是否确实成立。对于 bug，它会按照报告人的步骤复现。对于 PR，它会检出分支并运行相关测试。然后它会报告三种情况中的哪一种发生了：已确认，并附带代码路径；无法复现；或者细节不足而无法尝试——这本身就是最强的 `needs-info` 信号。

在同一次检查中，它还会对代码库运行两项额外检查——**冗余性**（是否已经实现，按领域概念而不是按报告人的措辞搜索？）和**先前拒绝**（`.out-of-scope/` 是否已经说了不？）。两者都很廉价，并且命中时都会产生 `wontfix`。

所有这一切的存在都是为了造就一件好的产物：**agent 简报**，即当 issue 移到 `ready-for-agent` 时发布的结构化评论。一旦发布，简报就是契约，原始报告只是背景。简报被写成**持久**而非精确的，因为一个 issue 可能停留在 `ready-for-agent` 中数周，而代码在其下方不断变化。因此它们命名类型、签名和行为契约，而绝不涉及文件路径或行号。已确认的复现比猜测能创造出强得多的简报。

## PR 就是附带代码的 issue

在跟踪器将外部拉取请求视为请求面的情况下，它们会经过同样的机器——相同的类别、相同的状态、相同的转换。状态只是对照 diff 来解读：`ready-for-agent` 意味着已附加简报，agent 应该对代码采取下一步行动，`ready-for-human` 意味着它已经准备好由人来合并。PR 上的简报描述的是对现有 diff 还需要做什么，而不是如何从零开始构建。

发现功能只显示*外部* PR，因为协作者在途的分支不是 triage 工作。该过滤器仅用于发现——明确指定一个 PR，无论谁写的它都会被 triage。一个粗糙的边缘情况：GitHub 模板的外部 PR 列命令要求 `gh pr list` 提供一个 `gh` 并不公开的 `authorAssociation` 字段，因此该命令按原样会直接失败（[#468](https://github.com/mattpocock/skills/issues/468)）。

## 常见问题

**我运行了 `/to-spec` 和 `/to-tickets`，现在那些工单未经 triage 停留在那里。我需要对它们运行 `/triage` 吗？**
不需要。它们已经是 agent-ready 的——`to-tickets` 在发布时就会应用 `ready-for-agent` 标签，正是为了让 AFK 运行器无需再次经过就能拾取它们。遇到这个问题的用户运行了 spec 流程，看到输出上的 `needs-triage`，并发现他们的 AFK 运行器忽略了一切。`triage` 是从外部到达的工作的上坡道；spec 流程是你在组织内发起的工作的车道。它们在 `ready-for-agent` 汇合，而不是之前。

**既然现在有了 `to-spec` → `to-tickets` → `implement` 流程，`triage` 仍然相关吗？**
只有当你有入站工作时才相关。`triage` 先于那条主线存在，并且做的是不同的工作：它是其他人提交的报告的车道。如果你跟踪器中的一切都来自你自己的规划，你很少会打开它。如果你维护任何公开项目，或者你的团队向你提交 bug，它就是前门。主要用途是接受外部贡献者 issue 的开源仓库。

**代理尝试应用 `ready-for-agent` 标签，但 `gh` 提示该标签不存在。** 已知的未修复 bug（[#616](https://github.com/mattpocock/skills/issues/616)）。`setup-matt-pocock-skills` 会将标签词汇写入 `docs/agents/triage-labels.md`，但不会在你的问题追踪器中创建这些标签。你需要自己一次性创建五个状态标签和两个类别标签，使用 `gh label create` 或追踪器的界面即可，之后就不会再出现这个问题。该 issue 中链接了一个社区修复分支，但尚未合并。

**五个状态不够用——那 blocked、deferred 或 implemented 呢？** 这是该技能上被反馈最多的缺口，表现为三种形式。一个是已完全明确但在等待另一个 issue 关闭（[#139](https://github.com/mattpocock/skills/issues/139)）——报告者抱怨说 `ready-for-agent` 在那里“技术上没错”但有误导性，于是代理接单后碰壁。一个是触发器门控的未来工作，虽然有意图但尚不可执行（[#297](https://github.com/mattpocock/skills/issues/297)）。还有一个是用于“已实现，等待验证”的终态，否则 AFK 运行器可能重新排队已完成的任务。Matt 已同意 blocked 的情况是真实的，但对命名（`blocked` 与 `paused`）尚未决定。这些都没有发布。人们使用的变通方法是在类别旁添加一个仓库本地的额外标签，这让规范状态槽位保持诚实，但代价是该技能不知道这个标签。一个社区衍生品更进一步，增加了 `needs-slicing`、`tracking` 和工作量标签——这有效，但那是他们的，而不是该技能的。

**这与 `/diagnosing-bugs` 有何不同？** 这里的验证步骤刻意保持浅层——足以回答“这是不是真的，大致在哪”，而不是找出根本原因。如果一个 bug 在几分钟内无法根据报告者的步骤复现，诚实的做法是标记 `needs-info`，或者如果你想现在就追查，可以使用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。目前两个技能的文字都没有提及对方；一位用户发现了这个缝隙，它至今仍未处理。

**我可以让它处理整个待办列表并放手运行吗？** 你可以这样要求，但要注意它读取了什么。“显示需要关注的内容”这一步是一个廉价的列表，用于*选择*——你挑一个，然后它才会针对你选的那个收集完整的[上下文](https://www.aihero.dev/ai-coding-dictionary/context)。如果一次性跑二十个 issue，代理可能会悄悄以那个廉价列表作为证据基础，这只会返回 issue 正文而不会返回评论。就有用户遇到了这种情况：三个 issue 都已经带有一条“已修复，建议关闭”的评论，但三个还是都被生成了新的代理简报。如果你想要批量处理，请明确说明必须逐 issue 阅读评论。

**它能用于 Linear 或 GitHub Issues 以外的其他工具吗？** 可以——追踪器是可配置的，不是硬编码假设，人们会用它对接 Linear（通过 `linear` CLI）、GitLab，以及 `.scratch/` 下的纯 markdown 文件。一种常见的分工是：issue 和规划用 Linear，代码和 PR 用 GitHub：提到“issue tracker”的技能映射到 Linear，提到“PR”的技能映射到 GitHub。在本地 markdown 追踪器上有一个未修复的模板 bug，生成的文件可能将验收标准写了两次，一次在顶层，一次在代理简报内部（[#200](https://github.com/mattpocock/skills/issues/200)）。

## 如果它起作用了

* 它接触的每个条目最终都恰好有一个类别角色和一个状态角色——绝不会为零，也绝不会出现两个冲突的状态。
* 它会给出带理由的建议并停下来，而不是重新打标签然后继续。
* 在任何内容达到 `ready-for-agent` 之前，bug 已经被复现，或者 PR 已经被检出并运行。
* 它编写的简报会指明类型和行为，且不包含文件路径和行号。
* 如果六个月前被拒绝的请求再次出现，它会说明这一点并引用旧理由，而不是重新做一次分流。
* 它发布的每条评论都以 `> *This was generated by AI during triage.*` 开头。

## 它在系统中的位置

`triage` 是一个**入口匝道**，而不是主链路上的一个步骤。主流程从一个想法开始——打磨、规格、工单、实现、审查——而 `triage` 是处理“外来”工作的并行车道。它汇合到同一处：一个标记为 `ready-for-agent` 并带有简报的 issue，[implement](https://aihero.dev/skills-implement) 会像处理来自 [to-tickets](https://aihero.dev/skills-to-tickets) 的工单一样接起它。当一个请求在形成简报前需要打磨时，`triage` 会同时运行 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling)，一轮一轮地提问，从而让决策在做出时就落入 `CONTEXT.md` 和 ADR 中。当你不确定自己处于哪条车道时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
