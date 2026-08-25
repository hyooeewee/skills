# 进行中

Beta。这些技能故意公开，目的是让你尝试它们并告诉我哪里出错了。在它们晋升到稳定类别之前，它们被排除在插件和顶层 README 之外，没有文档页面，并且它们可以在没有警告的情况下更改或消失。

插件不会提供这些。请直接安装其中一个：

```bash
npx skills@latest add mattpocock/skills --skill=<name>
```

* **[loop-me](./loop-me/SKILL.md)**：通过多次会话，利用当前目录作为有状态的工作区，将你的思路反复打磨成可执行的工作流规范。由用户调用。
* **[writing-beats](./writing-beats/SKILL.md)**：将文章构建为一连串节奏的旅程，类似“选择你自己的冒险”风格。选择一个起始节奏，只写那个节奏，然后转向下一个，直到文章达到自然的结尾。
* **[writing-fragments](./writing-fragments/SKILL.md)**：一个挖掘片段（写作中的多样化素材）并将其追加到单个文档中作为未来文章原材料的会话。
* **[writing-shape](./writing-shape/SKILL.md)**：拿一份原材料 markdown 文件，逐段将其梳理成文章，并在每一步阐述格式选择的理由。
* **[claude-handoff](./claude-handoff/SKILL.md)**：将当前的对话交接给一个全新的后台代理，该代理立即开始工作，并通过 `claude --bg` 注入交接摘要。由用户调用。
* **[setup-ts-deep-modules](./setup-ts-deep-modules/SKILL.md)**：将 dependency-cruiser 连接到 TypeScript 仓库，以便每个包都是一个深度模块：实现隐藏在子文件夹中，只能通过其入口文件访问，测试通过这些文件来执行。由用户调用。
* **[implement-spec](./implement-spec/SKILL.md)**：在一个分支上实现整个规范。将工单作为任务图而不是列表来处理，在就绪前沿运行实现者子代理以实现最大并发，并将结果作为一个 PR 交付。由用户调用。
* **[retro](./retro/SKILL.md)**：在会话后建议改进编码代理的环境（引导文件、编码标准、自动化检查、工具）。存根：仅包含设计笔记，尚未实现功能。由用户调用。
