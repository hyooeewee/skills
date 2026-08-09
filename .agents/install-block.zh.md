# 标准安装块

一种安装方式，一种措辞。`README.md`、`.changeset/*` 以及 `docs/` 下的每一页都必须说 **这句话**，除此之外别无他言。请先在这里修改，然后进行传播。

`mattpocock-skills` 列在 **Claude Code 的官方市场** 中——配置名称为 `claude-plugins-official`，源代码仓库为 `anthropics/claude-plugins-official`——这是每个 Claude Code 安装自带的功能。不需要先添加市场。官方 Anthropic 市场默认启用了自动更新（[discover-plugins](https://code.claude.com/docs/en/discover-plugins)），因此“更新会自动到来”是一个真实的声明，而不是一种希望。

## Claude Code — 插件

<canonical-block name="claude-code">

```bash
claude plugins install mattpocock-skills
```

或者，在会话内部：

```
/plugin install mattpocock-skills
```

它在 Claude Code 的官方市场里，所以不需要先添加任何东西，更新会自动到来。

</canonical-block>

## Codex, and other agents — skills.sh

该插件仅适用于 Claude Code。在其他地方，[skills.sh](https://skills.sh/mattpocock/skills) 会将可编辑的技能文件复制到项目中。在 `README.md` 上使用整套形式：

<canonical-block name="skills-sh-whole-set">

```bash
npx skills@latest add mattpocock/skills
```

选择你想要的技能，以及要在哪个编码代理上安装它们。**安装程序允许你选择要获取哪些技能——确保 `setup-matt-pocock-skills` 在其中之一。**

</canonical-block>

……以及单技能形式，即任何单独命名的技能。请注意，**`docs/` 页面不是此块的消费者**：ai-hero 在正文上方渲染安装小部件，因此写出命令的页面会重复它。请参阅 [writing-docs.md](./writing-docs.md)。

<canonical-block name="skills-sh-one-skill">

```bash
npx skills@latest add mattpocock/skills --skill=<name>
```

```bash
npx skills@latest update <name>
```

</canonical-block>

`skills@latest` 是这三者中的固定拼写。`docs/` 下的页面以前携带这些命令的副本；那些块现在已被删除而不是被修正，因为网站本身会渲染安装命令。

## 两条路径是互斥的

该插件是一个你订阅的托管、只读包。skills.sh 会写入你拥有并编辑的文件。同时安装两者会让用户拥有每个技能的副本两次——始终说“选一个”。

## 非安装说明

.claude-plugin/marketplace.json 将仓库变成了它自己的单一插件市场（`/plugin marketplace add mattpocock/skills`，然后 `/plugin install mattpocock-skills@mattpocock`）。官方列表覆盖了它。它保留作为直接安装仓库的回退方案——未发布的提交或分叉版本——并且**未**向用户文档化。
