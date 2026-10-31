## 它做什么

`pr` 定义了 pull request 正文的结构：一个展示变更的 **Summary（摘要）**、一个证明其有效的 **Evidence（证据）**，以及一个关于合入风险的 **Merge Danger（合并危险）** 判定。它是一个格式参考，不是工作流。它不会推送分支、打开 PR，也不会决定里面放什么。它告诉 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 在撰写 PR 正文时应该长什么样。

Summary 是一个可视化视图，不是一段文字。默认的 PR 正文用散文描述 diff。这个格式会挑选出能让核心观点一目了然的 **最小视图**（伪代码、调用树、组件树、文件树、Mermaid 图或结构化 diff），并将其周围的文字精简到最少。审阅者已经打开了 diff，所以正文先给他们看变更的「形状」，再让他们去读细节。

## 何时使用它

输入 `/pr`，或者在 agent 撰写 PR 正文时，它会自动调用这个技能。

| 你的情况                  | 选择                                                             |
| --------------------- | -------------------------------------------------------------- |
| 分支已就绪，需要一个审阅者能快速浏览的正文 | `pr`                                                           |
| 代码已写完但还没人审阅           | [code-review](https://aihero.dev/skills-code-review)先做，然后 `pr` |
| PR 已打开，审阅意见陆续回来       | 本技能集暂无对应项； `pr`\`pr\` 只负责写正文                                   |

## 模板

三个部分，按此顺序：

* **Summary**：一个或多个小型可视化视图，每个视图旁配简短文字说明。用一个，有时用几个，极少全用。只保留审阅者需要的调用、文件、属性和边界。
* **Evidence**：前后对比。当变更涉及 UI 且 [environment](https://www.aihero.dev/ai-coding-dictionary/environment) 能截图时，截图是最强证据；否则用原本失败、现已通过的确切测试（写成伪代码），或变化了的控制台输出。
* **Merge Danger**：判定该变更是 **one-way door（单向门）** 还是 **two-way door（双向门）**，并给出 **blast radius（影响半径）**。双向门撤销成本低；单向门（破坏性迁移、公共 API 移除、难以逆转的决策）则不然。影响半径列出变更出错时可能受损的东西：布局偏移、API 消费者、移动端响应式布局等。

门的判定是核心观点。它把「能不能合？」从凭感觉变成了一个明确的声明，让审阅者可以反驳。它还告诉审阅者该把 [human review](https://www.aihero.dev/ai-coding-dictionary/human-review) 精力花在哪里：双向门且影响半径小的略读，单向门的细读。

## 常见问题

**能信 agent 自己判的门吗？**

不能盲信，这才是要把判定写出来的原因。写代码的 agent 自己给自己打分，所以当它说「双向门、影响半径小」时最该警惕。Diff 本身也常常藏着能否逆转的信息。有用户说得好：「回滚提交撤不回已经发出去的一批邮件」，带旗标的灰度发布也只在第一次写入新格式前是双向门。技能给 agent 一个定义（破坏性动作和难以逆转的决策是单向门），而不是核对清单，所以最仔细地核对 Merge Danger 那一行。两件事有帮助：确保 agent 面前有 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 或 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，不光是 diff；把你们仓库始终视为单向门的变更（schema 迁移、任何对外发布或删除的东西）写下来，放在 agent 做判定前会读到的位置。

**它能帮我把 PR 打开吗？**

不能。`pr` 只管正文。[implement](https://aihero.dev/skills-implement) 以提交到当前分支结束。关于「打开 PR」的技能或选项（`/to-pr`，或让 `implement` 直接开 PR 而不是提交）目前仍是开放提议。有用户的变通办法是给 `implement` 加一句本地覆盖指令，让它去开 PR。[implement-spec](https://aihero.dev/skills-implement-spec) 是例外：当你的事务系统通过 PR 关闭工单、或你显式要求时，它会打开一个草稿 PR。因为 `pr` 是模型调用的，所以无论何时你让 agent 开 PR，它都会用这个结构生成正文。

**它会不会又产生一堆文字和图？**

技能设计初衷是防止这种情况，但仍可能发生。用户对 agent 写的 PR 正文意见一致：「一大段摘要，我只想知道改了什么、怎么验证的、什么可能坏、安不安全合并。」技能指示 agent 跳过前言、精简文字、选最小视图，通常只要一个视图，极少全上。如果还是给了一堆图，说明 agent 忽视了指示。如果正文巨大是因为 diff 巨大，那问题出在 PR 太大，`pr` 不会帮你拆分。

**我仓库已经有 PR 模板了。听谁的？**

默认都不听。`pr` 自带模板，不会去找 `.github/pull_request_template.md` 之类的文件。有多位用户希望技能能用仓库的模板。如果你不干预，agent 会对同一文档收到两套相互竞争的指示。在仓库的 agent 文档里把这事定下来，比如先填仓库模板，再把 Summary、Evidence、Merge Danger 放在其下。

**输出是 HTML 吗？为什么 Mermaid 图不渲染？**

输出是 Markdown 格式的 PR 正文，不是 HTML 页面。GitHub 和 GitLab 在 PR 描述里会渲染 Mermaid 代码块，但终端不会，所以 agent 在本地展示给你时，图表会显示为原始文本。用 CLI 调用的用户有两种变通：用 ASCII Mermaid 渲染器，或让 agent 把 HTML 版本作为附件挂到 PR 上。Mermaid 只是六种视图之一。调用树、文件树、结构化 diff 在任何地方都能直接读。

**它能帮我梳理回来的审阅意见吗？**

不能。它写完正文就停了。用户不止一次提过，想要一种方式来分拣其他开发者或审阅机器人的评论（哪些值得处理，哪些是非问题）。这个技能不做这事。

**它能在 PR 变更时保持正文最新吗？**

不能。它在某一时间点写一次正文，PR 在审阅中继续变更会让正文过时。大改动后让 agent 重写正文即可。agent 再次写 PR 正文，自然还是用同一个结构。

**我的改动没有 UI。Evidence 放什么？**

除截图外的一切。截图只有在变更是视觉类时才是最强证据。对于迁移、后台任务、重构，证据是原本失败、现已通过的确切测试，或变化了的控制台输出。「测试全绿」本身只是个断言，不是前后对比。

**它能在正文里标记「由 LLM 撰写」吗？**

自身不能。有用户的做法是在仓库的 agent 文档里立一条常规指令：agent 写的每个 issue、评论、PR 结尾都要加一行披露声明。这条规则属于仓库级别，能覆盖 agent 发布的所有内容，而不是只在一种文档的模板里生效。

## 如果它起作用了

* 只看 Summary 视图，不打开 diff 就能知道 PR 改了什么。
* 正文无前言：直接从 Summary 标题开始。
* Evidence 部分给出前后对比，而不是「测试通过」这种断言。
* 每个 PR 都给出门的判定和影响半径，单向门是你放慢脚步仔细看的那些。

## 它在系统中的位置

当构建以 PR 形式发布时，`pr` 位于 review 和 retro 之间：`to-spec → to-tickets → implement → code-review → pr → retro`。它是模型调用的，所以任何时候 agent 在该链路之外写 PR 正文，也会自动触发。

* [code-review](https://aihero.dev/skills-code-review) 在它之前运行，因为 PR 正文应描述一个已被审阅过的 diff。
* [implement](https://aihero.dev/skills-implement) 产出正文所描述的提交。

[ask-matt](https://aihero.dev/skills-ask-matt) 会路由到整个技能集合，当你不确定当前情况需要哪个技能时。
