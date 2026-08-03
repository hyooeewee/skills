Quickstart:

```bash
npx skills add mattpocock/skills --skill=code-review
```

```bash
npx skills update code-review
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/code-review)

## 功能说明

`code-review` 审查 `HEAD` 和你提供的固定点之间的差异——提交、分支、标签或合并基点——沿着两个独立的轴：**标准**（代码是否遵循此仓库的文档化约定？）和**规范**（它是否实现了原始问题或规范所要求的内容？）。它将每个轴作为独立的并行子代理运行，并并排报告它们。它从不合并或重新排序两组发现结果——保持它们分离是整个重点，因为变更可能在一个轴上通过而在另一个轴上失败，而单一的混合裁决会让一个掩盖另一个。

## 何时使用

输入 `/code-review`，或者当你要求审查分支、PR、进行中的更改或任何“自 X 以来”的内容时，代理会自动使用它。

当有差异需要对照已知良好的点进行判断，并且你希望两个问题——*构建正确吗？* 和 *是正确的事吗？*——被独立回答时，使用它。它在构建循环结束时运行；对于实际以测试优先的方式编写代码，请使用 [tdd](https://aihero.dev/skills-tdd)，而对于将整个规范构建成代码，请使用 [implement](https://aihero.dev/skills-implement)，它在提交前会运行自己的 `/code-review` 检查。

## 前置条件

**规范**轴需要一个地方来查找原始规范——提交消息中的问题引用、你传入的路径，或 `docs/`/`specs/` 下的规范。该问题跟踪器配置来自 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills)；如果没有规范，规范轴会直接跳过并说明这一点。**标准**轴不需要任何设置——即使在没有文档化约定的仓库中，它也始终携带内置的 Fowler 代码异味基线。

## 两个轴，永不合并

核心思想是**两个轴**。**标准**轴询问差异是否符合此仓库的代码编写方式——它的 `CODING_STANDARDS.md` 或 `CONTRIBUTING.md`，加上固定的约 12 个 Fowler 代码异味（Mysterious Name, Duplicated Code, Feature Envy, Data Clumps, …）。两条规则保持基线安全：文档化的仓库标准总是覆盖它，而且每个代码异味都是主观判断，而不是硬性违规。**规范**轴询问正交的问题——代码是否做了问题或规范实际要求的事情，而没有遗漏需求或偷偷引入范围蔓延？

它们作为并行子代理运行，因此不会污染彼此的上下文，最终报告在单独的 `## 标准` 和 `## 规范` 标题下展示它们，并附带每个轴的摘要。轴之间故意没有唯一的获胜者。

## 判断是否生效

* 它首先固定并确认固定点（`git rev-parse`），在错误的引用或空差异时快速失败，而不是在子代理内部。
* 标准和规范发现结果出现在两个不同的块中，每个都引用其来源——一个是仓库标准或基线代码异味，另一个是引用的规范行。
* 当找不到规范时，规范轴报告“无可用规范”，而不是编造需求。

## 在系统中的位置

`code-review` 是主构建链末尾的审查步骤：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

它最近的邻居是 [implement](https://aihero.dev/skills-implement)，它驱动构建并在提交前将其作为自己的审查步骤调用；在上游，它检查的规范是由 [to-spec](https://aihero.dev/skills-to-spec) 和 [to-tickets](https://aihero.dev/skills-to-tickets) 生成的。当你不确定哪个技能或流程合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
