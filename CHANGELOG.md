# mattpocock-skills

## 1.2.3

### 补丁更改

* 使 `diagnosing-bugs` 脱敏敏感信息。

  * 在 `SKILL.md` 中添加一个 **Redact**（脱敏）部分。该技能让代理显示命令、输出和捕获的工件；该部分使脱敏成为每一步的第一步 —— 写入 `<REDACTED>`，针对环境变量构建循环，以便凭据保持在环境中，并且只引用捕获工件中携带信号的行。
  * 第一阶段完成标准说的是 "粘贴调用及其输出"。现在说的是以脱敏方式显示，并且第一阶段要求用户提供一个 **redacted**（脱敏）捕获工件。
  * 注意 `scripts/hitl-loop.template.sh` 中 `capture` 会将其值打印回终端，因此在登录时，它作为 `step` 进行，而获取观察（takes observations）继续进行。

* 从 `code-review`、`codebase-design` 和 `improve-codebase-architecture` 中的子代理分发指令中移除 Claude Code 的工具和代理类型名称，以便该步骤在 Codex 和其他工具上可跟随。

* 向导：移除时间估算。模板移除了 `TOTAL_MINUTES` 和剩余时间显示，`stage` 仅接受一个名称，进度按阶段计算。

## 1.2.2

### 补丁更改

* 再次使 `writing-for-agents` 在 Codex 中可由模型调用。

  * 从 `agents/openai.yaml` 中删除 `policy.allow_implicit_invocation: false`。Codex 将该技能从模型可见的技能列表中过滤掉了，因此其描述无法触发它 —— 只有显式的 `$writing-for-agents` 提及才有效。
  * 更新过时的 `interface.display_name` 和 `interface.short_description`，它们仍然命名旧的 `writing-great-skills` 技能。
  * 将技能从 `README.md` 和 `skills/productivity/README.md` 中的 **User-invoked**（用户调用）列表移动到 **Model-invoked**（模型调用）列表。

## 1.2.0

### 次要更改

* 在每个技能的 Claude Code 前置元数据旁添加 Codex 元数据，以便该集合在两个 harness 中都能正常工作，无需生成副本。

  * 在每个 `SKILL.md` 旁边添加一个 `agents/openai.yaml`，其中包含 Codex UI 元数据 (`interface.display_name`, `interface.short_description`)。
  * 为每个用户调用的技能标记 `policy.allow_implicit_invocation: false`，这是 Codex 对应 `disable-model-invocation: true` 的配置，这样 Codex 就会将其从隐式调用中排除，而显式的 `$skill` 调用仍然有效。
  * 在 `.agents/invocation.md`、`CLAUDE.md` 和 promoted-bucket READMEs 中记录双 harness 调用模型。
  * 将 `AGENTS.md` 添加为指向 `CLAUDE.md` 的符号链接，以便 Codex 读取相同的仓库指令。

* 将 **`to-questionnaire`** 从 `in-progress/` 移至 **Productivity** bucket，以便它在插件中发布。它将你无法独自回答的决定转化为 Markdown 问卷，供能回答的那个人填写——异步填写，或在会议中一起处理。

  它的标志性动作是它询问的是 **send**，而不是主题：正常的审问会审问话题，而这正是你在这里无法回答的，所以访谈只问问卷发给谁以及你需要什么回来，然后将每个问题都针对两者之间的差距。

  现在作为 promoted skill 连接起来 —— 插件入口，**User-invoked** 下的顶层 + Productivity READMEs，`docs/productivity/to-questionnaire.md` 上的文档页面，以及 `ask-matt` 中的 Standalone 路由，将其描述为 `/grill-me` 的逆操作（为别人挖掘，而不是你自己）。

