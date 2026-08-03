Quickstart:

```bash
npx skills add mattpocock/skills --skill=implement
```

```bash
npx skills update implement
```

[源码](https://github.com/mattpocock/skills/tree/main/skills/engineering/implement)

## 功能说明

`implement` 构建规格说明或一组票据中描述的工作——通过测试驱动开发、类型检查和完整测试套件来驱动它，然后移交给审查并提交到当前分支。

它**不**决定构建什么。规格说明已经确定，接口已经达成一致；`implement` 执行该计划，而不是重新打开它。它是双手，不是头脑——思考发生在上游。

## 何时使用

你通过输入 `/implement` 来调用它——代理不会自行使用它。

一旦工作被记录为规格说明或拆分为票据，并且你准备好将其转化为代码，就使用它。如果规格说明尚不存在，请先编写它——为此，使用 [to-spec](https://aihero.dev/skills-to-spec)，或使用 [to-tickets](https://aihero.dev/skills-to-tickets) 将规格说明拆分为票据。如果你只想在没有完整规格说明的情况下先构建某些测试驱动的内容，请直接使用 [tdd](https://aihero.dev/skills-tdd)。

## 预商定的接口

`implement` 运行的理念是**接口**——一个功能被测试的稳定接口，在任何代码编写之前就已选定。它不会在构建过程中发明接口；它使用已经选定的接口（在[to-spec](https://aihero.dev/skills-to-spec)期间）并通过[tdd](https://aihero.dev/skills-tdd)针对它们编写测试。在预商定的接口上工作能保持实现的诚实性：测试针对的是持久的东西，因此底层的代码可以在不移动测试的情况下移动。

围绕核心，它保持循环紧凑——经常进行类型检查，在过程中运行单个测试文件，最后运行整个套件——然后以审查通过和提交到当前分支结束。

## 在系统中的位置

`implement` 是主链末尾的构建步骤，就在审查之前：

```txt
grill-with-docs → to-spec → to-tickets → implement → code-review
```

工作已被规格化并排序后，再使用它，而不是在此之前。它的关键邻居是[to-tickets](https://aihero.dev/skills-to-tickets)，它生成票据——每个票据都声明其阻塞边界——它处理这些票据，以及[tdd](https://aihero.dev/skills-tdd)，它在内部驱动\[tdd]在每个接口编写测试，然后运行其自己的[code-review](https://aihero.dev/skills-code-review)审查和提交。当你不确定哪个技能或流程适合时，[ask-matt](https://aihero.dev/skills-ask-matt)会为你路由。
