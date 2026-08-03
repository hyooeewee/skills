Quickstart:

```bash
npx skills add mattpocock/skills --skill=diagnosing-bugs
```

```bash
npx skills update diagnosing-bugs
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/diagnosing-bugs)

## 功能说明

`diagnosing-bugs` 为疑难 Bug 和性能回归运行一个严谨的诊断循环——构建复现步骤、最小化复现步骤、对假设进行排序、添加 instrumentation，然后通过回归测试来修复。

在你拥有**紧密反馈循环**之前，它会拒绝进行假设——即一个已经在*这个* Bug 上变红的可运行命令。在那个命令存在之前通过阅读代码来构建理论，正是该技能试图防止的失败。没有能够变红的循环，就没有诊断。

## 何时使用

输入 `/diagnosing-bugs`，或者当任务符合条件时，智能体（agent）会自动调用它——它在遇到“诊断” / “修复这个 Bug”时触发，或者当你报告某样东西出现损坏、抛出异常、失败或变慢时触发。

针对疑难问题使用它：那些第一眼看上去就难以攻克的 Bug、间歇性不稳定、在两个已知良好状态之间悄悄出现的回归。如果想快速验证设计问题而不是去追踪缺陷，请改用 [prototype](https://aihero.dev/skills-prototype)。

## 紧密的循环即技能

一旦你有了信号，其他所有事情——二分法、假设检验、instrumentation——都只是机械性的操作。因此，该技能在第一阶段投入了不成比例的精力：构建一个通过/失败命令，驱动实际的 Bug 代码路径并断言用户的精确症状，然后**收紧**它，直到它变得快速、确定且能被智能体运行。30 秒的不稳定循环几乎比没有好不到哪里去；2 秒的确定循环则是调试的超级能力。

它为你提供了一系列构建该循环的方法——失败的测试、curl 脚本、CLI 差异、无头浏览器、重放追踪、一次性测试套件、模糊测试循环、`git bisect run`、差异运行——并且，仅在万不得已时，使用人工参与的 bash 脚本。对于非确定性的 Bug，目标不是完美的复现，而是**更高的复现率**：循环触发器、并行化、增加压力，直到不稳定现象变得可调试。

## 判断是否生效

* 它在形成理论*之前*构建并运行复现命令——并粘贴调用过程及其红色的输出结果。
* 该循环断言的是你实际报告的症状，而不是附近的失败。
* 假设以一个排序的可证伪列表的形式呈现给你，在测试任何假设之前。
* 调试 instrumentation 被打上标签（`[DEBUG-...]`），并在它宣布完成之前被 grep 过滤掉。

## 在系统中的位置

`diagnosing-bugs` 是一个随时可用的独立技能——一旦出现故障，你立刻进入它；一旦修复和回归测试完成，你立刻退出。当真正的发现是找不到合适的位置来锁定 Bug 时，它的事后分析会移交给出 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture)——代码才是问题所在，而不是 Bug。当你不确定哪个技能适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
