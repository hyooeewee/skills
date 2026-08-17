# 模型调用 vs 用户调用

本仓库中的每个 `SKILL.md` 都是一个技能。区分它们的唯一轴是**调用方式**——谁可以触发它：

* **用户调用** —— 仅**由人类键入其名称**即可触发。在前置元数据中设置 `disable-model-invocation: true`（Claude Code），并在 `agents/openai.yaml` 中设置 `policy.allow_implicit_invocation: false`（Codex）。`description` 是**面向人类的**：供浏览斜杠命令的人阅读的一行摘要。去掉触发器列表（“当用户说……时使用”）。
* **模型调用** —— 可由**模型或用户**触发。默认情况：省略 `disable-model-invocation` 和 `agents/openai.yaml` 中的 `policy` 块。`description` 是**面向模型的**，并保留丰富的触发措辞（“当用户想要……、提到……、要求……时使用”），以便自动调用触发。判断一个技能是否应保持模型调用的测试标准是：*模型能否自主地有效获取并使用它？*（复用是提取技能的原因，而不是测试标准。）

每个执行环境都以自己的方式将用户调用技能排除在模型的可及范围之外，因此除了人类之外，没有任何东西能触发它——其他技能也不能。用户调用技能可以调用模型调用技能，但它永远无法触达另一个用户调用技能。

每个技能还在其 `SKILL.md` 旁边带有一个 `agents/openai.yaml`。它包含 Codex UI 元数据——用于技能选择器的 `interface.display_name` 和 `interface.short_description`——以及对于用户调用技能，与 `disable-model-invocation` 配对的 `policy.allow_implicit_invocation: false`。保持两者同步：一个技能要么在两个执行环境中都是用户调用的，要么都不是。

将各个 `README.md` 以及顶层 `README.md` 中的条目分组为**用户调用**和**模型调用**。

## 它们之间的依赖关系

依赖关系被表示为调用具名技能的**显式指令**（`Call the Skill tool with "grilling"`），而不是深层的 `../other-skill/FILE.md` 交叉引用，也不是留给模型自行解读的裸 `/skill` 风格提及。指名道姓地提到工具才是让它触发的关键：大多数执行环境将技能调用暴露为模型可调用的工具，明确写出这一点比在散文中丢一个 `/name` 并指望它被当作命令来解读具有更高的命中率。去掉开头的 `/` 也使其保持执行环境中立，而不是相反——单独一个技能名称不会携带它属于哪个执行环境触发语法的假设。共享的参考文档位于拥有它们的技能内部；其他技能通过调用 Skill 工具来获取该材料，而不是跨文件夹链接。

这涉及**操作性**指令——技能自身的步骤告诉智能体立即去运行另一个技能。仅仅是列举技能供人类选择的路由说明（`ask-matt`、桶 `README.md`）并没有触发任何东西，因此它们将 `/skill` 风格名称保留为普通标签。

Skill 工具每次调用接受一个技能。需要两个技能的步骤就是两次调用，而不是一次调用带两个名称——要明确说明（`Call the Skill tool twice, for "grilling" and "domain-modeling"`），而不是“用 X 和 Y 调用它”，那会被理解为一次调用同时接受两者。

这整个约定仅在所命名的技能是**模型调用**时才成立。用户调用技能永远无法以这种方式被触达，就此打住——根据上述不变式，其他任何技能都不能调用它，包括向 Skill 工具指名它。当某一步骤的前提条件是用户调用技能（例如 `setup-matt-pocock-skills`）时，应将其表述为供人类执行的指令——“告诉用户运行 `/setup-matt-pocock-skills`”——绝不能作为 Skill 工具调用。

## 被动 vs 主动领域工作

仅仅*阅读* `CONTEXT.md` 以获取词汇只是一个一行式的散文指示，而不是 `domain-modeling` 技能。只有主动的构建/打磨纪律（挑战术语、边缘场景、编写 ADR、内联更新 `CONTEXT.md`）才是 `domain-modeling`。
