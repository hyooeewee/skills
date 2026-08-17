---
name: handoff
description: 将当前对话压缩为交接文档，供另一个代理接手。
argument-hint: What will the next session be used for?
disable-model-invocation: true

---

编写一份交接文档，总结当前对话，以便新代理可以继续这项工作。保存到用户操作系统的临时目录，而不是当前工作区。

在文档中包含一个“建议技能”部分，说明下一个代理应调用 Skill 工具来获取哪些技能。

不要重复其他工件（规格说明、计划、ADR、问题、提交、差异）中已捕获的内容。改为通过路径或 URL 引用它们。

隐去任何敏感信息，例如 API 密钥、密码或个人身份信息。

如果用户传入了参数，请将其视为对下一阶段工作重点的描述，并据此调整文档。
