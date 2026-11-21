# Git 托管服务

> 2026年10月调查，不保证是最新的。用它来了解大概情况，然后自己研究当前的选项：产品、价格、限制和测试版变化很快。

## 类型

### 一体化平台

Git 托管与工单追踪器、作为编排器的 CI 系统、托管运行器捆绑在一起，三者共享同一个事件流和同一个权限模型。一个产品同时填补 Git 托管、工单追踪、编排器和运行器四个槽位，其 CI 运行页面往往也覆盖了可观测平台。每个 PR、评审、标签和检查事件既是 Webhook 又是原生触发器，因此此类型拥有最丰富的触发器目录。

**适用场景**：工厂起步规模小、代码已托管于此，或用户想要一张账单、一次登录。这就是 [SKILL.md](SKILL.md) 中的“全 GitHub”设计。厂商也会提供自家的代理，它们要么与用户的运行竞争，要么作为其中一个运行器插槽。

* * [GitHub](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api)：默认选择；Issues、Actions、Apps 及最大的集成生态。
* * [GitLab](https://about.gitlab.com/pricing/)：Issues、合并请求、CI/CD 和运行器集于一身，提供 SaaS 或自托管。

### 套件 Git 托管

作为更广泛套件的一个模块出售的 Git 托管，工单追踪器是其同胞产品（Jira、Azure Boards），通过链接和 Webhook 连接。追踪器事件到达第二条流，因此编排器需汇合两个事件源。

**适用场景**：组织已在运行该套件，且人员活跃于 Jira 或 Azure Boards。工厂继承该追踪器作为其人机界面。

* * [Bitbucket Cloud](https://support.atlassian.com/bitbucket-cloud/docs/api-request-limits/)：Atlassian 的 Git 托管，搭配 Jira 和 Bitbucket Pipelines。
* * [Azure DevOps (Azure Repos)](https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rate-limits)：Repos、Boards、Pipelines 与 Entra 身份在同一组织内，因此填满四个槽位。
* * [AWS CodeCommit](https://aws.amazon.com/blogs/devops/aws-codecommit-returns-to-general-availability/)：仅提供 Git 托管和 IAM 身份；需搭配外部追踪器和 CI。

### 自托管 Git 托管

用户在自有基础设施上运行的 Git 托管，通常与编排器和运行器部署在同一网络。运营商自行设定速率限制，上限取决于硬件。用户需对其填补的每个槽位负责升级、备份和正常运行时间。

**适用场景**：代码必须留在本地或物理隔离环境、用户想要零人头成本、或 SaaS API 预算限制了繁忙工厂。Forgejo 和 Gitea 适合小团队，捆绑 Issues 与兼容 GitHub Actions 的 CI；GitLab 自托管和 GitHub Enterprise Server 适合组织，保留其 Apps 和服务账号。

* * [Forgejo](https://forgejo.org/2026-10-release-v17-0/)：社区治理、轻量级，带有 Forgejo Actions。
* * [Gitea](https://about.gitea.com/)：轻量级，带有 Gitea Actions；也提供托管版。
* * [GitLab 自托管](https://about.gitlab.com/install/) 和 [GitHub Enterprise Server](https://docs.github.com/en/enterprise-server@latest)：完整平台运行在用户自有基础设施上。

### 评审优先与去中心化 Git 托管

评审模型偏离分支与 PR 形态的 Git 托管：带投票标签的逐提交补丁集，或跨用户运行节点联邦化的仓库。大多数工厂工具假设使用 PR，因此预期需要构建更多胶水代码。在 Gerrit 中，自动化检查运行从机器人账号投出 `Verified` 票，人工投出 `Code-Review` 票。

**适用场景**：团队已采用此评审方式，或看重联邦化胜过生态。

* * [Gerrit](https://www.gerritcodereview.com/)：补丁集评审与标签投票；Android 和 Chromium 模式。
* * [Tangled](https://blog.tangled.org/ci/)：基于 AT Protocol 的 Git 托管，支持堆叠 PR 与自带 CI 运行器。

### 代理原生 Git 托管

以代理为主要作者构建的新型 Git 托管：高提交吞吐、以服务端读取替代克隆、按代理归因。整个类型尚早（截至 2026-10），门槛和速率限制可能未公开：请向厂商咨询。

**适用场景**：工厂对单一大型仓库运行大量并发任务，且克隆时间或 API 限制成为瓶颈。在单一仓库镜像上试点，保留 GitHub 作为回退。

* * [Cursor Origin](https://cursor.com/origin)：Graphite 团队出品，支持 GitHub 同步与签名的 PR、评审和推送 Webhook。
* * [GitLab 下一代 SCM](https://about.gitlab.com/blog/gitlab-next-gen-scm/)：重构的 GitLab 后端，提供免克隆的代理 API。

### 运行时 Git 存储

成千上万地创建的无头、API 优先 Git 仓库：分支、提交和差异，无 PR、评审或门禁。它们填补半个槽位，按运行铸造短期的仓库级令牌。

**适用场景**：运行需要廉价的单运行临时仓库或分叉（每张工单一个分叉、代理的工作状态），而 PR 最终仍落在上述类型的完整 Git 托管上。

* * [Cloudflare Artifacts](https://www.cloudflare.com/product/artifacts/)：Workers 上的 Git 兼容存储；按任务分叉基线。
* * [Pierre code.storage](https://code.storage/docs)：白标 Git API，提供临时分支。

## 进一步研究

对照以下问题逐一考察候选项：

1. 1. **触发器**：哪些事件（PR 打开、推送、评审提交、标签添加、检查完成）以 Webhook 形式到达编排器，且投递是否签名？
2. 2. **身份**：运行能否以专用机器人或 App 身份行动？这能保持运行可归因、让自动化评审运行批准另一身份撰写的 PR、并决定运行自身的推送是否触发下一运行。
3. 3. **门禁**：必需检查、必需批准和分支保护在用户的计划和仓库可见性下是否可用，且机器人能否进入旁路名单？
4. 4. **速率限制**：每个运行器上的每个运行共享何种 API 预算，按何种单位（安装、令牌、用户、工作区），如何扩展？
5. 5. **填补槽位**：同一产品覆盖了工单追踪器、编排器、运行器和可观测平台中的哪些？

在一手来源中寻找答案：REST API 与 Webhook 参考、速率限制页面、定价与计划对比页面、访问令牌或 App 文档、以及更新日志或发布说明。

最快失效的信息：Beta 和预览状态（整个代理原生与 Git 存储类型）、速率限制数值、哪个计划解锁门禁、以及收购、关闭和重新开放后的可用性。例如截至 2026-10，AWS CodeCommit 已重新向新客户开放，而 Bitbucket Data Center 仅作为与 Cloud 捆绑的混合许可证销售。

## 持久陷阱

在设计依赖之前，逐一核实以下各项仍然成立。

* * **GitHub `GITHUB_TOKEN` 会切断链路。** 由内置 Actions 令牌引发的事件不会启动新的工作流运行（`workflow_dispatch` 和 `repository_dispatch` 除外），因此运行若用它推送会静默阻断下一运行。从第一天起就给运行分配 GitHub App 身份。
* * **门禁常隐藏在付费计划后。** 截至 2026-10，GitHub 私有仓库的规则集需要 Pro、Team 或 Enterprise，GitLab 批准规则需要 Premium，强制执行的 Bitbucket 合并检查需要 Premium。
* * **速率限制按身份计算，且机器人类型会改变限额。** 截至 2026-10，GitHub App 安装每小时 5,000 请求，而 `GITHUB_TOKEN` 仅 1,000；Bitbucket 用户令牌维持基础速率，访问令牌则随之扩展；Azure DevOps 随访问级别提升机器人预算。
* * **套件追踪器意味着双事件流。** 编排器自行汇合 Git 托管与工单追踪器的事件。
