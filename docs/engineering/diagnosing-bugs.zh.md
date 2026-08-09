## 功能说明

`diagnosing-bugs` runs a six-phase diagnosis on a hard bug or a performance regression: build a repro, minimise it, rank hypotheses, instrument, fix with a regression test, clean up.

It will not let the agent form a theory until a **tight** feedback loop exists — one named command, already run once, that goes red on *this* bug and green when it is fixed. The default behaviour of a coding agent handed a bug report is to read code and guess; this skill blocks that. If no red-capable command exists, there is no Phase 2. That single gate is what the skill is for. Everything after it — bisection, hypothesis-testing, instrumentation — is mechanical once the signal exists.

## 何时使用

类型`/diagnosing-bugs`, or the agent reaches for it on its own when a task fits — it is model-invoked, and fires on "diagnose" / "debug this" or on a report that something is broken, throwing, failing, or slow.

在棘手的问题上使用它：一种经不起初步审视的错误，间歇性故障，或潜入两个已知良好状态之间的回归。它设计得很重，对于你希望在一句话中得到答案的问题来说，是错误的工具。

| 你的情况                     | 去往哪里                                                                            |
| ------------------------ | ------------------------------------------------------------------------------- |
| 你可以描述为症状的特定缺陷            | 此技能                                                                             |
| 一个已知的“之前和之后”对比的慢速端点或时间回归 | 此技能 —— 它有一个性能分支（测量基线，然后二分查找）                                                    |
| “此代码库中的瓶颈在哪里？” — 没有特定症状  | 不是此技能。它诊断一个已知的失败，它不进行审计                                                         |
| 其他人提供的原始错误报告，尚未确认或记录     | [优先级评估](https://aihero.dev/skills-triage)首先                                     |
| 用于回答设计问题的临时代码，而非追查缺陷     | [prototype](https://aihero.dev/skills-prototype)                                |
| 构建预定的行为测试优先              | [tdd](https://aihero.dev/skills-tdd)                                            |
| 没有好的接口/接缝来锁定该错误          | [改进代码库架构](https://aihero.dev/skills-improve-codebase-architecture)— 此技能会将其转交到那里 |

## 紧密的循环即技能

第 1 阶段会得到不成比例的精力，因为它是唯一困难的阶段。该技能提供了一组构建循环的方法，大致按偏好顺序排列：

1. 在任何通往错误的接缝处放置一个失败的测试。
2. 对运行中的开发服务器的 curl 或 HTTP 脚本。
3. 带有 fixture 输入的 CLI 调用，与已知良好的快照进行 diff 对比。
4. 在 DOM、控制台或网络上断言的 headless 浏览器脚本。
5. 重放的捕获 — 保存的请求、有效载荷或事件日志，独立地通过代码路径运行。
6. 一次性测试工具：系统的最小子集，一次函数调用。
7. 属性或模糊测试循环，用于“有时输出错误”的情况。
8. 你可以交给 `git bisect run` 的二分查找测试工具。
9. 差异循环 — 相同输入，旧版本对比新版本。
10. 一个[人工参与循环](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop) bash 脚本，作为最后手段。该技能为此提供 `scripts/hitl-loop.template.sh`：代理运行脚本，你在终端中跟随提示，你的回答作为可解析的输出发回。

*一个*循环不是目标。**紧密**才是：快速（秒级），确定性（每次运行裁决相同），精准（断言你的确切症状，而不是“没有崩溃”），且无需人工干预即可由代理运行。30 秒的不稳定循环几乎比没有好不到哪里去。对于只在某些时候出现的错误，目标不是干净的复现，而是**更高的复现率** — 循环触发器，并行化，增加压力，注入休眠，直到不稳定率足够高以便调试。

当它确实无法构建一个时，被指示停止并说明，列出它尝试过的方法，并要求你提供[环境](https://www.aihero.dev/ai-coding-dictionary/environment)访问权限、捕获的工件，或添加临时工具的许可。无论如何，它都不应继续进行假设。

## 阶段之间的门

阶段是门，不是清单。每个门都拒绝打开，直到某件特定的事情发生。

| 门        | 必须为真                                                       |
| -------- | ---------------------------------------------------------- |
| 进入第 2 阶段 | 一个已命名且已运行并粘贴了其输出的命令，该命令在此错误上可以变红                           |
| 进入第 3 阶段 | 复现已被复现 *和&#x20;*&#x5E76;已最小化 — 所有剩余元素都是承载性的                |
| 进入第 4 阶段 | 存在 3-5 个已排序、可证伪的假设，每个假设陈述其预测，在测试任何假设之前展示给你                 |
| 进入第 5 阶段 | 探针映射到特定预测，一次一个变量，每个调试日志都标记为 `[DEBUG-a4f2]`风格，以便清理只需一次 grep |
| 完成       | 原始复现不再复现，工具已移除，且结果正确的假设被写入提交消息                             |

第 5 阶段有一个值得了解的逃生舱。回归测试是在修复之前编写的，但仅当存在**正确的接缝**时 — 即测试在调用点发生时执行真实错误模式的接缝。当唯一可用的接缝太浅时，该技能被告知说明这一点，而不是编写给出虚假信心的测试。这种缺失本身就是发现，它将事后分析路由到 `improve-codebase-architecture`。

## 常见问题

它在我想直接得到答案的快速问题上触发。
这是该技能报告最多的问题，而且确实存在。
特别是在 GPT-5.6-Sol 上，用户报告它在描述问题的简单文本上触发：“模型触发了相当正式的 diagnosing-bugs 技能。”
然后它继续构建复现场景 — 通常构建价值有限的模拟场景 — 然后才给我回复或建议。
这导致相当大的回复延迟。
四个不同的人在 issue #578 上报告了相同的情况。
可接受的修复方法是从较轻的方法开始，只在问题值得时才过渡到更重的方法，但该更改尚未落地。
该技能是根据 Claude Code 的调用行为进行校准的；具有较低激活阈值的[模型](https://www.aihero.dev/ai-coding-dictionary/model)会过度触发它。
在它毕业之前，实用的修复方法是说出你想要什么（“只回答这个问题，不要诊断”），或者在你的[harness](https://www.aihero.dev/ai-coding-dictionary/harness)中禁用对其的模型调用。

我可以将它指向一个代码库并询问性能问题在哪里吗？
不。它诊断一个你已经可以命名的失败。
它的性能分支用于有症状的回归 — 建立基线测量，然后二分查找，先测量后修复 — 而不是用于主动扫描。
用于主动版本的技能[已被提议并关闭](https://github.com/mattpocock/skills/issues/431)；目前没有针对它的技能。

它在编写修复之前会停下来问我吗？
不。只有第 3 阶段有人类检查点 — 排名列表在测试任何假设之前展示给你，如果你不在场，它会根据自己的排名继续。
工具和修复之间没有门，所以代理可以在你同意其根本原因之前开始编写代码。
Issue #124 要求设置那个门，并且仍然开放。如果你想要它，请在调用该技能时说明。

我已经在这个错误报告上运行了 `/triage`。这是在重复做同样的工作吗？
部分是，而且两个技能都不承认。
正如一位读者所说：“Triage 的第 3 步基本上是 diagnosing-bugs 第 1-2 阶段的浅层、有界实例，但两个文件都没有提到对方。”
Triage 进行有界的‘这实际上是一个错误，表面是什么’的检查；此技能做的是彻底的版本。
首先运行 triage 并不是浪费 — 它的验证通常给你第 1 阶段的大部分原始材料 — 但要期望在这里正确地重做它，并期望没有任何交叉引用来告诉你这一点。

它粘贴的复现输出会泄露机密吗？
可能会。该技能要求代理粘贴调用及其输出，并请求 HAR 文件、日志转储和核心转储等工件。
指令中没有对这些进行清理。
Issue #674 正好提出了这个问题 — 凭证、令牌、Cookie 和个人数据随同进入聊天、问题或 PR — 并提出了脱敏护栏。
它是开放的且未实现。暂时将脱敏视为你的工作，特别是在输出发布到任何公开位置之前。

**我的安全扫描器将此技能标记为高风险。** Snyk 对其进行了标记，但这是一个误报。这是集合中唯一一个附带运行说明和 curl 开发服务器指令的可执行 shell 脚本 (`hitl-loop.template.sh`) 的技能。打包的 `.sh` 加上运行指令加上出站 HTTP 足以触发静态扫描器。脚本本身大约有 30 行 `read -r -p` 提示，用于暂停以等待人工输入。扫描器评估的是能力面，而不是已知的漏洞利用方式。

**`/diagnose` 去哪了？** 在 v1.0.0 中重命名为 `/diagnosing-bugs`。旧名称不再存在。任何链接到 `/diagnose` 的内容——包装技能、保存的提示——都需要更新。

## 判断是否生效

* 它在提出单一理论之前会显示命令及其红色输出。如果理论先出现，则该技能未在运行。
* 它复现的失败是你报告的那个，而不是它在途中找到的附近的一个。
* 它在开始猜测之前会缩小复现步骤，并可以告诉你每个剩余部分为何至关重要。
* 在测试其中任何一个之前，你会看到一个 3-5 个假设的排序列表，每个假设都有一个你可以证伪的预测。
* 它添加的每个调试日志都带有像 `[DEBUG-a4f2]` 这样的标签，当它声明完成时，对该标签的 grep 返回为空。
* 提交信息或 PR 消息指明了哪个假设是正确的。
* 当它无法通过测试锁定漏洞时，它会明确说明，而不是写一个浅显的测试。

## 在系统中的位置

`diagnosing-bugs` 是一个随时可用的独立技能。当出现问题时，你会进入它；当修复和回归测试就绪时，你会退出；它不保存状态，也不需要预先设置。[ask-matt](https://aihero.dev/skills-ask-matt) 将 "Something's broken" 路由到这里。

两个相邻的技能很重要。[improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 在真正的发现是代码没有锁定漏洞的接口时接管 [交接](https://www.aihero.dev/ai-coding-dictionary/handoff)——建议是在修复就绪、拥有更多信息之后做出的。[triage](https://aihero.dev/skills-triage) 位于它上游，用于处理作为原始报告从其他人那里到达的漏洞，并执行前两个阶段较浅的版本。
