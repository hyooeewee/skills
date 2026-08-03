Quickstart:

```bash
npx skills add mattpocock/skills --skill=grilling
```

```bash
npx skills update grilling
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/productivity/grilling)

## 功能说明

`grilling` 是一种持续的面试，旨在在构建之前对计划或设计进行压力测试。它会逐个分支地遍历决策树，逐一解决决策之间的依赖关系，直到您和代理达成一致的理解。

它会一次问**一个问题**，在下一个问题到来之前等待你的回答——绝不是一个令人困惑的批量列表。每个问题都附带代理的推荐答案，如果代码库能够自行解决某些问题，它会直接探索这些方案，而不是问你。只有在确认已达成共同理解后，它才会开始执行计划。

## 何时使用

输入 `/grilling`，或者当任务匹配时，代理会自动调用它——这是底层的基础机制，而不仅仅是面向用户的入口。

当计划或设计仍存在薄弱点，且你希望在编写代码之前将其暴露出来时，使用它。实际上，你通常通过它的两个封装器之一来调用它，而不是直接使用其名称：对于简单的 grilled 会话，使用 [grill-me](https://aihero.dev/skills-grill-me)；如果希望会话在进行过程中同时写入 ADR 和术语表，则使用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)。

## 决策树

心理模型是一棵**决策树**：每个计划都会分支成决策，而这些决策又相互依赖。`grilling` 会逐个节点地遍历这棵树，因此早期的回答可以改变接下来出现的问题。这就是为什么问题会逐一出现并按依赖顺序排列——一股并行问题的洪流会破坏那种使面试最终达成共识的结构。

## 专门提取

`grilling` 是该面试技术的**单一事实来源**，被拆分出来作为模型调用的**原语**，这样每个需要面试的技能都可以调用它，而不是重新发明一个。 [grill-me](https://aihero.dev/skills-grill-me) 和 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 是它的两个用户调用的入口，但 [improve-codebase-architecture](https://aihero.dev/skills-improve-codebase-architecture) 和 [triage](https://aihero.dev/skills-triage) 也依赖它来压力测试它们自己的决策。

将该技术集中在一个地方，意味着当你只想进行面试时，也可以直接使用它，而不必受其包装器额外添加的 ADR 编写或工单定义的干扰。

## 在系统中的位置

`grilling` 是主构建链下的面试**原语**：[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 会在 [to-spec](https://aihero.dev/skills-to-spec) 编写规范之前运行它以明确上下文。当你不确定哪个入口点合适时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指引方向。
