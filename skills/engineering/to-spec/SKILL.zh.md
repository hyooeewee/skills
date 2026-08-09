---
name: to-spec
description: 将当前对话转换为规范并发布到项目问题跟踪器 — 无需访谈，只需综合您已经讨论过的内容。
disable-model-invocation: true

---

This skill takes the current conversation context and codebase understanding and produces a spec. Do NOT interview the user — just synthesize what you already know.

问题跟踪器和分类标签词汇表应该已经提供给你了——如果没有，请运行 `/setup-matt-pocock-skills`。

## 流程

1. 探索仓库以了解代码库的当前状态（如果尚未完成）。在整个规范中使用项目的领域词汇表，并尊重您所涉及的区域中的任何 ADR。

2. 勾勒您将测试功能的接缝。应优先使用现有的接缝而不是新的接缝。尽可能使用最高级别的接缝。如果需要新的接缝，请在您能提供的最高点提出。代码库中的接缝越少越好 - 理想数量是一个。

与用户确认这些接缝是否符合他们的期望。

3. 使用下方的模板编写规范，然后将其发布到项目问题跟踪器。应用 `ready-for-agent` 分类标签 — 无需额外的分类。

<spec-template>

## 问题陈述

用户所面临的问题，从用户的角度来看。

## 解决方案

从用户的角度来看解决问题的方案。

## 用户故事

用户故事的编号列表。每个用户故事应采用以下格式：

1. 作为一个 <actor>，我想要一个 <feature>，以便 <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

此用户故事列表应非常详尽，并涵盖该功能的所有方面。

## 实施决策

做出的实施决策列表。这可能包括：

* 将要构建/修改的模块
* 将要修改的这些模块的接口
* 来自开发者的技术澄清
* 架构决策
* 模式变更
* API 协议
* 特定的交互

不要包含具体的文件路径或代码片段。它们可能会很快过时。

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## 测试决策

做出的测试决策列表。包括：

* 什么构成了好的测试的描述（仅测试外部行为，不测试实现细节）
* 将测试哪些模块
* 测试的先例（即代码库中类似类型的测试）

## 超出范围

本规范未涵盖事项的描述。

## 关于功能的任何进一步说明。

关于功能的任何进一步说明。

</spec-template>
