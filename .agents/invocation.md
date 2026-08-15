# 模型调用 vs 用户调用

本仓库中的每个 `SKILL.md` 都是一个技能。区分它们的唯一轴是**调用方式**——谁可以触发它：

* **用户调用** —— 仅**由人类键入其名称**即可触发。在前置元数据中设置 `disable-model-invocation: true`（Claude Code），并在 `agents/openai.yaml` 中设置 `policy.allow_implicit_invocation: false`（Codex）。`description` 是**面向人类的**：供浏览斜杠命令的人阅读的一行摘要。去掉触发器列表（“当用户说……时使用”）。
* **模型调用** —— 可由**模型或用户**触发。默认情况：省略 `disable-model-invocation` 和 `agents/openai.yaml` 中的 `policy` 块。`description` 是**面向模型的**，并保留丰富的触发措辞（“当用户想要……、提到……、要求……时使用”），以便自动调用触发。判断一个技能是否应保持模型调用的测试标准是：*模型能否自主地有效获取并使用它？*（复用是提取技能的原因，而不是测试标准。）

每个执行环境都以自己的方式将用户调用技能排除在模型的可及范围之外，因此除了人类之外，没有任何东西能触发它——其他技能也不能。用户调用技能可以调用模型调用技能，但它永远无法触达另一个用户调用技能。

每个技能还在其 `SKILL.md` 旁边带有一个 `agents/openai.yaml`。它包含 Codex UI 元数据——用于技能选择器的 `interface.display_name` 和 `interface.short_description`——以及对于用户调用技能，与 `disable-model-invocation` 配对的 `policy.allow_implicit_invocation: false`。保持两者同步：一个技能要么在两个执行环境中都是用户调用的，要么都不是。

将各个 `README.md` 以及顶层 `README.md` 中的条目分组为**用户调用**和**模型调用**。

## 它们之间的依赖关系

依赖关系以 **`/skill` 风格的散文式调用**（“运行 `/grilling` 技能”）来表达，而不是深层的 `../other-skill/FILE.md` 交叉引用。共享参考文档位于拥有它们的技能内部；其他技能通过调用该技能来获取这些材料，而不是跨文件夹链接。

## 被动 vs 主动领域工作

仅仅*阅读* `CONTEXT.md` 以获取词汇只是一个一行式的散文指示，而不是 `domain-modeling` 技能。只有主动的构建/打磨纪律（挑战术语、边缘场景、编写 ADR、内联更新 `CONTEXT.md`）才是 `domain-modeling`。
