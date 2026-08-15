# 生产力

通用工作流工具，不特定于代码。

## 用户调用

仅在您输入它们时才可访问（Claude Code：`disable-model-invocation: true`；Codex：`agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

* **[grill-me](./grill-me/SKILL.md)** — 针对计划或设计接受持续盘问，直到设计树的每一条分支都被解决。
* **[handoff](./handoff/SKILL.md)** — 将当前对话压缩成一份交接文档，以便另一个代理可以继续这项工作。
* **[teach](./teach/SKILL.md)** — 在多次会话中教用户掌握一项新技能或概念，将当前目录用作有状态的教学工作区。
* **[to-questionnaire](./to-questionnaire/SKILL.md)** — 将你独自无法定夺的决策变成一份 Markdown 问卷，交给那位唯一能回答的人——可异步填写，也可在会议中共同完成。
* **[wait-what](./wait-what/SKILL.md)** — 当消息未能被理解时立刻触发。代理会使用你的 `CONTEXT.md` 词汇，用通俗的英语、结合你所缺失的上下文重新解释。

## 模型调用

模型或用户均可访问（采用丰富的触发措辞，便于模型主动调用）。

* **[grilling](./grilling/SKILL.md)** — 就计划、决策或想法对用户进行无情追问，直到设计树的每一条分支都被解决。
* **[writing-for-agents](./writing-for-agents/SKILL.md)** — 为代理编写文档：技能、AGENTS.md/CLAUDE.md，以及代理通过指针访问的任何文档。
