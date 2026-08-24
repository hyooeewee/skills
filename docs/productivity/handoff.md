## 它做什么

`handoff` 将你当前所在的对话压缩成一份 **handoff 文档**：一个 markdown 文件，写入你操作系统的临时目录而不是工作区，以便一个新的 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 可以读取它来接手工作。

它带来的好处是 **可移植性**，而不是压缩。这使得该技能比听起来要狭窄。只有当工作必须 *转移* 时，你才需要文件：转移到一个新的 [harness](https://www.aihero.dev/ai-coding-dictionary/harness)、一个新的目录、一位同事，或者你想分叉出的一个副任务。如果没有东西在转移，你不需要 handoff：留在 [session](https://www.aihero.dev/ai-coding-dictionary/session) 中，使用 `/clear`、一个 [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent) 和 `/compact` 就可以覆盖普通的阶段结束情况，而且 `/compact` 比这个技能更频繁地覆盖这种情况。

## 何时使用它

You invoke this by typing `/handoff`; the agent won't reach for it on its own. Pass a note about what the next session is for, and the document is written for it.

四种情况构成了全部触发条件：

| 情况                                                                                                               | 为何需要文件                                                                       |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| 交换 harness (Claude → Codex)&#xA;新的 harness 无法看到旧的 \[上下文]\(https\://www\.aihero.dev/ai-coding-dictionary/context) | 新的 harness 无法看到旧的 [上下文](https://www.aihero.dev/ai-coding-dictionary/context) |
| 移动到不同的目录或仓库                                                                                                      | 原型目录是常见情况                                                                    |
| 把工作发送给同事                                                                                                         | 他们需要可以阅读的东西                                                                  |
| 分叉出阶段中途发现的副任务                                                                                                    | 你继续工作；第二个 agent 接手分叉                                                         |

对于其他所有情况（相同的 harness，相同的目录，你已经完成了 [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) 并准备进入实现），`/compact` 是正确的选择。[ask-matt](https://aihero.dev/skills-ask-matt) 带有跨越阶段边界所有五个选项的有序树。

## 分叉是人们会跳过的用法

这个技能的描述读起来像是会话恢复：写一份摘要，在这里结束，在那里继续。这样读起来它像是一个更差的 `/compact`，所以会被略过。分叉场景才是值得了解的。你**留在自己的 session 中**，并把累积的上下文副本交给一个并行工作的第二个 agent。

这正是绕道 [prototype](https://aihero.dev/skills-prototype) 的用途。你正深入一场设计对话，遇到了一个只有运行代码才能解决的问题，而你不想把已经建立起来的对话线索花在弄清楚它上。移交给一个原型 session，得到答案，再把答案交回来，并从原始线索中引用它。两次跨越，一次实时对话，无需重新解释任何内容。

阶段边界上的五个选项中有三个保留不同的东西：`/compact` 保留你的意图，`/clear` 什么都不保留，`/handoff` 保留工作的可移动性。

## 什么会转移，什么不会

文档携带实时线程（正在进行的内容、原因和下一步），以及一个 **suggested skills** 部分，命名下一个 agent 应该调用什么。在写入之前，机密信息会被删除。

它刻意不携带任何已经写下来的内容。规格、计划、ADR、issue、提交和 diff 通过路径或 URL 引用，从不复制。这保持了文件的精简，也让已确定的细节只存在于一处，而不是两处逐渐偏离的地方。

## 常见问题

**Handoff 还是 compact？**
除非有什么东西在转移，否则使用 `/compact`。停留在同一项任务上是 compact，而不是 handoff：相同的 harness，相同的目录，而且你需要保持参与其中，这是阶段边界树在大多数日子里落下的地方。`/handoff` 的优势不在于它总结得更好；而在于结果是你可以带到 `/compact` 无法到达的地方的一个文件。

**那么 compact、clear 和 handoff 之间的实际区别是什么？**
正在保存三件不同的事情。`/compact` 压缩此上下文并在一个新窗口中让你继续：意图得以保留。`/clear` 清空窗口并从头开始：当你身后的一切都是可抛弃的时候是正确的，如果不是可抛弃的则是单向的。`/handoff` 写入一个可移植的文件：工作在转移到别处时得以幸存。请注意，这三者都将 **[primary source](https://www.aihero.dev/ai-coding-dictionary/primary-source)**（实际发生的对话）变成了 **[secondary source](https://www.aihero.dev/ai-coding-dictionary/secondary-source)**（它的摘要）。继续是唯一没有这样做的方法，这就是为什么它是第一个被排除的方法。

**我的 handoff 文件去哪了？**
临时目录，这是该技能报告最多的摩擦点：路径很长，它们因操作系统而异，而且在 Windows 上，agent 有时需要尝试几次才能找到正确的那个。请求路径并保留它，然后再继续。使用临时目录是有意为之：handoff 是一份过境文档，而不是你维护的工件。它也不是持久的；请看下一个问题。

**我的 handoff 在会话之间消失了。**
某些环境会在会话之间清除临时文件（Codex 是报告的案例），并且 `/private/tmp` 在重启后会消失。如果下一个会话不是在一小时内开始，或者是在不同的 harness 下开始，请在文件写入后尽快将其复制到某个持久的地方。这也适用于文档 *指向* 的任何内容：引用临时文件中其他文件的分派是下一个 agent 无法跟随的分派。

**我实际上如何把它交给下一个 agent？**
打开新会话并将其指向路径：读取此文件，然后继续。指向文件而不是将摘要粘贴到 shell 命令中：包含反引号或 `$(...)` 的摘要在插入到 `claude "<summary>"` 中时会被破坏，通常的失败是静默截断而不是错误，因此新的 agent 以一个安静不完整的简报开始。

**这和 `/branch`、`--fork-session` 或内置的 `/handoff` 一样吗？**
类似但不相同，而且 `/branch` 不是这里已发布的一个技能；`/handoff` 是规范名称。分叉继承上下文的精确副本；这个技能在一个文件中产生针对已声明下一个任务的 *针对性* 压缩。分叉能做的地方（同一台机器，相同的 harness，相同的目录），分叉的工作量更小。一旦目的地是分叉无法去的地方，文件就赢了。

**什么时候内容应该放在 `CLAUDE.md` 里？**
问问它下个月是否仍然成立。`CLAUDE.md` 是关于项目的长期上下文，无论是否相关都会被加载到每个 session 中。handoff 关于一项进行中的工作，一旦这项工作完成它就失效了。那些不断被重新解释的事实是 `CLAUDE.md` 的问题；一个半成品任务是 handoff。

**它捕捉的是“是什么”，而不是“为什么”。**
一个公平且反复出现的批评。有两件事有帮助。传递参数（告诉它下一个会话的目的是什么），这样与 *那个* 相关的推理就会被保留而不是被扁平化。还要注意 session 从未实际验证过的自信声明：“X 没有构建”，“Y 已完成”。下一个 agent 将文档视为合同，不会重新检查它，因此写为事实的信念变成了后续内容的错误前提。在你移交之前阅读文档，并降级你只是假设的内容。

**为什么它是一个技能而不是斜杠命令？**
两者都有效；它们适合不同的情况。作为一个技能，它通过与其他所有内容相同的安装路径进行分发和更新，这就是使其可共享的原因；agent 不会自己触发它的约束是由其 frontmatter 设置的，而不是由机制设置的。

## 如果它起作用了

* 文档只占对话的一小部分，规格、issue 和 diff 以路径和 URL 的形式出现在其中，而不是复制的文本。
* 你可以在没有打开原始 session 的情况下直接阅读它，并知道接下来该做什么。
* 新的 agent 直接开始工作，而不是要求你重新解释设置。
* 在分叉的情况下，当你回来时，你的原始 session 仍然原封不动地在那里。
* suggested-skills 部分列出了你自己会去使用的技能。
* 其中没有任何密钥、令牌或密码。

## 它在系统中的位置

`handoff` 是一个 **随时可用的独立工具**，它存在于会话的接缝处而不是构建链内部，但这是一个狭窄的工具，诚实的说法是你会在阶段边界上比其他四个选项更少地使用它。它最近的邻居是 [prototype](https://aihero.dev/skills-prototype)，因为原型生活在自己的目录中，来回往返正是这个技能所跨越的。当你处于边界且不确定是继续、清除、移交、委托还是压缩时，[ask-matt](https://aihero.dev/skills-ask-matt) 带有排序这五个选项的树，并将你引导到集合的其余部分。
