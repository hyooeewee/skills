# 工单追踪器

> 2026年10月调查，不保证是最新的。用它来了解大概情况，然后自己研究当前的选项：产品、价格、限制和测试版变化很快。

工单追踪器是工厂的人机界面。下面的每种类型都是一个起点：阅读它是什么以及工厂何时选择它，然后对照[进一步研究](#researching-further)中的问题检查示例。

## 类型

### 1. Git 托管追踪器

内置于 Git 托管平台的追踪器，因此工单、分支和 PR 共享同一个产品、同一个权限模型和同一个身份。当工厂规模较小、团队已经在 Git 托管平台上工作，或者仓库是公开时，工厂会选择它。这是“全在 GitHub”的设计：Git 托管平台的 CI 无需胶水代码即可填补编排器的空位。标签是可靠的路由器，因为状态位于单独的看板上。外部开发者在代码所在地提交工单；非开发者需要账号，所以反馈按钮通过 API 提交工单。自托管的 Git 托管平台自带追踪器。

* [GitHub Issues](https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents)：默认选择；支持将工单分配给 Copilot、Claude 和 Codex。
* [GitLab Issues](https://about.gitlab.com/blog/how-to-use-a-work-item-created-trigger/)：由工作项触发的 Duo Agent Platform 流程。
* [Forgejo](https://forgejo.org/2026-10-release-v17-0/)：自托管，兼容 Actions。

需验证：哪些分配给代理的功能仍处于预览阶段，哪些计划包含这些功能；原生代理是读取后续评论还是仅在分配时读取工单（截至 2026-10，Copilot 仅在分配时读取）。

### 2. 代理原生开发追踪器

一个面向软件团队的独立 SaaS 追踪器，将代理视为一流的受托人，拥有代理 API 和自己的编码代理。当产品、支持或设计人员安排工作、团队已经在使用它、或需要精良的分类和规划视图时，工厂会选择它。“Linear 工单”设计。状态和工作流是一流公民，因此“移动到 Ready”是一个干净的触发器。代理作为应用用户出现，人类委派给它们，运行可以将进度流式传输到工单中。这里的原生代理是所有类型中集成最深的，因此一个代理可以部分填补编排器和运行器的空位。

* [Linear](https://linear.app/developers/agents)：最像代理的追踪器。
* [Jira](https://support.atlassian.com/jira-software-cloud/docs/collaborate-on-work-items-with-ai-agents/)：状态触发的代理；适合已经在使用 Atlassian 的组织。
* [Shortcut](https://www.shortcut.com/help/integrations/using-ai-agents)：分配一个故事或在评论中提及代理。
* [YouTrack](https://www.jetbrains.com/youtrack/)：代码即工作流；可自托管。

需验证：代理 API 状态（截至 2026-10，Linear 处于开发者预览阶段）；原生代理运行如何计量和定价；代理席位成本；代理 webhook 确认截止时间；哪个计划解锁面向客户的表单；自托管版本的生命周期结束日期。

### 3. 开源开发追踪器

一个 Linear 风格的追踪器，可运行在自己的基础设施上，旁边还有一个云版本。当数据必须留在内部（气隙隔离、受监管环境）或用户想要阅读和修补追踪器源码同时仍购买它时，工厂会选择它。通过状态、标签和自定义属性进行路由的工作方式与类型 2 非常相似；原生代理较新。

* [Plane](https://developers.plane.so/dev-tools/agents/overview)：类 Linear；每个应用有机器人用户；代理在评论中回复。
* [Huly](https://github.com/hcengineering/huly-selfhost)：带有双向 GitHub 同步的一体化工作空间；运维堆栈较重。
* [OpenProject](https://www.openproject.org/)：成熟、经典的项目管理。

需验证：代理功能状态（截至 2026-10，Plane 处于 beta 阶段）；哪个版本包含访客角色和入口表单；许可证；自托管堆栈的运维成本。

### 4. 通用工作工具

一个工作管理或文档工具，其数据库或任务列表兼作追踪器。当它服务于已经在使用它的非工程团队，或者工单主要是内容、运维或研究而非代码时，工厂会选择它。表单是强项：任何有链接的人都能提交工单。路由是灵活的数据库属性，因此每个工厂都发明自己的模式。开发者上下文（差异、PR）在 Git 托管平台上只需点击一下即可查看。仅限 SaaS。

* [Notion](https://www.notion.com/help/custom-agents)：数据库即追踪器；代理在数据库变更时触发。
* [Asana](https://asana.com/product/ai/ai-teammates)：AI 队友接手分配的任务。
* [ClickUp](https://clickup.com/brain/cloud-agents)：云代理在任务线程中返回 PR。
* [Trello](https://developer.atlassian.com/cloud/trello/guides/rest-api/webhooks/)：看板和列表；webhook 加 Butler 自动化。

需验证：webhook 延迟和粒度（截至 2026-10，Notion 批量处理内容更新，延迟一两分钟才到达）；哪个计划包含代理。

### 5. 仓库原生追踪器

工单作为文件存储在仓库中（JSONL、Markdown），通过 CLI 或 MCP 服务器由代理编辑，像代码一样合并。当一名开发者在本地驱动代理并希望计划在 git 中，或作为真实追踪器旁边的运行工作记忆时，工厂会选择它。推送到 Git 托管平台是唯一触发器，因此编排器通过 diff 文件来发现变化。身份是 Git 托管平台的。外部提交是 commit 或 PR，是所有类型中最弱的，因此一旦其他人提交工单，工厂就会升级到 Git 托管追踪器。

* [Beads](https://github.com/steveyegge/beads)：带有依赖跟踪的 JSONL，因此运行会挑选下一个未被阻塞的工单。
* [Backlog.md](https://github.com/MrLesk/Backlog.md)：带有看板视图和 MCP 服务器的 Markdown 任务。

### 6. 自建

工厂自己的应用在自己的数据库中存储工单，通常在编排器旁边。“自建追踪器”设计。当追踪器*就是*产品（内部平台、面向客户的支持工厂）或没有现成模型适用时，工厂会选择它。触发器变成函数调用；下面问题中的每个特性都变成要编写和维护的代码，这就是为什么技能倾向于购买。带有 webhook 的 [Airtable](https://airtable.com/developers/web/api/webhooks-overview) 基础是一个半成品的中间方案。

## 进一步研究

根据这些问题检查每个候选者。它们决定槽位：

* **触发器**：哪些工单变更（创建、标记、状态移动、分配、评论、提及）会发出 webhook 或事件，编排器可将其转化为运行？速度和粒度如何？
* **路由**：运行可以读取哪些标签、状态或字段来知道其类型（探索、实现、返工）？
* **线程**：人类能否在评论中回复代理，该回复是否会触发下一次运行？
* **身份**：是否有 API 加上机器人身份，以便运行以自己的身份行事，人类能区分代理和同事？机器人是否占用席位？
* **原生代理**：工单能否直接分配给代理？哪些代理，在哪个计划上，如何计量？它读取什么：分配时的工单，还是整个线程？
* **外部提交**：工单来源或客户如何创建工单：表单、邮件、聊天、API？哪个计划解锁它？
* **托管**：仅限 SaaS，还是自托管，以及什么许可证？

答案存在于主要来源中：API 和 webhook 参考（事件列表、载荷、投递和重试规则）、代理或应用开发者文档、定价页面和计划对比表、以及变更日志或发布说明。优先参考这些，而不是博客文章和对比网站。

最陈旧的优先：代理功能及其预览或 beta 状态、代理计量和信用价格、计划对代理和表单的限制、速率限制、以及自托管版本的收购、关闭和生命周期结束日期。Webhook 事件列表和路由模型变化缓慢。

## 持久陷阱

在依赖之前验证每个是否仍然成立：

* **GitHub 的 `GITHUB_TOKEN` 陷阱**：使用默认 `GITHUB_TOKEN` 进行的更改不会触发新的工作流运行。一个在工单上打标签或评论以触发下一次运行的运行，必须通过 GitHub App token 进行。
* **分配时上下文**：某些原生代理仅在分配时读取工单，因此后续上下文会放在 PR 上。
* **代理 webhook 期望快速确认**：编排器立即确认，然后启动运行。
* **通用工作工具中的批量 webhook** 到达较晚；需要及时运行的工厂会在属性变更时触发，或选择另一种类型。
