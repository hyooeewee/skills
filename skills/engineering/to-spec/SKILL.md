---
name: to-spec
description: "Turn the current conversation into a spec and publish it to the
  project issue tracker: no interview, just synthesis of what you've already
  discussed."
disable-model-invocation: true

---

This skill takes the current conversation context and codebase understanding and produces a spec. Do NOT interview the user; just synthesize what you already know.

你的问题跟踪器和分流标签词汇表应该已经提供给你。如果没有，请告诉用户运行 `/setup-matt-pocock-skills`。

## 流程

1. 如果尚未完成，请探索代码仓库以了解代码库的当前状态。在整个规格说明中始终使用项目的领域术语表词汇，并尊重你所涉及区域内的任何 ADR。

2. 描绘出你将在哪些接缝（seams）处测试该功能。应优先使用现有接缝，而不是新建接缝。使用尽可能高的接缝。如果需要新的接缝，请在尽可能高的位置提出。整个代码库中的接缝越少越好——理想数量是一个。

与用户确认这些接缝是否符合他们的预期。

3. 使用下面的模板编写规格说明，然后将其发布到项目议题跟踪器。应用 `ready-for-agent` 分流标签——无需额外分流。

<spec-template>

## 问题陈述

从用户的角度出发，用户当前面临的问题。

## 解决方案

从用户的角度出发，针对该问题的解决方案。

## 用户故事

一个冗长的、带编号的用户故事列表。每个用户故事的格式应为：

1. 作为 <actor>，我想要 <feature>，以便 <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

这份用户故事列表应当极其详尽，覆盖该功能的各个方面。

## 实现决策

所做的实现决策列表。可以包括：

* 将构建/修改的模块
* 将被修改的这些模块的接口
* 来自开发者的技术澄清
* 架构决策
* Schema 变更
* API 契约
* 具体交互

不要包含具体的文件路径或代码片段。它们可能很快就会过时。

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.

## 测试决策

所做的测试决策列表。包括：

* 描述什么构成了一个好的测试（只测试外部行为，而不是实现细节）
* 哪些模块将被测试
* 测试的既有先例（即代码库中类似类型的测试）

## 范围外

对本规格说明而言不在范围内的内容的描述。

## 补充说明

关于该功能的任何进一步说明。

</spec-template>
