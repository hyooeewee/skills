# 运行器

> 2026年10月调查，不保证是最新的。用它来了解大概情况，然后自己研究当前的选项：产品、价格、限制和测试版变化很快。

运行器是**一次性的**：每次运行都在一台全新的机器上开始，运行结束后不留任何痕迹。五种类型填补这个槽位。它们的主要区别在于谁拥有 Agent 循环、谁拥有机器：

| 类型          | Agent 循环     | 机器         | 同时填补         |
| ----------- | ------------ | ---------- | ------------ |
| CI 运行器      | 你的，在 CI 作业内部 | CI 厂商的，或你的 | 编排器、可观测性平台   |
| MicroVM API | 你的，在编排器或运行器上 | 供应商的       | 无            |
| 捆绑 Agent 平台 | 供应商的         | 供应商的       | 部分编排器、可观测性平台 |
| 自带 Worker   | 供应商的         | 你的         | 部分编排器、可观测性平台 |
| 自有 VPS 或容器  | 你的           | 你的         | 无            |

运行器选择是按运行决定的：评审运行用廉价 CI 运行器，实现运行用 MicroVM，这是常见设计。

## CI 运行器

每个 CI 作业启动一个全新的 VM 或 Pod，由 Git 宿主自带的工作流引擎启动；Agent CLI 只是作业中的一个步骤。编排器通过工作流触发器（Issue、PR、定时、手动触发）启动运行，并发组防止同一工单有两个运行同时进行。托管运行器天生一次性；自托管运行器通过临时模式实现，每个作业一个 Pod，用完即删。按分钟计费。

**适用场景**：Git 宿主是 GitHub 或 GitLab，且所有触发器都是 Git 宿主事件或定时器：工作流文件就是编排器，运行页面就是可观测性平台，一个产品填补三个槽位。

起步参考：&#x20;

