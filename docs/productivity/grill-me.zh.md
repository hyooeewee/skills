Quickstart:

```bash
npx skills add mattpocock/skills --skill=grill-me
```

```bash
npx skills update grill-me
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/productivity/grill-me)

## 功能说明

`grill-me` 会针对计划或设计进行持续的访谈，遍历决策树的所有分支，直到你和代理达成 **共识**。

它**一次问一个问题**并等待。它从不向你抛出一大堆问题——那会令人困惑——并且当一个问题可以通过阅读代码库来回答时，它会去阅读而不是提问。每个问题都带有代理自己的推荐答案，因此你是在对提议做出反应，而不是盯着空白的提示词。

## 何时使用

你通过输入 `/grill-me` 来调用它——代理不会自行使用它。

在构建之前使用它，当计划看起来大致正确但你能感觉到其中隐藏着未决的决策时——这正是你想找到薄弱点并将其暴露出来的时刻。如果你希望这种同样的审问也能留下 ADR 和术语表的书面记录，请改用 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)。如果工作量太大无法在一次会话中完成，且通往目标的道路仍然模糊不清——比如一个全新的项目或一个巨大的功能构建——请在更上游处使用 [wayfinder](https://aihero.dev/skills-wayfinder)，它首先将其绘制成决策地图，然后再合并回此流程。

## 决策树

会话将计划作为一棵决策树来遍历，逐一解决它们之间的依赖关系——父决策在附着其下的选择之前就已确定。目的不是为了快速达成一致；而是为了让每一个隐含的调用都变得明确，这样就没有重要的事情被默默地假设了。你会最终得到一个所有分支都已遍历过的计划。

`grill-me` 是 **无状态的**：它不写入任何内容，也不留下任何工作区。它可以在任何地方运行，唯一的产物就是对话本身中加深后的理解。这是与 [grill-with-docs](https://aihero.dev/skills-grill-with-docs) 的刻意对比，后者将相同的访谈捕获为持久的 ADR 和术语表。

## 在系统中的位置

`grill-me` 是一个随时可用的独立工具——即每当计划需要加固时你运行的构建前压力测试。它是 [grilling](https://aihero.dev/skills-grilling) 原语的无状态、用户调用的入口；它的最近邻是 [grill-with-docs](https://aihero.dev/skills-grill-with-docs)，这是一个有状态的兄弟工具，运行相同的访谈但额外将决策记录为 ADR 和术语表。如果结果是你要写下来的规范，请转交给 [to-spec](https://aihero.dev/skills-to-spec)，它会将已确定的理解综合成规范，而无需重新对你进行访谈。当你不确定哪个流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt) 会为你指路。
