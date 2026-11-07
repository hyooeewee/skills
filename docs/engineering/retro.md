## 它做什么

`retro` 会回顾一个编码 [session](https://www.aihero.dev/ai-coding-dictionary/session)，并向 agent 的 **[environment](https://www.aihero.dev/ai-coding-dictionary/environment)** 提出改进建议，以便下一次运行更顺利。它读取 session 自身的记录（默认为当前 session，或你在 session 日志中指向的某个 session）。它找出 agent 吃力的时刻，并给出一份候选修复列表，按严重程度从高到低排序。

它改变的是 environment，而不是代码。拿 agent 交付的 bug、它需要二十次 [tool calls](https://www.aihero.dev/ai-coding-dictionary/tool-call) 才找到的文件、或 reviewer 漏掉的规则来说，`retro` 不直接修复它们。它追问：repo 里什么东西让它们发生了？并提出能防止下次再发的 check、pointer 或 standard。它也仅仅是提出建议。除非你选中某个候选项，否则什么都不会变。

## 何时使用它

输入 `/retro` 来调用它，agent 不会自行使用它。

在比预期更艰难的 session 结束时使用它。例如：agent 搜索太久、犯了工具本可捕获的错误、或需要却获取不到的信息。顺利的 session 教不出什么东西，发现来自困难的那些。如果你想对 session 产出的代码做裁决，请改用 [code-review](https://aihero.dev/skills-code-review)。

## 发现落在哪里

每个候选项属于某一类别，类别决定修复去向：

| session 中出了什么问题            | 用什么修复                                                                                                                        |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| agent 花很长时间才找到某个文件或事实      | 一个 **navigation pointer**从它已读的文件出发                                                                                           |
| agent 犯了工具本可捕获的错误          | 一个 **[automated check](https://www.aihero.dev/ai-coding-dictionary/automated-check)**：lint 规则、类型检查、测试、pre-commit hook、CI job |
| reviewer 漏判了一个需要判断力的错误     | 在 \` `CODING_STANDARDS.md`\` 中为 reviewer agent 增加一条规则                                                                        |
| `AGENTS.md`或 `CLAUDE.md`很大 | 把其中的 steering 迁移到 standards 或 checks 中                                                                                       |
| 某次 tool call 代价高但返回价值低     | 精简该工具，或替换它                                                                                                                   |
| steering 文件全是不起作用的行        | 删除 **no-ops**                                                                                                                |
| agent 需要却够不着的信息            | 放宽它的访问：把 dev server 日志 tee 到文件、给服务只读权限等                                                                                      |

核心理念是：standards 属于 **reviewer**，而非 implementer。实现端 agent 承受最大的上下文压力，因为它要探索、写代码、调试失败。reviewer 只拿到一个 diff。所以新规则应放在有空间施行的地方——review。它永不进 [AGENTS.md](https://www.aihero.dev/ai-coding-dictionary/agents-md)，因为那会把内容加载进每个 session 的 [context window](https://www.aihero.dev/ai-coding-dictionary/context-window)，无论相关与否。

`retro` 在写规则前先给违规分类。**机械性** 违规（禁用 API、import 形状、文件位置规则）得 deterministic check，因为 check 会失败，而 standards 文件里的一句话不会。只有真正需要判断力、linter 无法强制的场景才变成文字。若 repo 完全没有护栏（无 pre-commit hook，且无跑 lint/typecheck/test 的 CI job），`retro` 会把这本身作为一个 finding 报出。

## 常见问题

**它会自己写 lint 规则，还是等确认？能不能接在每个 session 后自动跑？**

它等确认。`retro` 只提议，不直接改动、不自动装 hook。这是刻意设计：有用户因「被自动 hook 挡住合理改动而受伤」而要求如此。决定什么值得长期检查需要判断力，所以该 skill 保持 [human-in-the-loop](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop) 且由用户调用。有人确实每次实现后都跑。但顺利的 session 教不出东西，每次都跑多半产出没人需要的规则。没有 dry-run 模式。提议的 check 也是代码，合并前先在 repo 里试跑。

**这会让 lint 规则无限堆积吗？它会建议删规则吗？**

部分会，这是它最弱的地方。它能删文字：steering 文件里的 no-ops、应归入 standards 或 check 的 `AGENTS.md`/`CLAUDE.md` steering。文件大时它会标记这些为删除候选。它只针对读到的这一个 session 判断，所以把每项当作「删除测试」的候选而非定论。它不审计上个月自己提议的 lint 规则、hook、CI job。它只看一个 session，无法告诉你某规则现在触发太频繁、或背后的 bug 已消失。你仍需自己修剪 check。规则在好代码上频繁触发，就是该删的信号。

**它会不会为了填满分类而编造通用建议？**

这是对它最强烈的批评。有用户发现：「工作完成后，AI 倾向于忘掉 session 中期的挣扎，编造通用建议以填满 retro 分类。」每个候选必须来自 session 自身记录，所以建议针对该 session。这有利有弊：极少编造无关内容，但可能过度重视这一个 session 碰到的事。丢弃任何追溯不到具体时刻的候选。严重度排序也视作草稿：安静而昂贵的错误可能排在吵闹但廉价的错误之下。

**我的 session 很长。现在跑，还是重开？**

默认复盘当前 session。这是最佳情况，因为挣扎还在 context window 里。若 session 已滑出 [smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone)，[clear](https://www.aihero.dev/ai-coding-dictionary/clearing) 掉，再用新的 `/retro` 指向 session logs 里的上一个 session。

**agent 反复犯同一个错。我该往 \` `CLAUDE.md`？**

通常别加，`retro` 对此反对最坚决。`CLAUDE.md` 里的一行会加载进每个 session，稀释文件里其他内容的权重，且随代码变更而过时。若是机械性错误，修复是会失败的 check；若是判断力问题，进 reviewer 读的 coding standards。`AGENTS.md` 和 `CLAUDE.md` 只放 navigation pointer 为主。同理，`retro` 不是 [memory system](https://www.aihero.dev/ai-coding-dictionary/memory-system)。它不存发生过什么。它改 environment，让错误无法再发生。

**我的配置提到 \` `CODING_STANDARDS.md`\` 但我没有这个文件。它从哪来？**

没有 skill 自带该文件。session 首次为 reviewer 发现判断力类规则时，`retro` 会提议创建它。接受后，[code-review](https://aihero.dev/skills-code-review) 会读取它。你已有的其它 standards 文档（如 `CONTRIBUTING.md`）同理。

**它和 \` `improve-codebase-architecture`？**

输入不同。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 只需代码，找代码结构上的改进。`retro` 需要 session 历史，改进 agent 工作的 environment，而非代码。两者都用；互不替代。

## 如果它起作用了

* 每个候选都能追溯到 session 中的具体时刻，而非通用最佳实践。
* 重复错误变成会失败的 check，且你的 `AGENTS.md` 随时间变短而非变长。
* 若 check 已存在但未连上，finding 是去连它，而非提议造新的。
* 同类任务的下一个 session 能更快找到所需。

## 它在系统中的位置

`retro` 是主链的最后一步，用来复盘链路跑得如何：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review → retro
```

在值得复盘的构建后运行，在同一 session 中或指向该 session 的日志。顺利的构建可跳过。

* [code-review](https://aihero.dev/skills-code-review) 是 `retro` 最常调优的审查代理。新的编码标准放在其 Standards 轴读取的位置。
* [writing-for-agents](https://aihero.dev/skills-writing-for-agents) 为 `retro` 提出的每个引导文件和技能设定写作风格，且 `retro` 在开始前会加载它。

[ask-matt](https://aihero.dev/skills-ask-matt) 在你不确定需要哪个技能时，跨整个技能集进行路由。
