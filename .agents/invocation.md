# 模型调用 vs 用户调用

此仓库中的每个 `SKILL.md` 都是一个技能。将它们区分开来的唯一轴心是 **调用方式**，即谁能触及它：

* **用户调用**：只能通过人工输入其名称来调用。在前置元数据中设置 `disable-model-invocation: true`（Claude Code）和在 `agents/openai.yaml` 中设置 `policy.allow_implicit_invocation: false`（Codex）。`description` 是**面向人类的**：由浏览斜杠命令的人员阅读的单行摘要。删除触发列表（“当用户说……”）。
* **模型调用**：可被**模型或用户**调用。默认情况：省略 `agents/openai.yaml` 中的 `disable-model-invocation` 和 `policy` 块。`description` 是**面向模型的**并保留丰富的触发短语（“当用户想要……、提到……、要求……时使用”），以便自动调用触发。判断技能应保持模型调用的测试：*模型是否有用且自主地获取此技能？*（重用是提取技能的原因，而非测试标准。）

每个工具链都以自己的方式将用户调用的技能排除在模型的可达范围之外，因此除了人类之外没有任何东西能触发它：没有其他技能可以。用户调用的技能可以调用模型调用的技能，但它永远无法触及另一个用户调用的技能。

每个技能在其 `SKILL.md` 旁边也携带一个 `agents/openai.yaml`。它包含 Codex UI 元数据：技能选择器的 `interface.display_name` 和 `interface.short_description`，以及对于用户调用的技能，与 `disable-model-invocation` 配对的 `policy.allow_implicit_invocation: false`。保持两者同步：一个技能在两个框架中都是用户调用的，或者都不是。

将各个 `README.md` 以及顶层 `README.md` 中的条目分组为**用户调用**和**模型调用**。

## 它们之间的依赖关系

依赖关系被表达为明确的指令：**调用 Skill 工具** 并指定技能名称（`Call the Skill tool with "grilling"`），而不是深层 `../other-skill/FILE.md` 跨引用，也不是留给模型解释的裸 `/skill` 风格提及。命名工具是触发它的关键：大多数框架将技能调用暴露为模型调用的工具，明确说明这一点比在文中随意插入 `/name` 并希望被读作命令具有更高的命中率。去掉前导 `/` 也使此框架保持中立：单独的技能名称不假设其属于哪个框架的触发语法。共享参考文档位于拥有它们的技能内部；其他技能通过调用 Skill 工具并传入该材料来访问它，而不是通过跨文件夹链接。

这关于**操作**指令：技能自身的步骤指示代理现在去运行另一个技能。仅列出供人类选择的技能的路由器文本（如 `ask-matt` 或对 `README.md` 进行分桶）并不调用任何东西，因此它将 `/skill` 风格的名称保留为普通标签。

Skill 工具每次调用接受一个技能。需要两个技能的步骤需要两次调用，而不是一次带有两个名称的调用：明确说明（`Call the Skill tool twice, for "grilling" and "domain-modeling"`），而不是“用 X 和 Y 调用它”，后者读起来像是单次调用接受了两者。

整个惯例仅在命名技能为**模型调用**时成立。用户调用的技能绝无可能以这种方式被触及：根据上述不变量，没有其他技能可以调用它，包括通过将其名称传递给 Skill 工具。当步骤的前提条件是用户调用的技能（例如 `setup-matt-pocock-skills`）时，应将其表述为对人类采取行动的指令：“告诉用户运行 `/setup-matt-pocock-skills`”，而不是作为 Skill 工具调用。

## 被动 vs 主动领域工作

仅仅*阅读* `CONTEXT.md` 以获取词汇只是一个一行式的散文指示，而不是 `domain-modeling` 技能。只有主动的构建/打磨纪律（挑战术语、边缘场景、编写 ADR、内联更新 `CONTEXT.md`）才是 `domain-modeling`。
