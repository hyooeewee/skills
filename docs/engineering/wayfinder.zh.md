Quickstart:

```bash
npx skills add mattpocock/skills --skill=wayfinder
```

```bash
npx skills update wayfinder
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/wayfinder)

## 功能说明

`wayfinder` 接管一个超出单次会话处理能力的任务——它被笼罩在迷雾中，此时从起点到目标的路径尚不可见——并将其绘制成你问题追踪器上的 **共享地图** 和 **决策票据**，然后逐一解决它们，直到路径清晰。它 **制定计划，不执行操作**：每个票据都解决一个决策——一个需要解决的问题，而不是需要执行的一块构建切片——当在有人去构建它之前没有什么需要决定的时候，地图就完成了——因此它产生的是决策，而不是交付物。

## 何时使用

你通过输入 `/wayfinder` 来调用它——代理不会自行调用它。

当一项工作 **超过单个代理会话的承载能力** 且其 **目的地** 的路线仍然模糊不清时——你可以感觉到工作的轮廓但还无法将其写下来作为规范或计划——请使用它。对于将 *已经清晰* 的线程转换为规范，请使用 [to-spec](https://aihero.dev/skills-to-spec)；对于将已理解的计划拆分为可构建的票据，请使用 [to-tickets](https://aihero.dev/skills-to-tickets)。Wayfinder 位于两者的上游：当迷雾太浓而无法直接制定规范时，你运行的就是它。

## 前置条件

地图及其票据存在于仓库的问题跟踪器上，因此 wayfinder 需要由 [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) 建立的跟踪器连接——它植入了一个“Wayfinding operations”部分，描述了地图、子票据、阻塞和前沿查询在 GitHub、GitLab 或本地 Markdown 中的表达方式。如果没有该文档，wayfinder 默认使用本地 Markdown 地图。

## 地图是索引，迷雾是前沿

**地图**是单个 `wayfinder:map` 问题，其票据是其子问题——一个整个团队都可以观看的共享 URL。它是一个 **索引，而不是存储**：每个决策都确切地存在于一个地方（它的票据），地图只进行摘要和链接，从不重复陈述。会话以低分辨率加载地图，并按需放大查看单个票据。

在实时票据之外是 **战争迷雾**——你可以察觉到即将做出的决策，但还无法确定下来。判断某事是票据还是迷雾的测试是，你是否能 *现在就精确地陈述问题*，而不是能否回答它。解决票据会清除它前面的迷雾，将现在可以规范化的内容 **提升为** 新的票据。**前沿** 是开放的、未阻塞的、未认领的票据——已知事物的边缘——这是跟踪器原生阻塞功能在视觉上的呈现，因此你无需打开地图就能看到哪些是可执行的。迷雾只向 **目的地** 聚集；它之后的工作被判定为 **超出范围**、关闭，永远不会毕业。

每张票据都是 **HITL**（人在回路中——质询、原型）或 **AFK**（仅代理——研究）；HITL 票据只能通过实时交换解决，因此代理永远不会回答自己的问题。研究保持为真实的票据——下游决策依赖的共享阻塞器——但由于它是 AFK，会话不会停下来阅读：它会启动一个 `/research` **子代理** 并行烧毁票据，保持前沿快速，并将发现记录在一个一次性 `research/<name>` 分支上。

## 判断是否生效

* 命名 **目的地** 是第一项行动——在任何票据存在之前——因为它确定了每个票据衡量的范围。
* 一个地图是一个 `wayfinder:map` 问题；票据是其子问题，通过 **名称** 引用，而不是裸露的 `#42`。
* 会话最多解决 **一张票据**（研究票据除外），将答案记录为解决评论，关闭票据，并附加一行指向 *迄今为止的决策* 的指针。
* 如果初始的 grilled 面板没有揭示迷雾，它就会停止并告诉你旅程足够小，可以跳过地图。

## 在系统中的位置

`wayfinder` 是一个大型概念的 **入口**：一项太大且太模糊而无法一次完成的任务会生成一个清晰的决策地图，然后合并到主要的构建流程中。当迷雾被推回且路径清晰时，将其移交 [to-spec](https://aihero.dev/skills-to-spec) 以安排多会话构建（或者，如果任务结果很小，则直接实现）。它依赖 [grilling](https://aihero.dev/skills-grilling) 和 [domain-modeling](https://aihero.dev/skills-domain-modeling) 来解决单个票据，并依赖 [prototype](https://aihero.dev/skills-prototype) 和 [research](https://aihero.dev/skills-research) 来解决需要它们的票据类型。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
