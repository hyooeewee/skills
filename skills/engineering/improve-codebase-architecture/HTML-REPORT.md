# HTML 报告格式

架构审查会以单个自包含 HTML 文件的形式渲染到操作系统临时目录中。Tailwind 和 Mermaid 均来自 CDN。Mermaid 能可靠地处理图形类图表；手写的 div 和内联 SVG 则处理更具编辑风格的视觉内容（体量图、剖面图）。将两者混合使用——不要事事依赖 Mermaid，否则它会开始显得千篇一律。

## 骨架

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Architecture review — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* small custom layer for things Tailwind doesn't cover cleanly:
         dashed seam lines, hand-drawn-feeling arrow heads, etc. */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## 页眉

仓库名称、日期，以及一个紧凑的图例：实线框 = 模块，虚线 = 接缝，红色箭头 = 泄漏，粗黑框 = 深模块。不写引言段落——直接进入候选方案。

## 候选卡片

图表承担了主要表达。文字描述要精简、朴素，并直接使用（来自 `/codebase-design` 技能的）词汇表中的术语，不加任何渲染。

每个候选方案是一个 `<article>`：

* **标题** — 简短，指明深化方向（例如“收缩订单接收管道”）。
* **徽章行** — 建议强度（`Strong` = 翠绿色，`Worth exploring` = 琥珀色，`Speculative` = 石板灰），外加一个依赖类别标签（`in-process`、`local-substitutable`、`ports & adapters`、`mock`）。
* **文件** — 等宽字体列表，`font-mono text-sm`。
* **前后对比图** — 核心部分。两列并排。参见下面的模式。
* **问题** — 一句话。痛点是什么。
* **解决方案** — 一句话。改变了什么。
* **收获** — 项目符号，每条不超过 6 个词。例如：“测试只接触一个接口”、“定价逻辑不再泄漏”、“删除 4 个浅层包装器”。
* **ADR 标注**（如适用）— 琥珀色框内的一行文字。

不写解释性段落。如果图表需要一段文字才能看懂，请重画图表。

## 图表模式

选择适合候选方案的图表模式。可以混合使用。不要每张图都一样——多样性本身就是目的之一。

### Mermaid 图（处理依赖/调用流程的主力）

当要表达“X 调用 Y，Y 调用 Z，看看这堆乱麻”时，使用 Mermaid 的 `flowchart` 或 `graph`。用 Tailwind 样式的卡片包裹它，这样不会显得像是硬塞进来的。使用 classDef 将泄漏边染成红色，将深模块染成深色。时序图很适合表现“之前：6 次往返；之后：1 次”。

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

### 手工绘制的方框与箭头（当 Mermaid 的布局不合你意时）

模块用带边框和标签的 `<div>` 表示。箭头用内联 SVG 的 `<line>` 或 `<path>` 元素，绝对定位在相对定位的容器之上。当你想让“之后”图看起来像一个带粗边框、内部灰显的深模块时，就用这种方法——Mermaid 无法以正确的权重渲染出这种效果。

### 剖面图（适合表现分层浅层性）

堆叠水平色带（`h-12 border-l-4`）来展示一次调用穿过的各层。之前：6 条薄层，每层都没做任何事。之后：1 条厚带，标注着合并后的职责。

### 体量图（适合表现“接口与实现一样宽”）

每个模块两个矩形——一个表示接口表面积，一个表示实现。之前：接口矩形几乎与实现矩形一样高（浅层）。之后：接口矩形很矮，实现矩形很高（深层）。

### 调用图折叠

Before: a tree of function calls rendered as nested boxes. After: the same tree collapsed into one box, with the now-internal calls shown faded inside it.

## 样式指南

* 偏编辑风格，而非企业仪表盘风格。留白要充足。标题可选用衬线字体（`font-serif` 与 stone/slate 配色很搭）。
* 颜色要克制：一种强调色（翠绿或靛蓝），外加红色表示泄漏、琥珀色表示警告。
* 让图表高度保持在约 320px，这样前后对比可以舒服地并排展示，无需滚动。
* 图表内的模块标签使用 `text-xs uppercase tracking-wider`——它们应读作示意图，而不是界面。
* 唯一的脚本是 Tailwind CDN 和 Mermaid ESM 导入。报告其余部分都是静态的——没有应用代码，除了 Mermaid 自身的渲染外没有交互。

## 首选推荐部分

一张更大的卡片。候选名称、一句理由、指向其卡片的锚点链接。仅此而已。

## 语气

平实简洁的英语——但架构名词和动词直接来自 `/codebase-design` 技能。简洁不是偏离术语的借口。

**准确使用：** module、interface、implementation、depth、deep、shallow、seam、adapter、leverage、locality。

**切勿替代：** component、service、unit（来替代 module）· API、signature（来替代 interface）· boundary（来替代 seam）· layer、wrapper（来替代 module，当你想表示 module 时）。

**符合风格的表述：**

* “订单接收模块是浅层的——接口几乎与实现匹配。”
* “定价逻辑跨接缝泄漏。”
* “加深：一个接口，一个可测试的地方。”
* “两个适配器证明接缝的合理性：生产环境用 HTTP，测试环境用内存。”

**收获列表** 用术语命名收益：*"locality: bugs concentrate in one module"*、*"leverage: one interface, N call sites"*、*"interface shrinks; implementation absorbs the wrappers"*。不要写 *"easier to maintain"* 或 *"cleaner code"*——这些词不在词汇表中，不值得占位置。

不要含混不清，不要废话连篇，不要写“it's worth noting that…”。如果一个句子能变成列表项，就把它变成列表项。如果一个列表项可以删掉，就删掉。如果一个词不在 `/codebase-design` 词汇表中，先去找一个词汇表中的词，而不要发明新词。
