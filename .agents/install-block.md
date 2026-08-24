# 标准安装块

一个安装说明，一套措辞。`README.md`、`.changeset/*` 以及 `docs/` 下的每个页面都必须只使用**这一套**说法，而不是其他任何内容。先在这里修改，再传播到各处。

`mattpocock-skills` 列在 **Claude Code 的官方市场**（配置名称为 `claude-plugins-official`，源仓库为 `anthropics/claude-plugins-official`）中，这是所有 Claude Code 安装都自带的。无需先添加市场。官方 Anthropic 市场默认启用自动更新（[discover-plugins](https://code.claude.com/docs/en/discover-plugins)），因此“更新会自动到达”是确凿的陈述，而非一种希望。

## Claude Code: the plugin

<canonical-block name="claude-code">

```bash
claude plugins install mattpocock-skills
```

或者在会话内部：

```
/plugin install mattpocock-skills
```

它位于 Claude Code 的官方市场中，因此无需预先添加任何内容，更新会自动到达。

</canonical-block>

## Codex, and other agents: skills.sh

该插件仅适用于 Claude Code。在其他所有地方，[skills.sh](https://skills.sh/mattpocock/skills) 会将可编辑的技能文件复制到项目中。在 `README.md` 上使用整套形式：

<canonical-block name="skills-sh-whole-set">

```bash
npx skills@latest add mattpocock/skills
```

选择您想要的技能，以及要在哪个编码代理上安装它们。**安装程序允许您选择要安装哪些技能：确保 `setup-matt-pocock-skills` 是其中之一。**

</canonical-block>

……而在单独指名某个技能的地方，使用单技能形式。请注意，**`docs/` 页面不是此块的使用方**：ai-hero 会在正文上方渲染安装小部件，因此页面如果把这些命令写出来，就会重复。参见 [writing-docs.md](./writing-docs.md)。

<canonical-block name="skills-sh-one-skill">

```bash
npx skills@latest add mattpocock/skills --skill=<name>
```

```bash
npx skills@latest update <name>
```

</canonical-block>

`skills@latest` 是三种形式中固定的写法。`docs/` 下的页面原先带有这些命令的副本；这些块现在被删除而不是被修正，因为站点会自行渲染安装命令。

## 两种途径互斥

该插件是一个您订阅的托管、只读捆绑包。skills.sh 会写入您拥有并编辑的文件。同时安装两者会让用户拥有所有技能的两份副本：始终说“选一个”。

## 不是安装说明

`.claude-plugin/marketplace.json` 将该仓库设为其自身的单一插件市场（`/plugin marketplace add mattpocock/skills`，然后 `/plugin install mattpocock-skills@mattpocock`）。官方列表覆盖了它。它保留作为直接安装仓库的回退方案（未发布的提交或分支），并且**不**向用户文档化。
