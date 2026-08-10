# 生产力

通用工作流工具，不针对特定代码。

## 用户调用

仅在输入时可用（Claude Code: `disable-model-invocation: true`; Codex: `agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

* **[grill-me](./grill-me/SKILL.md)** — 对计划或设计进行无情的面试，直到设计树的每个分支都得到解决。
* **[handoff](./handoff/SKILL.md)** — 将当前对话压缩为交接文档，以便另一个代理继续工作。
* **[teach](./teach/SKILL.md)** — 在多个会话中向用户教授新技能或概念，使用当前目录作为有状态的教学工作区。
* **[to-questionnaire](./to-questionnaire/SKILL.md)** — 将你无法独自回答的决定转化为一份 Markdown 问卷，交给能回答的那个人——可以异步填写，也可以在会议中一起填写。
* **[wait-what](./wait-what/SKILL.md)** — 当消息无法送达的瞬间，立即使用此功能。代理会用你缺失的上下文重新解释它，使用简单的英语，并使用你的 `CONTEXT.md` 词汇表。

## 模型调用

模型或用户可达（丰富的触发短语，以便模型能够调用它们）。

* **[grilling](./grilling/SKILL.md)** — 对用户关于计划、决定或想法进行无情的面试，直到设计树的每个分支都得到解决。
* **[writing-for-agents](./writing-for-agents/SKILL.md)** — 为代理撰写文档：技能、AGENTS.md/CLAUDE.md，以及代理通过指针指向的任何文档。
