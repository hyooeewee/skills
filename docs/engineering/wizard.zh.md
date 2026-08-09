## 功能说明

`wizard` 生成一个交互式 bash 脚本，引导用户分步完成手动操作——连接第三方服务、运行一次性迁移、将项目从状态 A 迁移到状态 B。它会打开每个 URL，告诉你点击什么、复制什么，捕获返回的内容，并将其写入 `.env` 文件和 GitHub Actions secrets。

[agent](https://www.aihero.dev/ai-coding-dictionary/agent) 负责编写脚本；它从不运行。你在自己的机器上运行。所以，wizard 不是一个你需要遵循的指令列表——它是一个驱动流程并保存状态的程序，你的部分工作是点击、粘贴并按回车。

## 何时使用

You can type `/wizard`, and the agent can also reach for it on its own. When it hits a step you have to take — a key it can't mint, a dashboard it can't click — it builds you a wizard instead of writing the instructions into the chat, where they scroll away.

当阻碍你的下一步是穿过仪表盘时，使用它：

| 情况                   | wizard 做什么                      |
| -------------------- | ------------------------------- |
| 新开发人员需要在应用启动前配置六个服务  | 依次打开每个仪表盘，捕获密钥，将它们写入 `.env`和 CI |
| 一次性迁移需要按特定顺序切换开关     | 对确认门背后的不可逆步骤进行排序                |
| 项目必须从状态 A 移动到状态 B 一次 | 引导转换过程并报告它无法完成的工作               |
| 你正准备将这些步骤写入 README   | 写入一个可执行版本，这样它就不会悄无声息地腐烂         |

不要用它来*决定*要构建什么；为此，[grill-with-docs](https://aihero.dev/skills-grill-with-docs) 和 [to-spec](https://aihero.dev/skills-to-spec) 才是工具。

## 前置条件

生成一个没有任何要求。它编写的 wizard 运行在 bash 上，当某个阶段设置 GitHub secret 或变量时，会使用 `gh`。如果 `gh` 缺失或未认证，该阶段将变为警告，最后的摘要会告诉你需要手动设置什么，而不是导致运行失败。

## 阶段

**阶段**是屏幕上的一项专注任务。脚本会在阶段之间清除终端，因此会溢出屏幕的阶段会丢失滚动掉的部分。你按依赖顺序编写阶段并设置 `TOTAL_STAGES`，这会驱动进度显示。

作用域发生在写入行之前。[skill](https://www.aihero.dev/ai-coding-dictionary/skill) 会读取仓库而不是冷启动询问：`.env*`、`docker-compose*`、框架配置，以及 `.github/workflows/` 中的每个 `secrets.*` / `vars.*` 引用——这些都是 wizard 必须生成的值。然后它向你展示有序的阶段列表以供确认，之后才将每个阶段映射到人类遵循的确切路径（“仪表盘 → 开发者 → API 密钥 → 显示测试密钥 → 复制”）。在不知道当前 UI 的地方，它会问你或查阅文档，而不是编造点击操作。

对于每个捕获的值，作用域决定了它的落脚点：

| 目的地                 | 何时                        |
| ------------------- | ------------------------- |
| `.env`仅             | 本地开发需要它，CI 不需要            |
| GitHub secret       | CI 读取它，并且它是敏感的            |
| GitHub variable     | CI 读取它，并且它是公开的            |
| 两者 `.env`和一个 secret | 本地开发和 CI 都需要它             |
| 无处                  | 阶段是一个纯操作——一个开关被拨动，一个计划被升级 |

## 模板已经解决了 UX 问题

[template](https://github.com/mattpocock/skills/blob/main/skills/engineering/wizard/template.sh) 提供了完整的体验：带剩余时间的进度、确认门、跨平台 URL 打开（包括 WSL）、用于密钥的隐藏输入、幂等的 `.env` upserts、`gh secret` / `gh variable` 写入，以及它必须跳过的所有内容的最终摘要。`STAGES` 标记上方的所有内容都是固定库，在每个 wizard 中都相同且从未手动编辑。一致性就是重点。你的工作只是对流程进行作用域并编写其阶段。

编写 wizard 的 agent 从不端到端地运行它，因为它会打开浏览器并等待人类输入。相反，它进行静态验证：`bash -n`，在可用时使用 `shellcheck`，以及每个值都落在作用域所说位置的跟踪，其中每个 `set_secret` 名称都匹配 CI 中的真实 `secrets.*` 引用。相应地调整你的期望——第一次运行是给你的，那次运行就是测试。

## 默认为临时

| 你拥有什么                 | 如何处理脚本                                    |
| --------------------- | ----------------------------------------- |
| 一次性迁移、个人设置、你永远不会重复的转换 | 保存到临时文件或 `scripts/`路径，运行它，删除它             |
| 下一个仓库成员也会需要的设置路径      | 将其提交并从 README 链接它，这样他们就会运行脚本而不是再次询问 agent |

## 常见问题

**我的 API 密钥会进入模型的上下文吗？**

不会。agent 负责编写脚本；它不运行。你自己运行脚本，它使用隐藏的终端输入捕获密钥，并将其直接写入 `.env` 或 `gh secret`。wizard 是一个 CLI，模型与它没有连接。一个注意事项：这适用于 wizard 在运行时捕获的值。如果你在作用域流程时将密钥粘贴到聊天中，它就像任何其他粘贴的文本一样在 [context](https://www.aihero.dev/ai-coding-dictionary/context) 中。

**我可以回去修正我输错的值吗？**

运行中途不能。没有后退按钮——阶段向前运行，第 3 阶段回答错误意味着 Ctrl-C 和重新运行。重新运行按设计来说是廉价的：任何已经写入 `.env` 的值都会作为默认值再次提供，所以你按 Enter 通过你做对了的阶段，只重新输入错误的那个。这在发布周期间出现过，之后一直未关闭：“很喜欢！不过有一件事——有没有办法回去修正你输入的内容？”

有一个相关的开放 bug。`ask` 提示符中的箭头键会插入 `^[[D` / `^[[C` 而不是移动光标，因为提示符使用 `read -r` 而不是 Readline ([issue #741](https://github.com/mattpocock/skills/issues/741))。退格键有效；箭头键无效。删除回退到错误，而不是将光标移入其中。

**它知道我已经设置好了什么吗？**

部分知道，比发布时的反应假设的要少。它在询问之前会读取仓库——你的 `.env` 文件、`docker-compose`、框架配置、CI 中的 `secrets.*` 引用——所以它的作用域是真正缺失的值，而不是像 README 那样从零开始。它不检查第三方服务。如果 `.env` 中存在密钥，wizard 会将其重新提供并按 Enter 保留；如果你已经创建了 Stripe 帐户但从未保存密钥，wizard 仍然会带你到仪表盘。

**它在流程中处于什么位置——在 grilling 和 spec 之后？**

特定位置没有。它是一个独立的，不是链式步骤。常见的猜测是 `/grill-with-docs → /to-spec → /wizard`，该序列没问题，但触发点是手动流程的出现，这可能发生在任何时间：开始之前、构建中途或发布很久之后。它也可以作为发现工具使用——作用域会揭示任务隐藏的先决条件，例如你在承诺工作之前没有想到的那三个 API 密钥。

**它能在 Claude Code 之外工作吗？**

该产物可以无条件工作：它是一个普通的 bash 脚本，不关心是哪个 [harness](https://www.aihero.dev/ai-coding-dictionary/harness) 生成的它。该技能本身是由模型调用的，所以它列在所有地方——在 Claude Code 中输入 `/wizard`，在 Codex 中输入 `$wizard`，或者只是描述你卡住的设置。由模型调用也使其远离 [#693](https://github.com/mattpocock/skills/issues/693)，其中 Claude 的桌面和 web 界面从 [model](https://www.aihero.dev/ai-coding-dictionary/model)'s 列表中移除了 *user-invoked* 技能并将它们报告为未安装。

**这以前不是用户调用的吗？**

是的。现在它是模型调用的，所以 agent 在遇到你必须采取的步骤时会主动调用它。你以前能做的任何事情都没有失效——模型调用*增加了* agent 的触达范围，它从未移除你的触达范围，所以 `/wizard` 的行为与以前完全相同。变化的是它退休的失败模式：agent 在构建中途遇到凭据墙，并将六个编号步骤倾倒到聊天中供你手动遵循。

**以前在 `in-progress/`—— 现在在哪里？**

`engineering/`, as of v1.2. It graduated out of the beta bucket and now ships in the plugin, so it arrives with the rest of the promoted set rather than needing an individual install. Its behaviour didn't change on graduation.

## 判断是否生效

* 你会看到一个有序的阶段列表，以及每个阶段产生的值，并要求你确认 —— 在任何脚本存在之前。
* 在请求该页面的值之前，每个 URL 都会打开。你永远不会被要求粘贴你没有被发送去获取的内容。
* 秘密是盲打输入的。没有敏感信息会回显到你的滚动历史中。
* 每个阶段都适配一屏。你仍然需要的内容没有滚出屏幕。
* Ctrl-C 并重新运行会从你离开的地方继续，并以已保存的值作为默认值提供。
* 最后一屏列出了它写入的内容，并分别列出了它无法完成的内容，你需要手动完成。

## 在系统中的位置

`wizard` is a reach-for-it-anytime standalone, sitting at the line where automation stops and a human has to click. Its nearest neighbour is [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills), because both exist to get a repo into a working state — that one configures this skill set, while `wizard` generates a setup path for everything else. It also pairs with [implement](https://aihero.dev/skills-implement): when a build lands a feature that needs credentials or a manual cutover, a wizard is how the human half gets done. When you're unsure which skill fits the moment, [ask-matt](https://aihero.dev/skills-ask-matt) routes you.
