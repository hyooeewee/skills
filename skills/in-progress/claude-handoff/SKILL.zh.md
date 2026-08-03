---
name: claude-handoff
description: 将当前对话移交给一个新的后台代理，使其立即继续工作。
argument-hint: What will the next session be used for?
disable-model-invocation: true

---

写一份当前对话的交接摘要，以便新的代理可以继续工作。不要保存它，而是启动一个以该摘要作为提示词的后台代理：`claude --bg --name "<descriptive name>" "<handoff summary>"`。它在当前工作目录中启动并立即返回；用户使用 `claude agents` 来管理它。

始终使用 `-n`/`--name` 加上一个描述性名称（例如 `--name "Fix login bug"`）——它设置在任务列表、会话选择器和终端标题中显示的显示名称。

在摘要中包含一个"建议技能"部分，建议代理应该调用的技能。

不要重复其他工件中已经捕获的内容（PRDs、计划、ADR、问题、提交、差异）。而是通过路径或 URL 引用它们。

删减任何敏感信息，如 API 密钥、密码或个人身份信息——摘要将成为代理的提示词。

如果用户传递了参数，将它们视为下一个会话将关注的内容的描述，并相应地调整摘要。
