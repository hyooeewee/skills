# 进行中

Beta。这些技能故意公开——尝试它们并告诉我哪里出了问题。在它们毕业到稳定分类之前，它们被排除在插件和顶层 README 之外，没有文档页面，并且可以在没有警告的情况下更改或消失。

插件不会给你这些。直接安装一个：

```bash
npx skills@latest add mattpocock/skills --skill=<name>
```

* **[loop-me](./loop-me/SKILL.md)** — 在多个会话中迫使自己生成可实现的 workflow 规范，使用当前目录作为有状态的 workspace。用户调用。
* **[writing-beats](./writing-beats/SKILL.md)** — 将文章塑造成一系列节拍的旅程，类似文字冒险风格。选择一个起始节拍，只写那个节拍，然后切换到下一个，直到文章自然结束。
* **[writing-fragments](./writing-fragments/SKILL.md)** — 迫使你吐露写作片段——异构的核心内容——并将它们追加到单个文档中，作为未来文章的原始材料。
* **[writing-shape](./writing-shape/SKILL.md)** — 拿到一个原始材料的 markdown 文件，一段一段地将它塑造成一篇文章，并在每一步论证格式选择。
* **[claude-handoff](./claude-handoff/SKILL.md)** — 将当前对话移交到一个新的后台 agent，它立即接手工作，并通过 `claude --bg` 注入交接摘要。用户调用。
* **[setup-ts-deep-modules](./setup-ts-deep-modules/SKILL.md)** — 将 dependency-cruiser 连接到 TypeScript 仓库，使得每个包都是一个深层模块——实现隐藏在子文件夹中，只能通过其入口文件访问，测试通过这些文件来使用它。用户调用。
