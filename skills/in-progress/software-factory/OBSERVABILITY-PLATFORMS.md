# 可观测平台

> 2026年10月调查，不保证是最新的。用它来了解大概情况，然后自己研究当前的选项：产品、价格、限制和测试版变化很快。

用于填充可观测平台槽位的定向指南，或决定是否刻意不使用。类型按**人类需要在运行中看到什么**划分：日志、智能体追踪（每次模型调用和工具调用作为一个 span），或将该追踪与用户系统其余部分关联。自托管与 SaaS 模式贯穿后三种类型。当运行应根据平台所见采取行动时，将平台作为工单来源：监控器生成工单，运行读取它。

## 运行器如何接入

每种类型通过五种钩子之一接收数据。填槽时在产品旁标注钩子名称。

* **Harness OTel**：智能体 harness 自行导出 OpenTelemetry。Claude Code 和 Claude Agent SDK 使用 `CLAUDE_CODE_ENABLE_TELEMETRY=1` 加上 `OTEL_*` 变量，提示词和工具输出需显式启用；Codex CLI 在 `config.toml` 中使用 `[otel]` 块。([Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/observability), [Claude Code](https://code.claude.com/docs/en/monitoring-usage), [Codex config](https://developers.openai.com/codex/config-advanced))
* **Platform push**：模型平台替你发送追踪，如 Claude Managed Agents 将追踪发送到在 Claude Console 中设置的 OTLP 目标。([Claude 文档](https://platform.claude.com/docs/en/manage-claude/opentelemetry/api))
* **SDK wrapper**：在运行器代码中调用一次，例如 LangSmith 的 `configure_claude_agent_sdk()`。
* **Hook plugin**：Claude Code hook 在每次 Stop 时读取会话转录并发送，如 Langfuse 的 Claude Code 插件。
* **Gateway**：运行器的模型基础 URL 指向一个日志代理。零运行器代码，仅能看到模型调用：harness 内部的工具调用不可见。

OpenTelemetry 是通用线路：harness OTel 和 platform push 都落入任意 OTLP 后端。GenAI 语义约定（`invoke_agent`、`chat`、`execute_tool` spans）仍在敲定中，预期 span 名称会漂移。([OTel GenAI agent spans](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans))

## 类型 1：无（运行器或编排产品已足够）

**是什么**：人类在运行器或编排器已展示的地方阅读每次运行：CI 运行页面、沙箱仪表板、作为运行制品保存的智能体转录。无需构建钩子，与运行器一样自托管，成本打包在运行器中。

**工厂何时选择**：每天运行次数少，单人看护，问题仅是“失败了吗？打印了什么？”大多数工厂从此起步，当需要跨运行搜索、每运行 token 或成本汇总、或比运行器保留更久的留存时便长出此类型。

**起点**：[GitHub Actions 运行日志](https://docs.github.com/en/organizations/managing-organization-settings/configuring-the-retention-period-for-github-actions-artifacts-and-logs-in-your-organization)、[E2B 沙箱日志与指标](https://e2b.dev/docs)、[Daytona 沙箱遥测](https://www.daytona.io/docs/en/observability/otel-collection/)、[OpenAI Agents SDK 追踪](https://openai.github.io/openai-agents-python/tracing/)。沙箱产品常通过 OTLP 转发相同数据，这是通往类型 2 的路径。

**核实**：每个运行器产品的保留天数；部分沙箱保留极短（Daytona：3 天，截至 2026-10）。

## 类型 2：LLM 与智能体追踪平台

**是什么**：围绕智能体追踪构建的后端：每次运行作为模型调用和工具调用的树，包含 token 成本、提示词与会话，通常叠加评测与数据集。

**工厂何时选择**：人类跨多次运行问“为什么智能体那样做？”；回溯运行或人工对比运行；token 支出需每运行汇总。自托管开源产品可将转录留在内部。

**类型特征**：接收 harness OTel、SDK wrapper 或 hook plugin，常也接收 platform push。按量计费，智能体运行 span 密集（每工具调用一个 span），故按运行预算而非按请求预算。

**起点**：[Langfuse](https://langfuse.com/pricing) 与 [Arize Phoenix](https://phoenix.arize.com)（开源、可自托管）、[LangSmith](https://www.langchain.com/pricing-langsmith)、[Braintrust](https://www.braintrust.dev/pricing)。

**核实**：免费额度限制、计费单位（trace、span、unit 或 GB；Braintrust 2026-03 切为 GB）、哪个 tier 解锁自托管、当前归属。

## 类型 3：通用 APM 与日志后端

**是什么**：用户现有的可观测栈（所有服务的日志、指标、分布式追踪），叠加智能体或 LLM 视图。

**工厂何时选择**：团队已在付费；运行需与应用自身追踪关联（Agent SDK 传播 `TRACEPARENT`，运行嵌套在调用者追踪内）；或监控工单源已从中读取。

**类型特征**：harness OTel 直达厂商 OTLP 端点。按 GB、事件或主机计费，故工厂量在既有账单上是舍入误差，在新账单上却是惊喜。

**起点**：[Datadog Agent Observability](https://www.datadoghq.com/product/llm-observability/)、[Grafana Cloud](https://grafana.com/pricing/)、[Honeycomb](https://www.honeycomb.io/pricing)、[SigNoz](https://signoz.io/pricing/)（开源）。

**核实**：智能体视图是否 GA 或预览、哪些 harness 有部署指南、免费额度量。

## 类型 4：带请求日志的 LLM 网关

**是什么**：运行器与模型商之间的代理，记录每个请求与响应，附带路由、密钥、预算与缓存。

**工厂何时选择**：用户想要每运行支出上限或一个密钥保管库服务多运行器，日志随之而来；或 harness 自身无导出。仅见模型调用，故搭配类型 2 或 3 以获取工具调用。

**起点**：[LiteLLM Proxy](https://docs.litellm.ai/docs/proxy/logging)（开源，转发日志到 Langfuse 或 OTel）、[Portkey](https://portkey.ai/pricing)。

**核实**：SaaS 按请求定价、产品是否仍在活跃开发（Helicone 于 2026-10 被收购后进入维护模式）。

## 进一步研究

进一步研究

1. **运行器或编排器是否已足够？** 对比人类所需（跨运行搜索、每运行成本、留存）与运行页面或沙箱仪表板所给。此处为是即意味着刻意选择无。
2. **智能体 harness 如何导出追踪？** 找出 harness 支持的五种钩子中的哪些，再找产品接受哪些。确认工具调用以 span 到达，以及提示词和工具输出是否需显式启用。
3. **是否可自托管？** 许可证、哪个 tier 允许、运行在什么上。当转录必须留在内部时此项决定性。
4. **成本形状如何？** 计费单位、免费额度、按每工具调用一个 span 估算的每运行成本，乘以每日运行数。
5. **留存多久？** 各 tier 保留天数，对比运行出错时人类回溯多远。

**答案在哪里**：厂商定价页（单位、tier、留存）；厂商针对特定 harness 的集成文档；harness 自身的监控文档（导出什么、哪些标记是 beta）；OpenTelemetry GenAI 语义约定；产品仓库的许可证与发布活动；厂商更新日志或博客的归属变更。

**最快过期的内容**：免费额度与价格；beta 与预览标记（Claude Code 追踪导出、Console platform push、厂商智能体视图）；归属与维护状态；计费单位；留存天数；GenAI span 名称。
