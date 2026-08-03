Quickstart:

```bash
npx skills add mattpocock/skills --skill=improve-codebase-architecture
```

```bash
npx skills update improve-codebase-architecture
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/improve-codebase-architecture)

## 功能说明

`improve-codebase-architecture` 扫描代码库以寻找**加深机会**——即那些浅层模块（其接口几乎与它所隐藏的事物一样复杂）可以变成深层模块的地方——将它们呈现为独立的视觉 HTML 报告，然后通过你选择的任何一个进行审查。

它不会给你一个平铺的重构列表。每个候选者都必须通过**删除测试**——删除此模块是否会让复杂性*集中*在一个更小的接口背后，还是仅仅将其移动？只有“集中”的情况才会获得一张卡片。正是这个过滤器阻止报告变成通用的清理建议。

除非你将其指向特定区域，它也会将范围限定在实际开发落地的地方——通过阅读最近的提交，偏向于你仍在更改的代码。加深模块的回报在于使其未来的更改更容易，因此它会给最近更改的仓库部分增加额外的权重。

## 何时使用

你通过输入 `/improve-codebase-architecture` 来调用它——代理不会自行使用它。

将其作为定期健康检查来使用：每隔几天，或者每当代码库开始感觉理解一个概念需要在小模块之间来回跳跃太多时。它读取现有的架构并建议在哪里加深它。如果你已经知道你想重新设计的模块，只需要思考它的词汇，请改用 [codebase-design](https://aihero.dev/skills-codebase-design)——这个技能是发现候选者的调查；那个是设计工作台。

## 加深机会

整个技能围绕一个概念运转：**深度**。一个深层模块在小的、稳定的接口背后隐藏了大量功能；而一个浅层模块通过几乎与下方代码一样宽的接口泄漏其实现。报告寻找浅薄之处——仅为了可测试性而提取的纯函数，而真正的 Bug 隐藏在它们是如何被调用的（没有**局部性**），泄漏其**接缝**的模块，不打开五个文件就无法理解的概念——并提出能够修复它的加深方案。

它使用共享的设计词汇（**模块**、**接口**、**深度**、**接缝**、**适配器**、**杠杆作用**、**局部性**）以及你项目自身的领域语言（来自 `CONTEXT.md`），因此候选内容读起来是“加深订单录入模块”，而不是“重构 FooBarHandler”。

## 报告，然后是审查

输出是一个浏览器就绪的 HTML 文件，写入你的操作系统临时目录——没有任何内容会进入仓库。每个候选者都是一张卡片，包含涉及的文件、摩擦点、通俗易懂的解决方案、在局部性和杠杆作用方面的益处、前后对比图，以及一个 `Strong` / `Worth exploring` / `Speculative` 徽章。它以它打算首先解决的那个作为结尾。

然后它停止并询问你想探索哪一个。选一个，它就会在设计中运行 [grilling](https://aihero.dev/skills-grilling) 循环——约束条件、接缝后面是什么、哪些测试能存活——随着决策的明朗化，在线更新领域模型。

## 在系统中的位置

`improve-codebase-architecture` 是**定期维护**——每隔几天运行一次，而不是作为链中的一个步骤。它的邻居是 [codebase-design](https://aihero.dev/skills-codebase-design)，它拥有每个候选者所使用的深度和接缝词汇，[grilling](https://aihero.dev/skills-grilling)，在你选择候选者后遍历决策树，以及 [domain-modeling](https://aihero.dev/skills-domain-modeling)，它在重新设计定稿时保持 `CONTEXT.md` 和 ADRs 的最新状态。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