* 将 **`wizard`** 从 `in-progress/` 移至 **Engineering** bucket，以便它在插件中发布 —— 并使其成为模型调用的。它生成一个交互式 bash 脚本，引导人类完成手动流程 —— 第三方设置、一次性迁移、A→B 状态转换 —— 打开每个 URL，说出要点击什么，捕获值，并将它们写入 `.env` 文件和 GitHub Actions secrets。

  令人愉悦的 UX 由捆绑的 `template.sh` 预先解决（带剩余时间的进度、确认关卡、跨平台 URL 打开包括 WSL、隐藏秘密输入、幂等的 `.env` 上传、`gh secret`/`gh variable` 写入并优雅降级、关闭跳过摘要）。`STAGES` 标记上方的一切都是固定的库，永远不会手动编辑 —— 技能的工作只是定义流程并编写它的 **stages**。

  Engineering 而非 Productivity：它读取 `.env*`、`docker-compose*`、framework 配置和 `.github/workflows/` 中所有的 `secrets.*`/`vars.*` 引用来界定自身范围，写入 CI secrets，并使用 `bash -n` 和 `shellcheck` 验证其输出。

  因为它是模型调用的，代理可以在遇到只有人类才能执行的步骤时立即使用它，而不是将编号的指令堆砌到聊天中并希望你遵循它们。输入 `/wizard` 的效果与以前完全一样 —— 模型调用只是*增加*了代理的可达范围。描述被写成决定何时触发的指针：它产生什么，四个触发分支（基础设施供应、设置凭据或 CI secrets、浏览陌生的第三方仪表板、一次性迁移或切换），以及一个明确的非触发 —— 不要为代理可以自己执行的步骤调用它。代理能做的工作，代理应该做；wizard 是用于点击、批准和仪表板行程，你不会交给代理的。写入一行之前的阶段列表确认现在在代理在构建中途触发它时充当提案。

  现在作为 promoted skill 连接起来 —— 插件入口，**Model-invoked** 下的顶层 + Engineering READMEs，`docs/engineering/wizard.md` 上的文档页面，以及 `ask-matt` 中只有人类才能采取的步骤的 Standalone 路由。模型调用还使其无法触及 [#693](https://github.com/mattpocock/skills/issues/693)，该问题从 Claude 桌面和 Web 界面的列表中删除了用户调用的技能。

* 围绕两个想法重塑 **`prototype`** 技能：demo 是 **一个可分享的 HTML 文件**，而 prototype 是 **一个主要来源**。

  逻辑分支现在产生一个自包含文件（纯 HTML/CSS/JS，无构建，无服务器）而不是终端应用 —— 非开发者可以双击打开它并在自己的领域语言中操作它：标记的状态面板、始终可用的自由操作按钮，以及一组标签页式的 **guided walkthroughs**，每个都是一个场景，下面有序的按钮。可移植的纯逻辑模块仍然转换到真实代码；HTML 外壳是一次性使用的。

  Throwaway 不再意味着删除。与其在回答问题后删除，不如将 prototype 捕获为可运行的证据，放在主分支之外的 throwaway branch (`prototype/<name>`) 上，并在实现问题上留下指向它的上下文指针 —— 这样主分支只保留验证过的决定，而探索仍然可被发现。答案（裁决 + 问题）仍然在 issue/ADR/commit 中持久捕获。

* 作为原生 **Claude Code plugin** 发布技能集，列在 Claude Code 的官方 marketplace 中。你现在可以订阅 promoted skills 作为托管、只读的 bundle，而不是复制可编辑的文件：

  ```bash
  claude plugins install mattpocock-skills
  ```

  或者，在会话内部：

  ```
  /plugin install mattpocock-skills
  ```

  不需要先添加 marketplace —— 官方 marketplace 默认已配置。

  `.claude-plugin/plugin.json` 携带完整的插件元数据（版本、描述、作者、许可证、关键词）和 promoted skills 的明确列表。`skills.sh` 仍然是通用安装程序（也是 Codex 和其他 harness 今天的路径）；原生 Codex plugin 被推迟 —— 见 `.agents/adr/0002-ship-as-a-claude-code-plugin.md` 了解原因。

* 添加 **`wait-what`** —— 一个用于纠正模型冗长的一个词。当消息没有到达的瞬间输入它，代理会重新提出它：一点上下文、ASD-STE100 简明技术英语，以及你 `CONTEXT.md` 中的通用语言。用户调用的，三行长。

  机制就是名字。简洁技能通过增长而失败 —— 400 行的技能仍然让模型冗长 —— 所以这个只有一个精确的引导词，其他什么都没有。描述*输出*的名字（`/tldr`, `/no-fluff`）让模型截断单词并进一步失去你；命名*监听者*的状态要求同时得到两部分，更少的单词**和**你缺失的上下文。它还重用了你全局 `CLAUDE.md` 中已经存在的引导词，所以技能、`CLAUDE.md` 和每个 `CONTEXT.md` 都使用相同的 token。

  它修复一条消息；它不会阻止下一条。行话的解药是用 `/grill-with-docs` 预先建立的共享语言；当你还没有的时候，这就是你寻求的东西。

* 将 `/wayfinder` 单元命名为一个 **decision ticket**，并用子代理解决研究票据。

  人们一直将 wayfinder ticket 读作普通的 *implementation* ticket —— 一段要执行的构建切片 —— 当 wayfinder 将它们用作 **decision tickets**：解决结果是决策的问题。技能描述和它的开头一句现在引入了这个术语（并说明它是什么），`ask-matt` / engineering README 简介和文档页面与之匹配 —— 当术语建立后，"ticket" 仍然是日常词汇。`CONTEXT.md` 将 **Decision ticket** 记录为领域术语，所以 "avoid: ticket" 指导不再与 wayfinder 故意使用该词相矛盾。

  研究票据不再为单独启动的会话搁置。研究仍然是一个真实的票据类型 —— 它是一个真正的共同阻碍，下游决策依赖于它，而这种依赖正是前沿的阻塞边存在的渲染。变化的是它是如何解决的：因为研究是离线的，绘图不会停止并读取它。创建票据后，绘图会话为每个研究票据触发一个 `/research` 子代理并行解决它，在 throwaway `research/<name>` 分支上捕获发现，并带有上下文指针。研究票据是 *一个会话一个票据* 的唯一例外。

* 破坏性变更：重命名 **`writing-great-skills`** → **`writing-for-agents`**，重构它，并添加一个新的引导词。

  该参考现在涵盖代理消费的任何文档 —— 技能、`AGENTS.md` / `CLAUDE.md`，通过指针到达的文档 —— 不仅仅是技能。`GLOSSARY.md` 合并到 `SKILL.md`（每个术语一个权威处理；`_Avoid_` 同义词列表和独立的预测性定义消失了）；仅技能的机制（前置元数据、模型 vs 用户调用、路由技能、调用的切割）被披露到新的 `SKILL-MECHANICS.md`。该技能现在是 **model-invoked**：它在创建或编辑技能或修改 `AGENTS.md`/`CLAUDE.md` 时触发。`ask-matt` 的指针已更新。在新名称下重新安装；旧名称消失了（没有别名）。

  剪枝部分增加了 **cache**。单一真理来源现在延伸到文档之外进入环境 —— `package.json` 脚本、配置文件、目录布局、`--help` 输出本身就是权威的，所以重述它们的文档是一个查找的缓存，只在查找昂贵时加载。积极目标：缓存代理无法通过查找找到的东西（未写成的惯例、选择背后的原因、配置不承认的陷阱），并将单文件、单命令查找留给环境，在那里它们不会过时。

* 在 **`improve-codebase-architecture`** 技能的 Explore 步骤中添加一个 YAGNI 范围过滤器。与其均匀扫描整个 repo，现在它将范围限定在更改实际落地的地方：如果你指定一个方向，它就接受它，否则它读取最后 \~20 条提交消息以将探索偏向活跃开发路径。代码中无人触及的深化机会是你永远无法兑现的杠杆 —— 杠杆只在持续编辑时生效 —— 所以报告停止整理 repo 的休眠角落。

### 补丁更改

* [#763](https://github.com/mattpocock/skills/pull/763) [`77d207e`](https://github.com/mattpocock/skills/commit/77d207ef03219cc603e2832e1159cbdd1c91818e) 感谢 [@mattpocock](https://github.com/mattpocock)! - 优化 `/ask-matt` — 路由器现在涵盖了阶段边界、两个寻路错误以及它从未提及的两个技能。

  **阶段边界。** 一个**阶段**是会话中的一项工作——包括质询、实现和 QA（质量保证）——两个阶段之间的边界是你决定如何处理已构建上下文的地方。原本的两点 `Crossing sessions`（跨越会话）部分被一个决策树所取代，该决策树按顺序包含了所有五个选项（**continue**、`/clear`、`/handoff`、**subagent**、`/compact`），理由在新的 `PHASE-BOUNDARIES.md` 中披露。随之而来的是三个修复：

  * **`/handoff` 被过度宣传了。** 它被读作上下文窗口之间的通用桥梁。它是狭窄的：你只需要在必须将某物*转移*时使用它——例如新的工具集、新的目录、同事，或在阶段中途分叉的侧边任务。它带来的好处是可移植性。
  * **`/compact` 是默认选项，而不是首选的。** 它位于树的底部，位于其上方四个更便宜或更精确的问题之后。从那里开始会产生一个会话，它会自信地误解摘要所平铺的任何内容。
  * **两个分支完全缺失。** **Continue** 是首先需要排除的选项——它是唯一一种保持对话作为主要来源而不是摘要的移动方式——而 **subagent** 处理任何范围足够紧密以至于可以离线运行的任何内容。

  上下文卫生的退出点现在显示为 `/compact` 而不是 `/handoff`（相同的工具集、相同的目录、处于边界——不适用 `handoff` 条款），智能区域数字已从 \~120k 更新到 \~150k tokens。

  **寻路路由。** 人们在使用最繁重、认知负荷最大的流程时最常犯的两个错误：

  * **过度使用它。** 它比单一的质询更慢、更密集，因此被标记为最繁重的流程，并保留给那些确实无法在一个会话中容纳的想法——范围良好的功能属于 `/grill-with-docs`，而不是这里。
  * **在交接时迷路。** 当地图清除时，寻路器进行交接，它不构建：在 `/to-spec` 合并到主流程中（这将地图的关联决策折叠成可构建的计划），而不是直接将地图循环到 `/implement`。直接到 `/implement` 仅适用于结果真正较小的努力。

  **缺失的路由。** `/grilling` 和 `/resolving-merge-conflicts` 完全缺失于路由器中，现在已被添加，并且 `grill-me` 根据你是否在工作目录中与 `grill-with-docs` 分离。

* [#502](https://github.com/mattpocock/skills/pull/502) [`44eed54`](https://github.com/mattpocock/skills/commit/44eed545186ffd0263e8004867750b80cfddd215) 感谢 [@mattpocock](https://github.com/mattpocock)! - 使 `/setup-matt-pocock-skills` 更友好，并将本地 Markdown 追踪器与当前规范对齐。

  * **分类标签** 现在仅在安装了 `triage` 技能时才会被询问，并且作为一个单一的推荐“是”的问题（“保留默认分类标签？”）而不是强制审问。当未安装 `triage` 时，该部分——以及 `docs/agents/triage-labels.md`——将被跳过。
  * **将外部 PR 作为请求面** 不再是设置问题。GitHub/GitLab 模板仍然携带该标志，默认为关闭；用户稍后可以在 `docs/agents/issue-tracker.md` 中将其切换。
  * **领域文档** 默认为单上下文而不询问；只有在仓库显示单体仓库信号时才提供多上下文。
  * **本地 Markdown 工单** 现在每个工单一个文件，位于 `.scratch/<feature>/issues/<NN>-<slug>.md` 下——绝不是一个单独的合并 `tickets.md`。`/to-tickets` 和本地工单追踪器模板现在达成一致，规范文件是 `spec.md`（而不是 `PRD.md`）以匹配 `/to-spec`。

  `setup-matt-pocock-skills` 和 `to-tickets` 的文档页面已重新同步。

* [#532](https://github.com/mattpocock/skills/pull/532) [`170ad48`](https://github.com/mattpocock/skills/commit/170ad48655825783d0193e850e31a9aac957bb95) 感谢 [@mattpocock](https://github.com/mattpocock)! - 为通用使用重写 **`grilling`**。其描述和正文不再将面试限制在软件计划上：“this plan” → “this”，“enact the plan” → “act on it”，“exploring the codebase” → “exploring the environment”。技术保持不变；现在它读起来像是对任何计划、决策或想法的压力测试。

* [#593](https://github.com/mattpocock/skills/pull/593) [`a4b2009`](https://github.com/mattpocock/skills/commit/a4b2009a1a3ac9575506c10b4c84f08f9bba7a38) 感谢 [@mattpocock](https://github.com/mattpocock)! - 将 **`grilling`** 从逐题提问重制为轮次制。它现在映射决策树并询问整个**前沿**——即所有前提条件已满足的问题——在一个编号轮次中，然后根据用户的答案重新计算前沿并询问下一轮。相同的 13 个问题出现在 \~3 轮中而不是 13 轮。环境可以回答的事实被分派给后台子代理，因此研究永远不会阻塞轮次：只有运行中的探索下游的问题才会等待它。当前沿为空时，会话结束。

  每轮中的每个问题都以一种固定的形状发出——`❓ **Q1** - **<title>**`，然后是正文（散文或多项选择），然后是推荐意见，单独一行 `➡️`。一轮读起来像一个可扫描的编号列表，每个推荐意见在视觉上与问题分开，因此你可以通过数字回答而不是引用问题。

  `grill-me`、`grill-with-docs` 和 `triage` 也一次运行一轮前沿——`triage` 的质询步骤和 `grilling` 的 Codex `short_description` 现在说明这一点，而不是描述旧节奏。逐题提问的退出选项（你全局 `CLAUDE.md` 中的一行）未变。

* [#752](https://github.com/mattpocock/skills/pull/752) [`c66bdee`](https://github.com/mattpocock/skills/commit/c66bdeeee002d81e3f8b21403c07f9a0d7bea6da) 感谢 [@mattpocock](https://github.com/mattpocock)! - 从仓库中移除六个技能。它们都不在 Claude Code 插件中，但都可以通过 [skills.sh](https://skills.sh/mattpocock/skills) 安装，该服务提供仓库中的每个技能——这就是离开该列表的原因，以及它们各自的去向。

  四个已退休的技能，每个都被一个做得更好的技能吸收了：

  * **`ubiquitous-language`** → **`/domain-modeling`**，它构建和维护整个领域模型，而不是从一次对话中倾倒词汇表。
  * **`design-an-interface`** → **`/codebase-design`**。没有任何丢失：“设计两次”技术——由 Ousterhout 提出的并行子代理生成截然不同的设计——作为 `DESIGN-IT-TWICE.md` 装载在该技能中。
  * **`qa`** → **`/triage`** 和 **`/to-tickets`**。
  * **`request-refactor-plan`** → **`/to-spec`** 和 **`/improve-codebase-architecture`**。

  还有两个只属于我的——绑定在我的机器上，从未打算给其他人。`personal/` 桶随它们而去：

  * **`edit-article`**
  * **`obsidian-vault`**，它硬编码了我自己的 Obsidian 仓库的路径。

  `skills/deprecated/` 保持为一个桶，现在为空。`skills/in-progress/` 未变，现在被描述为它实际的样子：一个发布出来的 beta 通道，可以通过 skills.sh 一次安装一个技能。

* [#734](https://github.com/mattpocock/skills/pull/734) [`a2f9333`](https://github.com/mattpocock/skills/commit/a2f9333669ff53db762c87ecda5a15442060a3be) 感谢 [@mattpocock](https://github.com/mattpocock)! - 完成 `to-prd` → `to-spec` 的重命名：“spec”现在是已发布文本中唯一的术语。

  * **`to-spec`** 不再以“你可能知道这个文档是 PRD”开头——括号内容已从技能及其文档页面中删除。本地 Markdown 追踪器模板删除了相同的模糊说法。
  * **`code-review`** 在其前置元数据描述、双轴摘要和规范源搜索顺序中谈论的是原始问题/规范，而不是问题/PRD。两个 README 已重新同步。
  * **GitHub 和 GitLab 追踪器模板** 现在说“此仓库的问题和规范以 GitHub/GitLab 问题形式存在”——当本地模板更新时，它们被留在了“PRDs”上，因此过时的术语传播到了它们所写入的每个仓库中。
  * **`docs/engineering/research.md`** 指向 `https://aihero.dev/skills-to-prd`，这是重命名技能的无效 slug；它现在像其他十九个文档页面一样链接到 `to-spec`。

  CHANGELOG 和现有的 changesets 仍然命名 PRD，用于记录重命名本身，这是正确的。

## 1.1.0

### 次要更改

* [#406](https://github.com/mattpocock/skills/pull/406) [`930a450`](https://github.com/mattpocock/skills/commit/930a450089f77a49af09001d955db8452a4b867d) 感谢 [@mattpocock](https://github.com/mattpocock)! - 将 **`ask-matt`** 路由器更新至完整的技能集。它现在映射了它缺失的五个技能：**`tdd`**（作为驱动 `implement` 的红绿引擎融入主流程中），**`diagnosing-bugs`**（一个新的“某处坏了”的入口——之前没有针对 bug 的路由），**`domain-modeling`** 和 **`codebase-design`**（一个新的“底层的词汇”部分），以及 **`grilling`**（共享的面试原语）。`prototype` 被充实为独立技能，描述从“用户调用的技能”扩展为“技能”。在 `CLAUDE.md` 中添加了一条维护规则，以便任何未来的技能添加/重命名/删除或流程变更都会触发 `ask-matt` 重新检查，除了现有的文档页面同步规则。

* [#464](https://github.com/mattpocock/skills/pull/464) [`639df6e`](https://github.com/mattpocock/skills/commit/639df6e7386dfddc739b2aecdeff37a876f2483b) 感谢 [@mattpocock](https://github.com/mattpocock)! - 提升 **`code-review`** 并使其稳健。进行中的 **`review`** 技能重命名为 **`code-review`** 并从 `in-progress/` 移至 `engineering/`：它现在随插件分发，列在顶级和工程 README 中（模型调用），并在 `docs/engineering/code-review.md` 有文档页面。`/implement` 技能和文档指向 `/code-review`。

  它还在其标准轴上获得了一个始终开启的 **Fowler 坏味道基线**——精心策划的约 12 个高信号“代码坏味道”（神秘的名称、重复代码、特征依恋、数据聚集、基本类型偏执、重复的 switch 语句、枪炮手术法、发散式变化、投机泛化、消息链、中间人、拒绝被继承），内联到 `SKILL.md` 中作为固定基线，与仓库文档的任何内容并存，而不是一个新的第三轴。两条绑定规则使其安全：文档化的仓库标准覆盖基线，每种坏味道都作为判断而非硬性违规报告。

* [#464](https://github.com/mattpocock/skills/pull/464) [`639df6e`](https://github.com/mattpocock/skills/commit/639df6e7386dfddc739b2aecdeff37a876f2483b) 感谢 [@mattpocock](https://github.com/mattpocock)! - 在两个方面完善 **`grilling`**。

  **一个确认门控。** 代理不会执行该计划，直到你确认已达成共同理解——将技能现有的“共同理解”完成标准转变为显式的停止门控。`description` 还招募了预训练的 **`grill`** 引导词（"Relentlessly grill the user"）来 sharpen 调用，并同步文档页面。

  **事实 vs. 决策。** Grilling 现在将 *facts*（查找它们——探索代码库）与 *decisions*（把每一个交给人类并等待他们的回答）分开。旧的笼统说法——"如果一个问题可以通过探索代码库来回答，那就探索代码库"——是为实时人类情况编写的，但一旦另一个技能在解决工单的框架内运行 grilling，它就会被解读为自主回答 *decisions* 的许可。将两者分开可以防止 grilling 代理抢先回答自己的问题。

* [#463](https://github.com/mattpocock/skills/pull/463) [`af6d692`](https://github.com/mattpocock/skills/commit/af6d6922c3e2b5288eef155346cbe319e4ed3bd0) 感谢 [@mattpocock](https://github.com/mattpocock)! - 向 **`writing-great-skills`** 添加两个相邻的 Steering 失败模式，都关于你认为是“关闭”的语言仍然 steering 代理。**Negation** —— *elephant* —— 是通过禁止进行 steering：命名 *not* 要做什么将禁止行为带入上下文并使其 *more* 可用，而不是更少（"don't think of an elephant"），所以治愈方法是提示 **positive**。**Negative Space** —— void —— 是对你遗漏的东西所进行的 steering 的盲目：技能拒绝的每一个决定都被委托给代理的先验而不是保持中立，所以治愈方法是阅读草稿以发现其静默并有意决定每一次遗漏（填补它，或将其保持开放作为一个真正的 **branch**）。保留为两个条目，而不是一个——它们携带不同的诊断和不同的治愈方法——每个都是完整的 `GLOSSARY.md` 条目加上 `SKILL.md` 失败模式要点，与其他失败模式的方式相匹配。

* [`850873c`](https://github.com/mattpocock/skills/commit/850873cd73d5f81826ebf512ad35d2b1e113001f) 感谢 [@mattpocock](https://github.com/mattpocock)! - 使 **`prototype`** 技能模型调用，以便代理可以自主使用它（其他技能也可以）。其描述围绕引导词 *prototype* 进行重写——回答设计问题的废弃代码——每个分支有一个触发器（状态/逻辑健全性检查，或 UI 探索）。

* [#409](https://github.com/mattpocock/skills/pull/409) [`0d74d01`](https://github.com/mattpocock/skills/commit/0d74d01cbc64ca27778a49b38599f70c534e76a0) 感谢 [@mattpocock](https://github.com/mattpocock)! - 添加 **`research`** 技能——一个小的、模型调用的技能，它启动一个 **background agent** 针对主要来源（官方文档、源代码、规格说明、第一方 API）调查问题，然后在仓库保存笔记的地方留下一个引用的 Markdown 文件。它是可委托的阅读苦力：你在工作时它阅读，并返回一个文档供 grill、计划或设计使用。列在顶级和工程 README 中（模型调用），添加到 `.claude-plugin/plugin.json`，在 `docs/engineering/research.md` 有文档页面，并在 `ask-matt` 中路由为独立技能。

* [#469](https://github.com/mattpocock/skills/pull/469) [`a0329ba`](https://github.com/mattpocock/skills/commit/a0329ba95751f58566ed7ab484475917a68f1629) 感谢 [@mattpocock](https://github.com/mattpocock)! - 将 **`to-issues`** 技能拆分为精简的 **Process** 和 **Reference** 部分，并教会它处理 **wide refactor**——一个单一机械变更（如重命名列），其 **blast radius** 风扇般扩散到整个代码库，瞬间破坏数千个调用点，因此没有垂直切片能变绿。起草步骤现在指向两个相邻的参考块：普通 tracer bullets 的 **Vertical slice rules**，以及 **Wide refactors**，它通过 **expand–contract** 切割变更（在旧形式旁边展开新形式，按 blast radius 大小的批次迁移调用点，然后收缩旧形式），以便 CI 保持逐批变绿——或者，当它不能时，只在最终的集成和验证问题上变绿。问题体模板也移入 Reference。

* [#464](https://github.com/mattpocock/skills/pull/464) [`386d4ff`](https://github.com/mattpocock/skills/commit/386d4ff719a7c420ad1454232d0436b01f1b8c17) 感谢 [@mattpocock](https://github.com/mattpocock)! - 统一规划技能。**`to-prd` 重命名为 `to-spec`**——"spec" 现在是唯一的贯穿术语（它仍然以"你可能知道这份文档为 PRD"开头以增加可发现性）。**`to-plan` 和 `to-issues` 合并为一个 `to-tickets` 技能，并删除 `to-issues`。**

  `to-tickets` 将计划、规格或对话拆分为一组 **tickets**——tracer-bullet 垂直切片，每个都声明其 **blocking edges**。该工件根据配置的 tracker `/setup-matt-pocock-skills` 以两种方式读取：**本地文件** (`tickets.md`) 将边写成文本，你手动从上到下处理它；**真实 tracker** 将它们写成原生阻塞链接，因此任何阻拦器都完成的 ticket 都在前沿，并且多个代理可以同时运行。无论如何边都存在于 ticket 中——介质只决定是否有任何东西并行处理它们。

  发布倾向于使用 tracker 的 **native sub-issues** 进行 parent → slice，以及使用 **native blocking edges** 进行 `Blocked by`（如果 tracker 支持的话），保持 `## Parent` / `## Blocked by` 正文部分作为后备。"What to build" 模板指向 `/prototype` 的代码所在位置，而不是内联其片段。

  `ask-matt` 的主流程现在路由 `idea → /to-spec → /to-tickets → /implement`，并且在 `docs/engineering/to-spec.md` 和 `docs/engineering/to-tickets.md` 有面向人类的文档页面。

* [#464](https://github.com/mattpocock/skills/pull/464) [`0557d57`](https://github.com/mattpocock/skills/commit/0557d57579d9b3d39839fdaf8d4a6542b17539ce) 感谢 [@mattpocock](https://github.com/mattpocock)! - 将 wayfinder 在文档中的位置确定为 **situational on-ramp**，而不是新的主入口流程——grill 领导的 *idea → ship* 链仍然是前门（将 wayfinder 冠以默认脊柱是 v2 级别的举动，不是 1.1）。**`ask-matt`** 路由器现在命名 wayfinder 的具体触发器——一个 greenfield 项目或一个巨大的功能构建，太大无法在一个会话中完成——以及两个 grill 前门（**`grill-me`**，**`grill-with-docs`**）指向 *up* 到 wayfinder 用于在一个会话中无法容纳的努力，以便入口可以从读者实际开始的地方被发现。

* [#464](https://github.com/mattpocock/skills/pull/464) [`639df6e`](https://github.com/mattpocock/skills/commit/639df6e7386dfddc739b2aecdeff37a876f2483b) 感谢 [@mattpocock](https://github.com/mattpocock)! - 毕业 (graduate) 并重构 **`wayfinder`**——规划大量工作的技能，超过一个代理会话可以容纳。它从 `in-progress/` 移至 `engineering/`（插件入口，顶级 + 工程 README 在 **User-invoked** 下，`docs/engineering/wayfinder.md` 有文档页面，以及 `ask-matt` 中的路由），作为成熟技能着陆。使其到达那里的重命名和重构：

  * **`decision-mapping` 重命名为 `wayfinder`**，调用为 `/wayfinder`。"Decision map" 是行话且不准确——只有一个 ticket 类型实际上是一个决策。重构绘制了一条通过模糊问题的路线，给出一个连贯的引导词框架——**fog of war**（战争迷雾）、**frontier**（前沿）、**the map**（地图）——而不是一个发明的术语叠加在上面。
  * **Destination 作为引导词。** 导航寻找通往目的地的 *way*；它不会冲上去构建它。命名目的地是绘图的第一步——它固定范围并塑造每个 ticket——所以地图获得一个 `## Destination` 字段供每次会话定向，且分诊在 ticket 存在之前将其固定。
  * **计划，而不是做。** 地图产生 **decisions，而不是 deliverables**；当在某人构建该事物之前没有什么需要决定时，它就完成了。一个 effort 可以在它的 Notes 中覆盖这一点。
  * **地图是索引，而不是存储。** 一个决定只存在于一个地方——它的 ticket——所以地图只进行摘要和链接，从不重述；将雾毕业到 ticket 会清除毕业的补丁，以便没有任何东西停留在两个地方。
  * **默认协作。** 地图从本地 Markdown 文件移到仓库的 issue tracker：一个单一的 `wayfinder:map` issue，其 tickets 是其子 issues——一个团队可以观看的共享 URL。会话以低分辨率加载地图并根据需要放大到 tickets。Wayfinder 保持 tracker-agnostic（GitHub, GitLab, local-markdown）通过 `docs/agents/issue-tracker.md` 中的指针，并且 `setup-matt-pocock-skills` 种植 "Wayfinding operations" 部分。
  * **通过分配而不是标签来 Claim。** 一个会话通过将 ticket 分配给驱动开发人员来 claim 它——被分配者 *就是* claim——将标签词汇解放为仅 `wayfinder:<type>`。
  * **Native blocking。** 阻塞倾向于 tracker 的原生依赖关系，它在 tracker 的 UI 中视觉化地渲染前沿，以便人类在不打开地图的情况下看到什么是可用的。GitHub 和 GitLab 模板阐明了原生配方，并带有正文约定后备。
  * **Fog vs. out of scope, split。** 两个简单命名的地图部分——`## Not yet specified`（随着前沿推进而毕业的 in-scope fog）和 `## Out of scope`（被判定在目的地之外的 work，关闭，从不毕业）——以便 destination 以外的 work 不再被读取为可用的前沿。
  * **第四个 `task` ticket 类型。** 用于阻止决策的实际手动工作（配置访问权限、移动数据、注册服务）——唯一 *做* 而不是决定的类型，通过解除决策阻塞赢得其位置。
  * **HITL / AFK ticket classification。** 每个 ticket 类型都是 **HITL**（human in the loop——grilling, prototype）或 **AFK**（agent alone——research；task 是两者之一）。一个 HITL ticket 只能通过实时交换解决，所以"wait for the human" 从标签中脱落——一个回答自己问题的 grilling 代理，根据定义，打破了 HITL。（这修复了学生关于 `/wayfinder` grilling *itself* 而不是 human 的报告。）
  * **No-fog early exit restored。** 如果打开的广度优先 grilling 没有发现 fog，旅程足够小适合一个会话——所以它停止并询问你希望如何进行，而不是构建一个不需要的地图。

### 补丁更改

* 将 **`tdd`** 重塑为仅参考技能，并添加一个缺失的反模式。

  **仅参考。** 红色 → 绿色 → 重构循环由模型已持有的引导词锚定，因此逐步的 Workflow 大多是重述该循环。移除了 Workflow 和每周期检查清单；将其唯一持久化的想法——垂直切片/追踪子弹——折叠到反模式部分和一个简短的循环规则列表中。引入 **seam** 作为测试位置的引导词：仅在预商定的 seam 处进行测试，并在编写任何测试之前与用户确认。同时移除了重构阶段 —— TDD 现在是红色 → 绿色；重构属于审查阶段，因此重构规则和 `refactoring.md` 被移出（其家在 `code-review` 中）。

  **同义测试。** 添加了同义测试反模式：一种断言以代码计算断言的方式重新计算的测试，通过构造必然通过且无法提供信心——与已涵盖的实现耦合反模式不同。作为同级的添加到了相同位置：一个哲学原则（期望值必须来自独立的真理源），一个检查清单门禁，以及 `tests.md` 中的 BAD/GOOD 示例对。

* 扩展 **`triage`** 技能以处理外部拉取请求，将 PR 视为附带代码的问题，这些代码运行相同的角色和状态机。PR 与问题一起内联流动（由每个仓库的设置开关控制），发现界面仅显示外部 PR，仅针对错误的“重现”步骤被概括为单一的“验证主张”步骤，冗余检查将已实现的请求解决为 `wontfix`，而不会污染超出范围的知识库。`setup-matt-pocock-skills` 获得了 PR 作为请求界面的开关以用于 GitHub/GitLab。

* 修复 **`wayfinder`** 硬编码问题跟踪器文档路径，这破坏了套件其余部分所依赖的间接引用。

  `to-issues`、`to-prd` 和 `triage` 从不指定路径——它们通过 `setup-matt-pocock-skills` 写入 `CLAUDE.md` / `AGENTS.md` 中的 `### Issue tracker` 块来解析跟踪器，该块指向跟踪器文档所在的任何位置。Wayfinder 代替的是固定了字面量 `docs/agents/issue-tracker.md`，因此在将代理文档保存在其他地方的仓库中，它默默回退到本地 markdown 跟踪器——即使是那些 `CLAUDE.md` 明确声明为 GitHub issues 的跟踪器。现在它通过同一个指针解析文档，并按名称读取其“寻路操作”部分，保持套件中的一致间接引用。

## 1.0.1

### 补丁更改

* 使 **`teach`** 技能优先考虑重用。课程现在从 `./assets/` 中的可重用 **组件** 构建——样式表、测验小部件、模拟器、图表辅助工具。重用是默认的：代理在编写课程之前先读取 `./assets/`，基于现有的内容构建，并将任何新的可重用内容提取为组件，而不是内联。

## 1.0.0

### 主要更改

* 添加 **`ask-matt`** 技能——一个用户调用的路由器，将你指向适合你情况的正确技能或流程。

  **破坏性更改：** `ask-matt` 路由到本仓库中的其他用户调用的技能，因此它期望它们已安装。

* 添加共享设计技能，并将现有技能重新连接到它们上面。

  * 新的 **`codebase-design`** 技能——深层模块词汇（module, interface, depth, seam, adapter）以及将大量行为隐藏在小型接口背后的原则。以前位于 `improve-codebase-architecture/LANGUAGE.md` 中的语言现在位于这里，经过泛化以便跨技能重用。
  * 新的 **`domain-modeling`** 技能——积极构建和完善项目的领域模型，对照术语表对术语进行压力测试，并保持 `CONTEXT.md` 和 ADRs 更新。
  * `improve-codebase-architecture` 现在从 `/codebase-design` 获取其架构词汇，从 `/domain-modeling` 获取其领域模型。
  * `tdd` 现在依赖 `/codebase-design` 获取接口设计指导——其内联的 `deep-modules.md` / `interface-design.md` 注释已被移除，取而代之的是共享技能。
  * `grill-with-docs` 现在通过 `/domain-modeling` 内联构建领域模型。

  **破坏性更改：** 这些技能现在依赖于新的 `codebase-design` / `domain-modeling` 技能，因此你也必须安装它们。

* 移除 **`caveman`** 和 **`zoom-out`** 技能。

  * `caveman` 是我正在测试的另一个技能的副本，从未打算公开。
  * `zoom-out` 在实践中未使用，因此已从仓库中移除。

  **破坏性更改：** 两个技能都已被移除。

* 将 **`diagnose`** 技能重命名为 **`diagnosing-bugs`**。

  **破坏性更改：** 将其作为 `/diagnosing-bugs` 调用——旧的 `/diagnose` 名称不再存在。

* 用 **`writing-great-skills`** 替换 **`write-a-skill`**。

  * 移除了 `write-a-skill`。
  * 添加了 `writing-great-skills`（及其 `GLOSSARY.md`）——编写和编辑技能的参考：使技能可预测的词汇和原则，将无操作（no-ops）逐句揪出来。
  * 将 `grilling` 暴露为模型调用的技能——`grill-me` 和 `grill-with-docs` 背后的可重用面试循环。

  **破坏性更改：** `write-a-skill` 已被移除；请改用 `writing-great-skills`。

### 次要更改

* 添加 **`resolving-merge-conflicts`** 技能——一个用于解决进行中的 git 合并或变基冲突的循环。独立运行，不依赖其他技能。

* 在文档中将技能分类法从 **命令 / 技能** 重命名为 **用户调用 / 模型调用**，并添加 `docs/invocation.md` 定义这种划分：用户调用的技能仅在输入时可达，用于编排；模型调用的技能在任务合适时也可自动达到。用户调用的技能可以调用模型调用的技能，但绝不能调用另一个用户调用的技能。

### 补丁更改

* 收紧 **`review`** 技能：快速失败的重构检查、单一来源的规则和无操作移除。
