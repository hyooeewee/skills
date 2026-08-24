## 它做什么

`triage` 会遍历你项目跟踪器上的 issue，将每个 issue 通过一个微小的**triage 角色**（一个类别角色和一个状态角色）状态机移动，并留下一个 agent-ready 简报、一个针对报告人的具体问题，或一个带有记录原因的已关闭 issue。

它仅用于**你未创建**的 issue。原始 bug 报告、传入的功能请求、未通知到达的外部 pull request：从外部进入跟踪器的工作，无论报告人将其留成什么形状。[to-tickets](https://aihero.dev/skills-to-tickets) 产生的 [Tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) 本质上已经是 agent-ready 的，在它们上面运行 `triage` 至多是一种浪费。规则很简单：`/triage` 仅用于传入的 issue，不用于你自己创建的 issue。

它区别于手工打标签的第二点是：它会给出建议并等待。它会告诉你它对类别和状态的判断及理由，以及它在代码库中的发现，而在你给出指示之前，它不会应用任何更改。

## 何时使用它

你通过输入 `/triage` 然后用自然语言描述你想要什么来调用它。[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 不会主动调用它。“展示任何需要我关注的东西”，“让我们看看 #42”，“把 #42 移到 ready-for-agent”。

| 你拥有什么                                                                 | 去哪里                                                          |
| --------------------------------------------------------------------- | ------------------------------------------------------------ |
| 一个满是他人原始报告的跟踪器                                                        | `/triage`                                                    |
| 自己有一个粗略的想法，什么都没写下来                                                    | [grill-with-docs](https://aihero.dev/skills-grill-with-docs) |
| 将一次已定型的对话转化为 [规格说明](https://www.aihero.dev/ai-coding-dictionary/spec) | [to-spec](https://aihero.dev/skills-to-spec)                 |
| 将规格说明拆分为 agent-ready 工单                                               | [to-tickets](https://aihero.dev/skills-to-tickets)           |
| 一个已确认的 bug，需要的是根本原因，而不是标签                                             | [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs) |

## 先决条件

`triage` 会读取并写入你的 issue 跟踪器，因此 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须首先配置好该跟踪器及其标签词汇表。下面的角色名称是**规范**的；你跟踪器中的标签字符串可能不同，映射关系由 setup 提供。如果你的跟踪器已经精确使用了规范名称，那就没有什么需要映射，也没有什么需要设置。

跟踪器配置还决定是否将外部 pull requests 计为请求表面，以及谁算作外部。该标志默认为关闭，不再是设置问题，因此如果你希望 PR 在范围内，请在 `docs/agents/issue-tracker.md` 中将其翻转。

## 状态机

每个经过 triage 的项目最终都恰好携带一个类别角色和一个状态角色。两个类别：`bug`（某样东西坏了）和 `enhancement`（新功能或改进）。五个状态：

| 状态                | 含义                                                                                     |
| ----------------- | -------------------------------------------------------------------------------------- |
| `needs-triage`    | 你需要评估它。未标记的 issue 通常首先落在这里。                                                            |
| `needs-info`      | 等待报告人。他们回复后返回 `needs-triage`。                                                          |
| `ready-for-agent` | 完全明确，并附带 agent 简报。一个 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk)agent 可以接手。 |
| `ready-for-human` | 同样的简报，加上为什么不能委派的原因：判断、外部访问权限、手动测试。                                                     |
| `wontfix`         | 已关闭，并记录了原因。                                                                            |

这就是全部的词汇表，"恰好一个状态角色"的不变约束是保持查询简单的关键。这也是该 [skill](https://www.aihero.dev/ai-coding-dictionary/skill) 中被要求最多的领域：用户要求为那些已经明确但被另一个 issue 阻塞的工作增加第六个状态，为受未来触发条件门控的 `deferred` 工作，以及一个终态的 `implemented` 状态。这些都没有推出。请看下面的问题。

`wontfix` 有三种情况，区别很重要，因为只有其中一种会写入知识库：

| 你关闭它的原因  | 会发生什么                                                                     |
| -------- | ------------------------------------------------------------------------- |
| 已实现      | 一条评论指向它已经存在的位置。不会向 `.out-of-scope/`，因为它是一个内置功能，而不是一个被拒绝的功能，把它放在那里会污染去重检查。 |
| 被拒绝的 bug | 礼貌地解释，然后关闭。                                                               |
| 被拒绝的增强   | 在 `.out-of-scope/`中创建一个文件，在关闭评论中链接它，然后关闭。                                 |

`.out-of-scope/` 是针对每个被拒绝的**概念**（而非每个 issue）的一个 markdown 文件，写成简短的设计文档而不是数据库行：被拒绝了什么、为什么，以及所有请求过它的 issue。`triage` 在评估任何东西之前会读取整个目录，并按概念而非关键词进行匹配，所以“night theme”会匹配 `dark-mode.md`。当它命中匹配时，它会显示旧的决定并询问你是否仍有同感，而不是从头重新争论请求。

## 简报前先验证

在进行任何 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 之前，`triage` 会检查该声明是否确实成立。对于 bug，它会按照报告人的步骤复现。对于 PR，它会检出分支并运行相关测试。然后它会报告三种情况中的哪一种发生了：已确认，并附带代码路径；无法复现；或者细节不足而无法尝试——这本身就是最强的 `needs-info` 信号。

它在同一遍遍历中对代码库运行了另外两项检查：**冗余性**（这是否已经实现，通过领域概念搜索而非报告人的措辞？）和**之前的拒绝**（`.out-of-scope/` 是否已经说了不？）。这两项都很便宜，且命中时都会产生 `wontfix`。

所有这一切的存在都是为了造就一件好的产物：**agent 简报**，即当 issue 移到 `ready-for-agent` 时发布的结构化评论。一旦发布，简报就是契约，原始报告只是背景。简报被写成**持久**而非精确的，因为一个 issue 可能停留在 `ready-for-agent` 中数周，而代码在其下方不断变化。因此它们命名类型、签名和行为契约，而绝不涉及文件路径或行号。已确认的复现比猜测能创造出强得多的简报。

## PR 就是附带代码的 issue

当跟踪器将外部 pull requests 视为请求表面时，它们会通过相同的机器运行，具有相同的类别、相同的状态和相同的转换。状态只是根据 diff 读取：`ready-for-agent` 意味着附带了简报，agent 应该在代码上采取下一步，`ready-for-human` 意味着准备好由人来合并。PR 上的简报描述的是对现有 diff 还要做什么，而不是如何从零开始构建该事物。

发现功能只显示*外部* PR，因为协作者的进行中的分支不是 triage 工作。那个过滤器仅用于发现，明确命名 PR 就会对其进行 triage，无论谁写的。一个粗糙的边缘情况：GitHub 模板的 external-PR 列出命令向 `gh pr list` 请求一个 `gh` 没有暴露的 `authorAssociation` 字段，所以写死的命令会直接失败（[#468](https://github.com/mattpocock/skills/issues/468)）。

## 常见问题

**我运行了 `/to-spec` 和 `/to-tickets`，现在那些工单就那样静静地放在那里，没有被 triage。我应该在它们上面运行 `/triage` 吗？**
不。它们已经是 agent-ready 的，因为 `to-tickets` 在发布时就会应用 `ready-for-agent` 标签，正是为了让 AFK runner 在不经过另一轮的情况下接住它们。遇到这个问题的用户运行了 spec 流程，在输出中看到了 `needs-triage`，却发现他们的 AFK runner 把所有东西都忽略了。`triage` 是来自外部的工作的入口；spec 流程是你发起的工作的车道。它们在 `ready-for-agent` 处相遇，而不是在此之前。

**既然现在有了 `to-spec` → `to-tickets` → `implement` 流程，`triage` 仍然相关吗？**
只有当你有入站工作时才相关。`triage` 先于那条主线存在，并且做的是不同的工作：它是其他人提交的报告的车道。如果你跟踪器中的一切都来自你自己的规划，你很少会打开它。如果你维护任何公开项目，或者你的团队向你提交 bug，它就是前门。主要用途是接受外部贡献者 issue 的开源仓库。

**代理尝试应用 `ready-for-agent` 标签，但 `gh` 提示该标签不存在。** 已知的未修复 bug（[#616](https://github.com/mattpocock/skills/issues/616)）。`setup-matt-pocock-skills` 会将标签词汇写入 `docs/agents/triage-labels.md`，但不会在你的问题追踪器中创建这些标签。你需要自己一次性创建五个状态标签和两个类别标签，使用 `gh label create` 或追踪器的界面即可，之后就不会再出现这个问题。该 issue 中链接了一个社区修复分支，但尚未合并。

**五个状态不够：那被阻塞的、被延后的、或者已实现的怎么办？**
这是该 skill 中被提出最多的缺口，有三种形状。一个完全明确但等待另一个 issue 关闭的 issue（[#139](https://github.com/mattpocock/skills/issues/139)），报告人的抱怨是 `ready-for-agent` 在那里“技术上是真的”但具有误导性，所以 agent 接管它后撞上了墙。预期但尚不可执行的受触发条件门控的未来工作（[#297](https://github.com/mattpocock/skills/issues/297)）。以及“已实现，等待验证”的终态，没有它，AFK runner 无法重新排队已完成的工单。Matt 同意被阻塞的情况是真实的，但对名称尚未决定（`blocked` 还是 `paused`）。这些都没有发布。人们使用的变通方法是使用类别旁边的仓库本地额外标签，这以 skill 不知道它为代价，保持了规范状态槽被诚实的东西占用。一个社区衍生品更进一步，添加了 `needs-slicing`、`tracking` 和 effort 标签。那行得通，但那是他们的，不是 skill 的。

**这与 `/diagnosing-bugs` 有什么不同？**
这里的验证步骤是有意做得很浅（足以回答“这是真的吗，大概在哪儿”），而不是为了找到根本原因。当 bug 在几分钟内无法从报告人的步骤复现时，诚实的做法是 `needs-info`，或者如果你想现在就去追踪它，就用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。这两个 skill 的文本目前都没有提到对方；一个用户发现了这个联系，而且问题仍未解决。

**我可以把它指向我的整个积压工作让它运行吗？**
你可以问，但要注意它读取了什么。“显示需要关注的内容”这一遍是一个便宜的列表，用于*选择*，你从中选一个，然后它收集你选中的那个的完整 [context](https://www.aihero.dev/ai-coding-dictionary/context)。一次性在二十个 issue 上运行它，agent 可以安静地退回到那个便宜的列表作为其证据库，这会返回 issue 正文但不返回评论。一个用户正好遇到了这种情况：三个 issue 已经带有评论说“已修复，建议关闭”，结果这三个都获得了新的 agent 简报。如果你想要批量通过，请明确说明评论必须逐个 issue 读取。

它能和 Linear 配合使用吗，或者支持 GitHub Issues 以外的任何工具吗？是的，跟踪器是配置，而不是硬编码的假设，人们也用它来处理 Linear（通过 `linear` CLI）、GitLab，以及 `.scratch/` 下的纯 markdown 文件。常见的分工是 Linear 用于 issues 和规划，GitHub 用于代码和 PRs：说“issue tracker”的技能映射到 Linear，说“PR”的技能映射到 GitHub。在本地 markdown 跟踪器上有一个未修复的模板 bug，生成的文件可能会携带两次验收标准，一次在顶层，一次在 agent 简报内（[#200](https://github.com/mattpocock/skills/issues/200)）。

## 如果它起作用了

* 它所处理的每个条目都以恰好一个类别角色和一个状态角色结尾，绝不为零，也没有冲突的两个状态。
* 它会给出带理由的建议并停下来，而不是重新打标签然后继续。
* 在任何内容达到 `ready-for-agent` 之前，bug 已经被复现，或者 PR 已经被检出并运行。
* 它编写的简报会指明类型和行为，且不包含文件路径和行号。
* 如果六个月前被拒绝的请求再次出现，它会说明这一点并引用旧理由，而不是重新做一次分流。
* 它发布的每条评论都以 `> *This was generated by AI during triage.*` 开头。

## 它在系统中的位置

`triage` 是一个**入口**，而不是主流程中的一个步骤。主流程源于你的想法（grill, spec, tickets, implement, review），而 `triage` 是为那些以不同方式到达的工作开辟的并行车道。它在同一个地方汇合：一个标记为 `ready-for-agent` 并带有简报的 issue，[implement](https://aihero.dev/skills-implement) 会像处理来自 [to-tickets](https://aihero.dev/skills-to-tickets) 的 ticket 一样接手它。当一个请求在简报之前需要打磨时，`triage` 会同时运行 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling)，一次进行一轮提问，这样决策就会在做出时落入 `CONTEXT.md` 和 ADRs 中。当你不确定自己处于哪个车道时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
