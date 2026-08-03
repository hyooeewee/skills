---
name: design-an-interface
description: 使用并行子代理为模块生成多个截然不同的接口设计。当用户想要设计 API、探索接口选项、比较模块形状或提到“设计两次”时使用。

---

# 设计接口

基于《软件设计哲学》中的“设计两次”理念：你的第一个想法不太可能是最好的。生成多个截然不同的设计，然后进行比较。

## 工作流程

### 1. 收集需求

设计前，请了解：

* [ ] 这个模块解决了什么问题？
* [ ] 调用者是谁？（其他模块、外部用户、测试）
* [ ] 关键操作有哪些？
* [ ] 有什么约束吗？（性能、兼容性、现有模式）
* [ ] 应该隐藏内部实现还是暴露外部？

Ask: "What does this module need to do? Who will use it?"

### 2. 生成设计（并行子代理）

使用 Task 工具同时生成 3 个以上的子代理。每个代理必须产生一种**截然不同**的方法。

```
Prompt template for each sub-agent:

Design an interface for: [module description]

Requirements: [gathered requirements]

Constraints for this design: [assign a different constraint to each agent]
- Agent 1: "Minimize method count - aim for 1-3 methods max"
- Agent 2: "Maximize flexibility - support many use cases"
- Agent 3: "Optimize for the most common case"
- Agent 4: "Take inspiration from [specific paradigm/library]"

Output format:
1. Interface signature (types/methods)
2. Usage example (how caller uses it)
3. What this design hides internally
4. Trade-offs of this approach
```

### 3. 展示设计

展示每个设计，包括：

1. **接口签名** - 类型、方法、参数
2. **使用示例** - 调用者实际在实践中的使用方式
3. **它隐藏了什么** - 内部保留的复杂性

顺序展示设计，以便用户在比较之前能理解每种方法。

### 4. 比较设计

展示所有设计后，从以下几个方面进行比较：

* **接口简洁性**：方法更少，参数更简单
* **通用 vs 专用**：灵活性 vs 专注度
* **实现效率**：接口形状是否允许高效的内部实现？
* **深度**：小接口隐藏大量复杂性（好） vs 大接口但实现单薄（坏）
* **正确使用的便捷性** vs **误用的便捷性**

用散文（文字）讨论权衡，而不是表格。强调设计分歧最大的地方。

### 5. 综合/综合设计

最好的设计通常结合了多个选项的见解。提问：

* “哪个设计最适合你的主要用例？”
* “其他设计中是否有值得采纳的元素？”

## 评估标准

来自《软件设计哲学》：

**接口简洁性**：方法更少、参数更简单 = 更容易学习和正确使用。

**通用性**：无需更改即可处理未来的用例。但要注意过度通用化。

**实现效率**：接口形状是否允许高效的实现？还是会迫使内部实现变得笨拙？

**深度**：小接口隐藏大量复杂性 = 深度模块（好）。大接口但实现单薄 = 浅层模块（避免）。

## 反模式

* 不要让子代理产生相似的设计 - 强制追求根本差异
* 不要跳过比较 - 价值在于对比
* 不要实现 - 这纯粹是关于接口形状
* 不要基于实现工作量进行评估
