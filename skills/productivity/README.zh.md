# 生产力

通用工作流工具，不针对特定代码。

## 用户调用

仅在输入时可用（Claude Code: `disable-model-invocation: true`; Codex: `agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

* **[grill-me](./grill-me/SKILL.md)** — 被反复盘问关于计划或设计的问题，直到决策树的所有分支都得到解决。
* **[handoff](./handoff/SKILL.md)** — 将当前对话压缩为交接文档，以便另一个代理继续工作。
* **[teach](./teach/SKILL.md)** — 在多个会话中向用户教授新技能或概念，使用当前目录作为有状态的教学工作区。
* **[writing-great-skills](./writing-great-skills/SKILL.md)** — 编写和编辑优秀技能的参考：使技能具有可预测性的词汇和原则。

## 模型调用

模型或用户可达（丰富的触发短语，以便模型能够调用它们）。

* **[grilling](./grilling/SKILL.md)** — 反复面试用户关于计划、决策或想法的问题，直到决策树的所有分支都得到解决。
