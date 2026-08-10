# 模型调用 vs 用户调用

本仓库中的每个 `SKILL.md` 都是一个技能。区分它们的唯一维度是 **调用** —— 谁可以调用它：

* **用户调用** —— 仅能通过**人类输入其名称**来调用。在 frontmatter（Claude Code）中设置 `disable-model-invocation: true`，在 `agents/openai.yaml`（Codex）中设置 `policy.allow_implicit_invocation: false`。`description` 是 **面向人类的**：由浏览斜杠命令的人阅读的单行摘要。去除触发列表（“当用户说……”）。
* **模型调用** —— 可由 **模型或用户** 调用。默认情况：省略 `disable-model-invocation` 和 `agents/openai.yaml` 中的 `policy` 块。`description` 是 **面向模型的**，并保留丰富的触发短语（“当用户想要……、提及……、要求……”），以触发自动调用。判断技能应保持模型调用的标准是：*模型能否自主有效地调用它？*（复用是提取技能的原因，而非测试标准。）

每个框架都以自己的方式将用户调用的技能从模型的调用范围内排除，因此只有人类可以触发它 —— 其他技能都不行。用户调用的技能可以调用模型调用的技能，但它永远无法到达另一个用户调用的技能。

每个技能旁边还有一个 `agents/openai.yaml`。它包含 Codex UI 元数据 —— 用于技能选择器的 `interface.display_name` 和 `interface.short_description` —— 并且，对于用户调用的技能，它包含与 `disable-model-invocation` 配对的 `policy.allow_implicit_invocation: false`。保持两者同步：技能在两个框架中都应是用户调用的，或者都不是。

将 `README.md` 和顶级 `README.md` 中的条目归类到 **用户调用** 和 **模型调用** 中。

## 它们之间的依赖关系

依赖关系表示为 **`/skill` 风格的散文式调用**（“运行 `/grilling` 技能”），而不是深层的 `../other-skill/FILE.md` 跨引用。共享参考文档位于拥有它们的技能内部；其他技能通过调用该技能来获取这些材料，而不是通过跨文件夹链接。

## 被动与主动领域工作

仅为了词汇 *阅读* `CONTEXT.md` 是一行散文式的指针，不是 `domain-modeling` 技能。只有主动的构建/打磨规范（挑战术语、边缘情况场景、编写 ADR、内联更新 `CONTEXT.md`）才是 `domain-modeling`。
