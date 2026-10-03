## 它做什么

`retro` 会回顾一个编码 [session](https://www.aihero.dev/ai-coding-dictionary/session)，并针对 agent 的 **[environment](https://www.aihero.dev/ai-coding-dictionary/environment)** 提出改进建议，以便下一次运行更顺利。它读取该 session 自身的记录（默认为当前 session，或你在 session 日志中指定的某个 session），找出 agent 吃力的时刻，并按严重程度从高到低给你一份候选修复清单。

它改变的是环境，而非代码。agent 交付的 bug、花了二十次 [tool calls](https://www.aihero.dev/ai-coding-dictionary/tool-call) 才找到的文件、reviewer 漏掉的规则：`retro` 不会原地修复它们。它追问是什么让这些问题在仓库里得以发生，并提出能防止再次发生的检查、指针或标准。它也仅提出建议；除非你选中某个候选项，否则什么都不会变。

## 何时使用它

你通过输入 `/retro` 来调用它，agent 不会自行发起。

在感觉比应有难度大的 session 结束时使用它：agent 找东西找太久、犯了机器本能捕捉的错误、或需要却无法获取的信息。顺滑的 session 没什么可教的；痛苦的 session 才是发现的来源。如果你想要对 session 产出的代码做定论，请改用 [code-review](https://aihero.dev/skills-code-review)。

## 发现落地之处

每个候选项属于某一类别，类别决定修复去向：

| session 中出了什么问题            | 用什么修复                                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| agent 花了很长时间才找到某个文件或事实     | 一个 **导航指针**来自它已读的某个文件                                                                                              |
| 它犯了个工具本能捕捉的错误              | 一个 **[自动化检查](https://www.aihero.dev/ai-coding-dictionary/automated-check)**：lint 规则、类型检查、测试、pre-commit hook、CI job |
| reviewer 漏判了一个判断力型错误       | 在 `CODING_STANDARDS.md`中为 reviewer agent 添加一条规则                                                                    |
| `AGENTS.md`或 `CLAUDE.md`过大 | 将其引导内容迁出，放入标准或检查中                                                                                                  |
| 某次 tool call 代价高但回报低       | 精简该工具，或替换它                                                                                                         |
| 某引导文件全是不起作用的行              | 删除 **无效行**                                                                                                         |
| agent 需要却触达不到的信息           | 扩大其访问权：把 dev server 日志 tee 到文件、给某服务只读权限                                                                            |

核心思想是：标准属于 **reviewer**，不属于 implementer。实现型 agent 承担最大的上下文压力：它探索、写代码、调试失败。审查型 agent 只收到一个 diff。因此新规则应放在有空间应用它的审查环节，而绝不放进 [AGENTS.md](https://www.aihero.dev/ai-coding-dictionary/agents-md)——后者会加载进每个 session 的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)，无论相关与否。

写规则前，先给违规分类。**机械性** 违规（禁用 API、导入结构、文件位置规则）得到确定性检查，因为检查会失败，而标准文里的一句话不会。只有真正的判断力型决策、任何 linter 都无法强制执行的才变成文本。完全没有护栏的仓库（无 pre-commit hook、无跑 lint/typecheck/test 的 CI job）本身就会被报告为一条发现。

## 常见问题

**它会自己写 lint 规则，还是等确认？能不能接在每个 session 后自动跑？**

它会等。`retro` 只提议；除非你选中候选项，否则无任何变更，因此无手工编辑、也无自动应用的 hook。这是刻意设计：有用户正是因「被自动 hook 挡住好改动而受伤」才要求如此。决定什么值得永久检查需要判断力，所以该技能保持 [human-in-the-loop](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop) 且由用户调用。部分用户确实在每次实现运行后串联它，但顺滑的 session 少有可教之处，每次都跑多半只会产出没人需要的规则。没有 dry-run 模式：提议的检查像普通代码一样构建，先在仓库试跑，再决定是否让它拦截合并。

**这会无限堆积 lint 规则吗？它会建议删规则吗？**

部分会，这是它最弱之处。它能覆盖的删除面仅限于文本：引导文件里的无效行、属于标准或检查却留在 `AGENTS.md` 或 `CLAUDE.md` 的引导内容。文件较大时它会结合正在读的 session 标记这些待删项，视为删除测试的候选而非定论。它不会审计上月它自己提议的 lint 规则、hook 或 CI job。它只见一个 session，无法告诉你某规则已变吵闹或早已过时。清理检查仍是你的活；规则频繁在好代码上触发，就是信号。

**它会不会为了填满分类而编造通用建议？**

这是它面临最尖锐的批评。有用户发现：「任务一结束，AI 就倾向于忘掉 session 中期的挣扎，转而编造通用建议以满足 retro 分类。」抗辩在于：每个候选必须来自该 session 自身记录，所以建议针对该 session。这切两面：极少幻觉出无关建议，但可能过度聚焦于这一个 session 碰巧涉及的内容。丢弃任何无法追溯到具体时刻的候选。把严重度排序也视为草稿：安静且昂贵的错误可能排在吵闹但廉价的错误之后。

**我的 session 很长。现在跑，还是重开？**

默认复盘当前 session，这是最佳情况：挣扎还在 context window 里。若 session 已漂出 [smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)，请 [clear](https://www.aihero.dev/ai-coding-dictionary/clearing) 后，在 session 日志中指向上一个 session 重新跑 `/retro`。

**agent 老犯同一个错。我该不该在 `CLAUDE.md`？**

Usually not, and that's the most common place `retro` pushes back. A line in `CLAUDE.md` is loaded into every session, dilutes everything else in the file, and drifts as the code changes. If the mistake is mechanical, the fix is a check that fails. If it's a judgement call, it goes in the coding standards the reviewer reads. `AGENTS.md`和 `CLAUDE.md` are for navigation pointers, and little else. For the same reason `retro` is not a [memory system](https://www.aihero.dev/ai-coding-dictionary/memory-system): it doesn't store what happened, it changes the environment so it can't happen again.

**我的配置提到了 `CODING_STANDARDS.md`可我没有这个文件。它从哪来？**

没有任何东西会自动生成该文件。当某 session 首次为 reviewer 产出一条判断力型规则时，`retro` 会提议创建它；一旦你接受，[code-review](https://aihero.dev/skills-code-review) 便会从此读取它。你已有的任何其他标准文档（如 `CONTRIBUTING.md`）同理。

**它和 \`improve-codebase-architecture\` 有何不同？ `improve-codebase-architecture`？**

输入不同。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 只需代码，寻找代码本身的结构性改进。`retro` 需要 session 历史，改进的是 agent 的工作环境而非代码。两者并存；互不替代。

## 如果它起作用了

* 每个候选都指向 session 中的具体时刻，而非通用最佳实践。
* 重复错误变成失败的检查，且你的 `AGENTS.md` 随时间变短而非变长。
* 已存在但未接线的缺失检查作为发现浮现，而非提议新建一个。
* 同类任务的下一个 session 找路更快。

## 它在系统中的位置

`retro` 是主链的最后一步，流程在此回头审视自己：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

在值得复盘的构建后运行它，在同一 session 中或指向该 session 的日志。顺滑的构建可跳过。

* [code-review](https://aihero.dev/skills-code-review) 是 `retro` 最常调整的审查代理：新的编码标准落在其 Standards 轴读取的地方。
* [writing-for-agents](https://aihero.dev/skills-writing-for-agents) 为 `retro` 提出的每个引导文件和技能设定写作风格，且 `retro` 在开始前会加载它。

[ask-matt](https://aihero.dev/skills-ask-matt) 会路由到整个技能集合，当你不确定当前情况需要哪个技能时。
