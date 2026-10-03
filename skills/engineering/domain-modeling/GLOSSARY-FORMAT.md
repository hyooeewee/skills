# GLOSSARY.md 格式

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

* **要有明确立场。** 当同一概念存在多个词时，选出最佳的一个，并将其他词列在 `_Avoid_` 下。
* **定义要精炼。** 最多一两句话。定义它是什么，而不是它做什么。
* **只包含特定于本项目上下文的术语。** 通用编程概念（超时、错误类型、工具模式）不属于这里，即使项目大量使用它们也是如此。在添加术语之前，先问一问：这是该上下文独有的概念，还是通用编程概念？只有前者才属于这里。
* **当自然聚类出现时，将术语分组到子标题下。** 如果所有术语都属于一个统一的领域，使用扁平列表即可。

## 单上下文与多上下文仓库

**单上下文（大多数仓库）：** 仓库根目录下有一个 `GLOSSARY.md`。

**多上下文：** 仓库根目录下的 `GLOSSARY-MAP.md` 列出了上下文、它们的位置以及它们之间的关系：

```md
# Glossary Map

## Contexts

- [Ordering](./src/ordering/GLOSSARY.md): receives and tracks customer orders
- [Billing](./src/billing/GLOSSARY.md): generates invoices and processes payments
- [Fulfillment](./src/fulfillment/GLOSSARY.md): manages warehouse picking and shipping

## Relationships

- **Ordering → Fulfillment**: Ordering emits `OrderPlaced` events; Fulfillment consumes them to start picking
- **Fulfillment → Billing**: Fulfillment emits `ShipmentDispatched` events; Billing consumes them to generate invoices
- **Ordering ↔ Billing**: Shared types for `CustomerId` and `Money`
```

该技能会推断适用哪种结构：

* 如果存在 `GLOSSARY-MAP.md`，请阅读它以查找上下文
* 如果仅存在根目录下的 `GLOSSARY.md`，则为单上下文
* 如果两者都不存在，则在首个术语被解析时惰性创建根目录下的 `GLOSSARY.md`

当存在多个上下文时，推断当前主题与哪个相关。如果不清楚，请询问。
