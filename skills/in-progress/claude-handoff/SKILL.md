---
name: claude-handoff
description: 将当前对话移交给一个全新的后台代理，由其立即接手工作。
argument-hint: What will the next session be used for?
disable-model-invocation: true

---

编写当前对话的交接摘要，以便新代理可以继续开展工作。不要保存它，而是启动一个以摘要作为提示词（prompt）的后台代理：`claude --bg --name "<descriptive name>" "<handoff summary>"`。它会在当前工作目录中启动并立即返回；用户通过 `claude agents` 管理它。

始终使用描述性名称传递 `-n`/`--name`（例如 `--name "Fix login bug"`）；这会设置在任务列表、会话选择器和终端标题中显示的显示名称。

在摘要中包含一个“建议技能”部分，指明下一个代理应针对哪些技能调用 Skill 工具。

不要重复其他工件（规格说明、计划、ADR、问题、提交、差异）中已捕获的内容。改为通过路径或 URL 引用它们。

由于摘要将成为代理的提示词，请隐去任何敏感信息，例如 API 密钥、密码或个人身份信息。

如果用户传递了参数，请将其视为对下一个会话关注内容的描述，并据此调整摘要。
