# 标准安装块

一个安装说明，一套措辞。`README.md`、`.changeset/*` 以及 `docs/` 下的每个页面都必须只使用**这一套**说法，而不是其他任何内容。先在这里修改，再传播到各处。

`mattpocock-skills` 列在 **Claude Code 的官方市场**中——配置名为 `claude-plugins-official`，源仓库为 `anthropics/claude-plugins-official`——每个 Claude Code 安装都内置了该市场。无需先添加任何市场。Anthropic 官方市场默认启用自动更新（[discover-plugins](https://code.claude.com/docs/en/discover-plugins)），因此“更新会自动到达”是事实，而不是一种期望。

## Claude Code — 插件

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

## Codex 及其他智能体 — skills.sh

该插件仅适用于 Claude Code。在其他所有地方，[skills.sh](https://skills.sh/mattpocock/skills) 会将可编辑的技能文件复制到项目中。在 `README.md` 上使用整套形式：

<canonical-block name="skills-sh-whole-set">

```bash
npx skills@latest add mattpocock/skills
```

选择你想要的技能，以及要将它们安装到哪些编码智能体上。**安装程序允许你选择要安装哪些技能——请确保 `setup-matt-pocock-skills` 是其中之一。**

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

该插件是你订阅的一个受管、只读的捆绑包。skills.sh 则会写入你拥有并可编辑的文件。两者都安装会让用户把每个技能拿到两次——始终要说“选一个”。

## 不是安装说明

`.claude-plugin/marketplace.json` 使该仓库成为自己的单插件市场（先执行 `/plugin marketplace add mattpocock/skills`，再执行 `/plugin install mattpocock-skills@mattpocock`）。官方列表取代了它。它保留下来，作为直接安装该仓库（未发布的提交或 fork）的后备方案，并且**不**向用户公开文档。
