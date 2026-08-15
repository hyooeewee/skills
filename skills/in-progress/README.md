# 进行中

Beta 版。这些技能特意公开——试用它们并告诉我哪里出了问题。在它们进入稳定分类之前，它们不会包含在插件和顶层 README 中，也没有文档页面，并且可能随时更改或消失，恕不另行通知。

插件不会提供这些。请直接安装其中一个：

```bash
npx skills@latest add mattpocock/skills --skill=<name>
```

* **[loop-me](./loop-me/SKILL.md)** — 通过多次会话，将当前目录作为有状态的工作区，让自己打磨出可实施的工作流规范。由用户调用。
* **[writing-beats](./writing-beats/SKILL.md)** — 以节拍旅程的形式来构思文章，采用“选择你自己的冒险”风格。选择一个起始节拍，只写那个节拍，然后转向下一个，直到文章自然收尾。
* **[writing-fragments](./writing-fragments/SKILL.md)** — 一次追问式会话，挖掘你的写作片段——各种异质的小块文字——并将它们追加到单个文档中，作为未来文章的原始素材。
* **[writing-shape](./writing-shape/SKILL.md)** — 将一份原始素材的 Markdown 文件逐段塑造成一篇文章，并在每一步讨论格式选择。
* **[claude-handoff](./claude-handoff/SKILL.md)** — 将当前对话移交给一个新的后台代理，该代理会立即接手工作，并通过 `claude --bg` 预先注入交接摘要。由用户调用。
* **[setup-ts-deep-modules](./setup-ts-deep-modules/SKILL.md)** — 将 dependency-cruiser 接入 TypeScript 仓库，使每个包都成为一个深层模块——实现隐藏在子文件夹中，只能通过其入口文件访问，测试也通过这些入口文件来执行。由用户调用。
