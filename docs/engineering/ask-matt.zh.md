Quickstart:

```bash
npx skills add mattpocock/skills --skill=ask-matt
```

```bash
npx skills update ask-matt
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/ask-matt)

## 功能说明

`ask-matt` 是此仓库中技能的**路由器**。你描述你所处的状况；它告诉你哪个技能或流程最适合，以及按什么顺序运行它们。

它**本身不执行任何工作**。它不会审查、编写规范或修复任何问题——它只是提供方向。它最主要是为**用户调用的技能**而存在：没有任何东西会为你触发它们，所以*你*必须记住它们的存在，而 `ask-matt` 是你卸载记忆的地方。它还指向模型按名称调用的技能——`/tdd`、`/diagnosing-bugs`、`/prototype`、`/code-review`，以及两个词汇引用，`/domain-modeling` 和 `/codebase-design`。它回答“哪一个，以及何时”，然后将你移交给实际执行工作的技能。

## 何时使用

你通过输入 `/ask-matt` 来调用它——代理不会主动调用它。

当你不确定某个情况需要哪种技能或流程时，就使用它：你有一个想法但不知道从何开始，有一堆错误报告且不知道它们是否属于 `/triage`，或者有两个看起来可以互换的技能而你无法区分它们。如果你已经知道想要的技能，就跳过路由器直接调用它。

## 流程，而不仅仅是技能

`ask-matt` 给你用来思考的**流程**——一条*穿过*技能的路径，而不是单个技能。大多数工作沿着一条**主流程**运行（想法 -> 发布：审查 -> 规范 -> 工单 -> 实现 -> 回顾），两条**上坡道**汇入其中（用于处理传入错误和请求的分诊车道；用于生成想法的代码库健康车道），其余的一切都是你单独调用的**独立**技能。问一个问题，你就会被放置在正确的流程上，在正确的步骤——而不仅仅是一个工具。

## 在系统中的位置

`ask-matt` 是**路由器**——一个覆盖整个集合的独立地图。它是每个其他文档页面链接回的节点，称为 [ask-matt](https://aihero.dev/skills-ask-matt)，所以它从不处于*链*中；它指向*进入*每条链。从这里，你通常会在 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)（主流程的头部）或 [triage](https://aihero.dev/skills-triage)（你未创建的工作的上坡道）着陆。即使路由器自己的图片已经过时，它的 [Source](https://github.com/mattpocock/skills/tree/main/skills/engineering/ask-matt) 也是记录地图。