* [GitHub Actions](https://docs.github.com/en/actions/reference/limits)：全 GitHub 工厂的默认选择；[定价](https://docs.github.com/en/billing/reference/actions-runner-pricing)。
* [Claude Code GitHub Action](https://code.claude.com/docs/en/github-actions)：作为工作流步骤的 Claude Code。
* [GitLab 托管运行器](https://docs.gitlab.com/ci/runners/hosted_runners/linux/)：每个作业一个临时 VM。
* [Actions Runner Controller](https://docs.github.com/en/actions/concepts/runners/actions-runner-controller)：自托管，用户 Kubernetes 上每个作业一个临时 Pod。

## MicroVM API

一次 SDK 调用在约一秒内创建一个隔离的 MicroVM（厂商称「沙箱」）；编排器在其中执行命令，或在其中启动 Agent CLI。按 vCPU 和 RAM 的秒数计费。编排器为每次运行创建一个，运行结束时销毁，并设置超时作为兜底。供应商也出售持久化功能（暂停/恢复、快照、磁盘）；工厂将其用作缓存，在工单追踪器、Git 宿主和编排器中保持运行间的记忆。

**适用场景**：工厂有自己的编排器（服务器、队列），想要大量短生命周期、并行的运行，且不想管理机器。对比最大运行时长、并发上限、等待模型的时间是否计费：截至 2026-10，Vercel 仅对活跃 CPU 计费，因此模型等待时间在那里几乎免费。

起步参考：[E2B](https://e2b.dev/pricing)、[Daytona](https://www.daytona.io/pricing)、[Modal Sandboxes](https://modal.com/docs/guide/sandboxes)、[Vercel Sandbox](https://vercel.com/docs/vercel-sandbox/pricing)。

## 捆绑 Agent 平台

包含运行器的托管编码 Agent：厂商为每次运行提供全新的 VM 或容器，在其中运行自己的 Agent 循环，并推送分支或 PR。只有分支会保留。运行由 Git 宿主或工单追踪器中的 @-提及或分配、聊天消息、定时任务或 HTTP 调用触发，因此平台填补了大部分编排器功能；其会话记录视图填补了可观测性平台。费用包含在席位或订阅中，或按运行计费。

**适用场景**：用户想要最少的活动部件，且满意该厂商的 Agent。确认其触发器覆盖用户的工单追踪器。

起步参考：&#x20;

* [Claude Code 云端版](https://code.claude.com/docs/en/claude-code-on-the-web)，带有用于 cron、API 和 GitHub 事件触发的[计划任务](https://code.claude.com/docs/en/routines)。
* [Claude Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview)：API 优先；针对环境创建运行并流式传输事件。
* [Codex 云端版](https://developers.openai.com/codex/cloud)：每个任务一个容器。
* [Copilot 编码 Agent](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent)：将 Issue 分配给 Copilot；其运行器是一个 GitHub Actions 运行器。

## 自带 Worker

捆绑平台保留其 Agent 循环和运行队列，但工具调用在用户托管的 Worker 上执行。Worker 轮询厂商的出站接口，认领运行，执行，退出：形态与自托管 CI 运行器相同。一次性由用户负责：在 Kubernetes 或 MicroVM API 下运行 Worker，使每次运行获得全新磁盘。费用为厂商的席位或用量费，加上用户的机器费用。

**适用场景**：用户想要捆绑 Agent，但运行必须访问私有网络、数据必须留在用户机器上、或需要使用自定义硬件。

起步参考：[Claude Code 自托管环境](https://code.claude.com/docs/en/self-hosted-environments)、[Cursor 自托管池](https://cursor.com/docs/cloud-agent/self-hosted-pool)、[ARC 上的 Copilot 编码 Agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment)。

## 自有 VPS 或容器

用户租用或运行的一台机器，每次运行一个全新的容器（或机器）：机器可能存活数月，容器只存活一次运行。最大运行时长由编排器设定的截止时间决定。一次性通过 `docker run --rm`、设置 `ttlSecondsAfterFinished: 0` 的 Kubernetes Job、或运行结束销毁机器来实现；所有缓存放在下一次运行视为可选的路径上。费用为固定月费，或运行时按秒计费。

**适用场景**：运行时间长、稳定或负载重、每小时成本占主导、或已有服务器托管编排器。

起步参考：[Fly Machines](https://docs.fly.io/machines/overview) ([定价](https://docs.fly.io/about/pricing))、[Hetzner Cloud](https://www.hetzner.com/cloud/) + Docker、[Kubernetes Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)。

## 进一步研究

### 决定槽位的问题

手头拿着工厂的运行情况，对每个候选项核对以下问题：

1. **编排器如何启动一个运行？** 工作流触发器、SDK 调用、@-提及、HTTP 调用、或 Worker 认领的厂商队列。这符合设计中已定的编排器吗？
2. **最大运行时长和并发上限是多少？** 找出每个套餐层级的默认超时和上限，以及达到上限时会发生什么。拿它们对照设计中的最长运行和最繁忙突发。
3. **成本结构如何？** 按分钟、按 vCPU 和 RAM 秒数、仅活跃 CPU、按运行、按席位、或固定月费。等待模型的时间计费吗？模型 Token 在价格内还是额外计费？
4. **如何实现一次性？** 默认全新、临时标志、或编排器自行清理。存在哪些持久化功能，设计能否将其仅用作缓存？
5. **运行能访问什么？** 私有网络访问、出站防火墙或允许列表、密钥如何到达运行器、还有谁能读取它们。
6. **还填补了哪些其他槽位？** 统计它读取的触发器作为编排器覆盖，其运行页面或会话记录作为可观测性平台覆盖。

### 答案在哪里

* 厂商的**限制或配额页面**和**定价页面**，而非落地页。
* 创建调用的 **SDK 或 API 参考**：其超时、生命周期和网络参数。
* **安全或网络文档**：防火墙、允许列表和密钥处理。
* **更新日志或发布说明**：Beta、预览版和 GA 状态，以及计划门控（Team、Enterprise）。
* 对 CI 运行器，Git 宿主的**账单文档**。

### 什么最快过期

* **价格和免费额度。** 截至 2026-10，GitHub 曾宣布对自托管运行器按分钟收费，随后推迟了该计划。
* **Beta、预览版和早期访问状态**，以及哪些计划解锁某项功能。截至 2026-10，Claude Managed Agents 和 Claude Code 自托管环境均处于 Beta 阶段。
* **各层级的默认超时时间和最大运行时长**。
* **捆绑平台上的触发器集成**：哪些工单追踪器和聊天工具能启动运行。
* **启动时间**，这些是供应商宣称的数值；如果重要请自行测量。

上述类型特征（谁拥有循环、谁拥有机器、填补哪些槽位）变化缓慢；请信任它们，而非此处的任何数字。
