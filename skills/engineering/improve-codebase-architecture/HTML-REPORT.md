# HTML 报告格式

架构评审被渲染为操作系统临时目录中的一个独立 HTML 文件。Tailwind 和 Mermaid 都来自 CDN。Mermaid 可靠地处理图形化图表；手动构建的 div 和内联 SVG 处理更具编辑性的视觉效果（质量图、截面图）。将两者混合使用——不要完全依赖 Mermaid，否则它会开始看起来很普通。

## 脚手架

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

仓库名称、日期和紧凑的图例：实心框 = 模块，虚线 = 接缝，红色箭头 = 泄漏，厚实深色框 = 深层模块。没有介绍段落——直接进入候选项。

## 候选卡片

图表承担着主要内容。文字稀疏、朴实，且不带客套地使用词汇表中的术语（来自 `/codebase-design` 技能）。

每个候选都是一个 `<article>`：

* **标题** — 简短，命名加深操作（例如 "Collapse the Order intake pipeline"）。
* **徽章行** — 推荐强度（`Strong` = 翠绿，`Worth exploring` = 琥珀色，`Speculative` = 石板灰），以及依赖分类标签（`in-process`，`local-substitutable`，`ports & adapters`，`mock`）。
* **文件** — 等宽列表，`font-mono text-sm`。
* **之前 / 之后图表** — 中心部分。两列并排。见下方的模式。
* **问题** — 一句话。哪里痛。
* **解决方案** — 一句话。改变了什么。
* **收益** — 要点，每条 ≤6 个单词。例如 "Tests hit one interface"（测试命中一个接口）, "Pricing logic stops leaking"（定价逻辑停止泄漏）, "Delete 4 shallow wrappers"（删除 4 个浅层包装器）。
* **ADR 提示**（如适用）——琥珀色背景框中的一行。

没有解释段落。如果图表需要一段文字才能理解，就重绘图表。

## 图表模式

选择适合候选项的模式。混合使用。不要让每个图表看起来都一样——多样性是重点的一部分。

### Mermaid 图表（依赖/调用流程的主力）

当重点是 "X 调用 Y 调用 Z，看这团乱麻" 时，使用 Mermaid 的 `flowchart` 或 `graph`。将其包裹在 Tailwind 样式的卡片中，以免感觉突兀。使用 classDef 进行样式设置，将泄漏边缘标为红色，将深层模块标为深色。时序图很适合用于 "之前：6 次往返；之后：1 次"。

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

### 手动构建的方框和箭头（当 Mermaid 的布局与你的要求相悖时）

将模块作为带有边框和标签的 `<div>`。箭头作为内联 SVG `<line>` 或 `<path>` 元素，绝对定位在相对容器上。当你希望 "之后" 的图表看起来像一个厚边框的深层模块且内部呈灰色时，使用这个方法——Mermaid 无法以正确的权重渲染出这种效果。

### 截面图（适合展示分层浅层性）

堆叠水平带（`h-12 border-l-4`）以显示调用经过的层。之前：6 个细层，每层什么都没做。之后：1 个粗带，标有整合后的职责。

### 质量图（适合 "接口与实现一样宽" 的情况）

每个模块两个矩形——一个用于接口表面积，一个用于实现。之前：接口矩形几乎与实现矩形一样高（浅层）。之后：接口矩形短，实现矩形长（深层）。

### 调用图收拢

Before: a tree of function calls rendered as nested boxes. After: the same tree collapsed into one box, with the now-internal calls shown faded inside it.

## 样式指南

* 偏向编辑风格，而非企业仪表盘。充足的留白。标题可选衬线字体（`font-serif` 与 stone/slate 搭配效果很好）。
* 谨慎使用颜色：一种强调色（翠绿或靛蓝），红色用于泄漏，琥珀色用于警告。
* 保持图表高度在 \\\~320px 左右，以便之前/之后可以舒适地并排显示而无需滚动。
* 在图表中使用 `text-xs uppercase tracking-wider` 作为模块标签——它们应该读起来像示意图，而不是 UI。
* 唯一的脚本是 Tailwind CDN 和 Mermaid ESM 导入。报告其余部分是静态的——没有应用代码，除了 Mermaid 自身的渲染外没有其他交互性。

## 顶级推荐部分

一张较大的卡片。候选名称，一句关于原因的话，指向其卡片的锚点链接。仅此而已。

## 语气

简单的英语，简洁——但建筑名词和动词直接来自 `/codebase-design` 技能。简洁不是偏离的理由。

**准确使用：** module（模块）, interface（接口）, implementation（实现）, depth（深度）, deep（深层）, shallow（浅层）, seam（接缝）, adapter（适配器）, leverage（杠杆作用）, locality（局部性）。

**永远不要替换：** component（组件）, service（服务）, unit（单元，指模块） · API, signature（签名，指接口） · boundary（边界，指接缝） · layer（层）, wrapper（包装器，指模块，当你指的是模块时）。

**符合风格的措辞：**

* 订单接入模块很浅——接口几乎与实现匹配。
* 定价逻辑跨越接缝泄漏。
* 加深：一个接口，一个测试点。
* 两个适配器证明了接缝的合理性：生产环境用 HTTP，测试环境用内存。

**收益要点** 使用词汇表术语命名收益：*"locality: bugs concentrate in one module"*（局部性：Bug 集中在一个模块）, *"leverage: one interface, N call sites"*（杠杆作用：一个接口，N 个调用点）, *"interface shrinks; implementation absorbs the wrappers"*（接口收缩；实现吸收了包装器）。不要写 *"easier to maintain"* 或 *"cleaner code"* —— 这些术语不在词汇表中，也不配占据一席之地。

没有含糊其辞，没有寒暄，没有 "值得注意的是……"。如果一句话可以是一个要点，就把它变成要点。如果一个要点可以删减，就删掉它。如果一个术语不在 `/codebase-design` 词汇表中，在创造新术语之前，先找一个现有的。
