# CONTEXT.md 格式

## 结构

```md
# {Context Name}

{One or two sentence description of what this context is and why it exists.}

## Language

**Order**:
{A one or two sentence description of the term}
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request

**Customer**:
A person or organization that places orders.
_Avoid_: Client, buyer, account
```

## 规则

* 保持有主见。当同一个概念存在多个词时，选择最好的一个，并将其他的列在 `_Avoid_` 下。
* 保持定义简洁。最多一两句话。定义它*是*什么，而不是它*做*什么。
* 只包含特定于本项目上下文的术语。通用编程概念（超时、错误类型、工具模式）不属于此列，即使项目广泛使用了它们。在添加术语之前，请自问：这是一个特定于该上下文的概念，还是一个通用编程概念？只有前者才属于此列。
* 当自然出现分组时，将术语归类到子标题下。如果所有术语都属于一个连贯的领域，则使用扁平列表即可。

## 单个上下文仓库 vs 多个上下文仓库

单个上下文（大多数仓库）：仓库根目录有一个 `CONTEXT.md`。

多个上下文：仓库根目录有一个 `CONTEXT-MAP.md`，列出了上下文、它们所在的位置以及它们之间的关系：

```md
# Context Map

## Contexts

- [Ordering](./src/ordering/CONTEXT.md) — receives and tracks customer orders
- [Billing](./src/billing/CONTEXT.md) — generates invoices and processes payments
- [Fulfillment](./src/fulfillment/CONTEXT.md) — manages warehouse picking and shipping

## Relationships

- **Ordering → Fulfillment**: Ordering emits `OrderPlaced` events; Fulfillment consumes them to start picking
- **Fulfillment → Billing**: Fulfillment emits `ShipmentDispatched` events; Billing consumes them to generate invoices
- **Ordering ↔ Billing**: Shared types for `CustomerId` and `Money`
```

该技能会推断适用的结构：

* 如果存在 `CONTEXT-MAP.md`，请阅读它以查找上下文
* 如果仅存在根目录的 `CONTEXT.md`，则为单个上下文
* 如果两者都不存在，则在解析第一个术语时延迟创建根目录的 `CONTEXT.md`

当存在多个上下文时，推断当前主题与哪个上下文相关。如果不清楚，请询问。
