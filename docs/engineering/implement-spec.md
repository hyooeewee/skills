## 它做什么

`implement-spec` 接收一个 [spec](https://www.aihero.dev/ai-coding-dictionary/spec) 及其 [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket)，并在一次运行中将整个工作落地。编排用的 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 把每张票交给一个实现者 [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent)，后者在各自的 git worktree 中工作，随后将完成的分支合并到同一条 **integration branch**，对结果运行 [code-review](https://aihero.dev/skills-code-review)，最后把票标记为已解决。

它把 tickets 视作 **任务图**，而非列表。阻塞边决定什么可以启动，因此任意时刻都存在一个 **frontier（前沿）**，由所有阻塞项已落地的票组成，前沿上的每张票同时运行。这与逐张处理票的区别在于：图的形状决定节奏，而非追踪器上票的顺序。

## 何时使用它

你通过输入 `/implement-spec` 来调用它，agent 不会自行触发该技能。

| 你的情况                                                                                                                                               | 选择                                                   |
| -------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 一个已拆分为带阻塞边的 tickets 的 spec，你想在一次运行中全部落地                                                                                                            | `/implement-spec`                                    |
| 一次一张票，在你自己的 [上下文窗口](https://www.aihero.dev/ai-coding-dictionary/context-window), [清除](https://www.aihero.dev/ai-coding-dictionary/clearing)之间清除上下文 | [implement](https://aihero.dev/skills-implement)     |
| 尚未拆分为 tickets 的 spec                                                                                                                               | [to-tickets](https://aihero.dev/skills-to-tickets)优先 |
| 没有真正图结构的小块工作                                                                                                                                       | [implement](https://aihero.dev/skills-implement)直接   |

## 先决条件

* **议题追踪器。** 该技能从 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 配置的追踪器读取 tickets 并在其上解决它们。若未配置追踪器，它会停止并提示你先运行该设置，而不是盲目猜测。
* **带阻塞边的 tickets**，即 [to-tickets](https://aihero.dev/skills-to-tickets) 生成的形式。没有边的话图就是扁平的，所有票会同时启动。
* **能在后台运行 subagent 并为每个 subagent 提供 git worktree 的 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)。** 该技能的存在是为了并行跑票，所以在一次只跑一个 subagent 的 harness 上，它只是个更慢的 `implement`。

## 集成分支

所有工作落在同一条分支上。每个实现者：

1. 启动前确认其 worktree 基于集成分支，
2. 用 [tdd](https://aihero.dev/skills-tdd) 构建自己的票，红-绿一次一个切片，
3. 在报告完成前把集成分支最新提交合并进自己的分支，使落地成为快进合并。

追踪器决定是否需要拉取请求。如果你的追踪器通过 PR 关闭工作，或你主动要求，技能会在首次合并后打开草稿 PR 并在最后标记为就绪。否则运行停留在集成分支上，每张票按追踪器的方式解决，完全脱机工作在本地 markdown 追踪器上也没问题。

实现者通过 [context pointers](https://www.aihero.dev/ai-coding-dictionary/context-pointer)（spec、ticket、共享探索笔记、早期提交）与编排者通信，而不是粘贴摘要。这让每个 subagent 的提示词保持精简，并在编排者的窗口中为图留出空间。

## 常见问题

**这与我在每张票上运行 `/implement`有何不同？**

这正是该技能存在的意义。发布前，大家不断自造轮子，一位用户描述需求：他们想要“subagents 实现 tickets”，而不是“当一个 spec 包含 5 张以上票时，不得不逐个创建新 session 并让它们一张张实现”。用 `implement` 时你是分发者：每张票一个 [session](https://www.aihero.dev/ai-coding-dictionary/session)，中间清除上下文，自己跟踪哪些票已解除阻塞。`implement-spec` 把这活交给单个编排 session。代价是你不再看到每张票落地时的工作；你在最后审查集成分支。要开始运行，清空上下文并输入 `/implement-spec` 加上指向 spec 的指针（议题编号或文件路径）。对于没有真正图结构的小改动，跳过它直接用 `implement`。

**它需要 GitHub 吗？我希望它停在分支上。**

不需要了。一位喜欢在开发版的用户正是抱怨这一点：“它最后会创建 PR，这要求像 GitHub 这样的在线仓库。我希望它能离线完成相同工作，并停在所有工作已合并的分支上。” 现在运行结束于集成分支。只有当配置的追踪器通过 PR 关闭工作、或你主动要求时才会打开 PR，因此在本地 markdown 追踪器上，运行会以所有票已解决、工作已合并到分支上而结束。

**其审查-修复循环跑了数小时，或持续“修复”尚未构建的票。**

这两种情况都发生在 `code-review` 跑在技能指定的唯一节点之外时。它拿代码与整个 spec 比对，所以只有在所有票都落地后运行才有意义。如果在运行中途跑，每张未构建的票都会读作失败。Agent 接着去构建那张票，又触发一次审查。最后，技能跑一次 `code-review` 并把所有发现发给一个修复 subagent，但目前不会在修复后说明何时停止。一位用户报告一个五票特性“审查-修复循环约耗时四小时”。如果你看到第二轮大面积审查开始，让它针对已修复的发现跑聚焦检查并停下。预期第一轮审查会发现真问题。运行产出是草稿，由审查完成它，不能直接发布。

**它像 implement 那样驱动 tdd 吗？**

现在是的，最初不是。跑在开发版的用户注意到“实现者 subagent 没有继承 /tdd 指令”，所以一旦从单票扩展到整个 spec，红-绿就停了。现在每个实现者都用 `tdd` 构建自己的票。仍没有像 `implement` session 那样让你交互式确认接缝的步骤，所以如果想钉住接缝，请在 spec 或票里写明。

**两个并行运行的实现者在同一文件上冲突，或给同一事物取了不同名字。**

Worktree 不消除冲突，只把冲突推迟到合并时。根据票文本写出的阻塞边只是对各票将触及文件的猜测，两张处于“代码库不同部分”的票仍可能共享消息目录、配置注册表或类型。每个实现者只看到自己的票和共享笔记，永远看不到别人的进行中工作，所以有用户的 web 和 mobile 票分别把同一字符串加为 `blockedSince` 和 `blockedOn`。当两张前沿票触及同一文件，要么在它们之间加阻塞边让它们串行跑，要么让探索笔记固定每张票要添加的确切名称。

**阻塞票永远不启动，即使其阻塞项已合并。**

这是 GitHub 上的已知问题。追踪器的 blocked-by 计数仅在阻塞项*关闭*时递减，而票通常在 PR 合并时关闭，那是运行结束时刻。追踪器是起始图的正确来源，但运行中途它是陈旧的。让编排器自己跟踪哪些票已合并进集成分支，并以此计算前沿。

**这能替代 Sandcastle 或 AFK 脚本吗？**

不能。大家问是因为技能现已触达实现层：“Sandcastle 还相关吗？你们的技能似乎也能处理实现了。” `implement-spec` 让一个 agent 在单个 harness session 内负责编排，无需基础设施，且可观察和干预。对于真正 [AFK](https://www.aihero.dev/ai-coding-dictionary/afk) 的工作，确定性循环（[Sandcastle](https://github.com/mattpocock/sandcastle)、shell 脚本、CI 作业）更快、更省、更可靠，因为没有 agent 做编排决策。

**一张票的关键测试在其 worktree 内被跳过，却报绿。**

Worktree 仅包含 git 跟踪的内容。读取 gitignore 的 fixture、本地数据库或凭据的测试可能在那里静默跳过。对于验证依赖未跟踪材料的票，请让编排器在主检出目录运行它。

## 如果它起作用了

* 只要图允许，多个实现者会同时运行，而不是一个接一个。
* 只要其最后一个阻碍项落在集成分支上，工单就会立即开始，而不是等到整个运行结束。
* 每个工单的追踪都显示 `tdd` 正在运行，且在代码之前有一个失败的测试。
* 合并到集成分支是快进合并，而不是冲突解决。
* 运行在一个分支上结束，所有工单都已解决，仅当你的追踪器需要时才会创建 PR。

## 它在系统中的位置

`implement-spec` 是主链的构建步骤，作为并行替代方案，对比每个工单运行一次 [implement](https://aihero.dev/skills-implement)：

```txt
grill-with-docs → to-spec → to-tickets → implement-spec → retro
```

它的邻居是 [to-tickets](https://aihero.dev/skills-to-tickets)，它声明作为任务图读取的阻塞边，以及 [code-review](https://aihero.dev/skills-code-review)，它在收尾前在集成分支上运行。当你不确定处于哪个流程时，[ask-matt](https://aihero.dev/skills-ask-matt) 是整个集合的路由器。
