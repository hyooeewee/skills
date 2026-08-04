Quickstart:

```bash
npx skills add mattpocock/skills --skill=resolving-merge-conflicts
```

```bash
npx skills update resolving-merge-conflicts
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/resolving-merge-conflicts)

## 功能说明

`resolving-merge-conflicts` 逐个处理进行中的 git 合并或变基冲突，逐个补丁（hunk）进行，并完成操作——已解决、已检查并已提交。

它通过**意图**而非文本解决。在处理补丁之前，它会追溯每一方回溯到其**主要来源**——提交信息、PR、原始 issue——以了解更改的原因，然后在兼容的情况下保留两种意图。它从不编造新行为来掩盖冲突，也从不使用 `--abort`：合并总是会被完成。

## 何时使用

输入 `/resolving-merge-conflicts`，或者当任务符合条件时，代理会自动调用它。

当你正在进行合并或变基且 git 停在它无法自行解决的冲突上时，使用此技能。这是为了解决你面前的冲突——而不是为了规划合并或调试之后出现的错误行为。如果合并已完成但某些东西现在失败且原因不明，请改用 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)。

## 按意图解决

冲突中的陷阱在于将其视为文本问题——选择“ours”或“theirs”来让标记消失。此技能将其视为**意图**问题。补丁的每一侧之所以存在，是因为有人想要某样东西；解决方式必须在可能的地方尊重这两种需求，而在它们确实不兼容的地方，选择与合并既定目标相符的一方，并大声记录下权衡取舍。

这就是为什么主要来源很重要。你无法保留你没读过的意图，所以工作始于历史——提交、PR、工单——而不是在差异（diff）中。

## 判断是否生效

* 每个已解决的补丁都保留了两边的行为，或者无法保留时指出了权衡取舍。
* 没有出现任何不在任一分支上的新行为。
* 在提交之前，会找到并运行项目的自身检查——类型检查、测试、格式化——并且全部通过。
* 合并或变基一直进行到完成提交，从未被中止。

## 在系统中的位置

一个随时可用的独立技能：你在合并或变基停滞的瞬间调用它，它会将一个干净、已提交的树交还给你。它的自然邻接技能是 [diagnosing-bugs](https://aihero.dev/skills-diagnosing-bugs)，因为一个解决干净但之后表现异常的合并是一个诊断问题，而不是冲突问题。当你不确定哪个技能适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你路由。
