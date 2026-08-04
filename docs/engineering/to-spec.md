Quickstart:

```bash
npx skills add mattpocock/skills --skill=to-spec
```

```bash
npx skills update to-spec
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/to-spec)

## 功能说明

`to-spec` 将当前的对话和您对代码库的理解转换成规范（您可能知道这个文档被称为 PRD），然后将其发布到您的问题追踪器中。

它不会再次询问您。当您使用它时，对齐工作已经完成 —— `to-spec` 综合已有的已知信息，而不是提出新一轮的问题。

## 何时使用

您通过输入 `/to-spec` 来调用它 —— 代理不会自动调用它。

当变更讨论完毕且领域语言确定后，并且您希望在编写任何代码之前写下这种共同理解时，就使用它。如果您还没有对齐，请先“grill”——为此，请使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)。要将完成的规范拆分为工单，请使用 [to-tickets](https://aihero.dev/skills-to-tickets)。

## 前置条件

`to-spec` 会发布到您的问题追踪器，因此 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 必须先为此仓库配置追踪器和分类标签。它会自己应用 `ready-for-agent` 标签 —— 不需要单独的分类流程。

## 规范包含的内容

* **问题陈述** — 项目自己的术语中，什么是出问题或缺失的，以及为什么值得解决。
* **解决方案** — 高层概览的修复方案，在任何具体实现细节之前。
* **用户故事** — 变更必须支持的具体行为的广泛、编号列表，每一个都可以独立验证。
* **实施决策** — 对话中已经确定的选项，以免日后重新讨论。
* **测试决策** — 功能将被测试的接口，以及“完成”的样子。
* **超出范围的项目** — 此变更刻意不覆盖的内容，以保持工单范围可控。
* **进一步说明** — 其他值得保留但不适合上述章节的内容。

## 深度模块

在编写规范之前，`to-spec` 会识别出将被测试的**接口**，并寻找**深度模块**的机会 —— 即隐藏在小型稳定接口背后的大量功能。它更倾向于现有的接口而非新的接口，以及尽可能高的接口，理想情况下在整个变更中只有一个接口。

这对代理开发很重要：一个好的接口给了测试一个稳定的目标，因此底下的代码可以改变而不影响测试。

## 判断是否生效

* 它开始编写规范而不是向您提出新一轮的问题。
* 它在编写前与您检查接口，并尽可能少地提出建议。
* 规范以您的项目的领域语言返回，而不是通用的样板文本。

## 在系统中的位置

`to-spec` 是主构建链中的一个步骤：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

在计划和领域语言解决之后，并将工作拆分为实施工单之前，使用它。它的关键相邻技能是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，它能精炼上下文使规范精确，以及 [to-tickets](https://aihero.dev/skills-to-tickets)，它将规范转换为一组工单供 [implement](https://aihero.dev/skills-implement) 构建。当您不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为您指引。
