## 它做什么

`pr` 是 Pull Request 正文应有的形态：一个展示变更的 **Summary（摘要）**、一个证明其可行的 **Evidence（证据）**、以及一个关于合并风险等级的 **Merge Danger（合并危险）** 判定。它是格式参考，而非工作流。它不会推送分支、打开 PR，也不会决定里面放什么；它只是告诉 [agent](https://www.aihero.dev/ai-coding-dictionary/agent) 在撰写 PR 正文时，正文该长什么样。

Summary 是一幅图，不是一段文字。默认 PR 正文用散文叙述 diff，而这份格式会挑出能讲清要点的 **最小视图**（伪代码、调用树、组件树、文件树、Mermaid 图，或结构化 diff），并将其周围的文字压到最短。审阅者已经打开了 diff；正文的任务是让他们在细读前先看到变更的“形状”。

## 何时使用它

输入 `/pr`，或者在 agent 撰写 PR 正文时，它会自动调用此技能。

| 你的情况                  | 选择                                                             |
| --------------------- | -------------------------------------------------------------- |
| 分支已就绪，需要一份审阅者能快速浏览的正文 | `pr`                                                           |
| 代码已写完，但还没人审阅          | [code-review](https://aihero.dev/skills-code-review)先做，然后 `pr` |
| PR 已打开，评审意见陆续回来       | 本技能集暂无对应项； `pr`\`pr\` 仅撰写正文                                    |

## 模板

三个部分，顺序固定：

* **Summary（摘要）**：一个或多个小型可视化，每个紧贴其支撑的短文本。用一个，有时用几个，极少全用。只保留审阅者需要的调用、文件、属性和边界。
* **Evidence（证据）**：前后对比。当变更涉及视觉且 [environment](https://www.aihero.dev/ai-coding-dictionary/environment) 可截图时，截图是最强证据；否则用之前失败、现在通过的确切测试（写成伪代码），或变化后的控制台输出。
* **Merge Danger（合并危险）**：变更是 **one-way door（单向门）** 还是 **two-way door（双向门）**，以及其 **blast radius（影响半径）**。双向门回滚成本低；单向门（破坏性迁移、公共 API 移除、难以逆转的决策）则不然。影响半径列出变更出错时可能波及的范围：布局偏移、API 消费者、移动端响应式等。

“门”的判定是核心。它把“能不能合？”从凭直觉变成一句审阅者可反驳的明确断言，并指引他们在 [human review](https://www.aihero.dev/ai-coding-dictionary/human-review) 上花多少精力：双向门且影响半径小可略读；单向门值得细读。

## 常见问题

**能信任 agent 自己给的门判定吗？**

不能盲信，这正是要把它写出来的原因。写代码的 agent 自己给自己打分，而自述最舒服的时候，往往就是它写着“双向门、影响半径小”。可逆性在 diff 里往往不可见：正如一位用户所说，“回滚提交无法撤回已发出的一批邮件”，而灰度发布也只在第一笔新格式写入前保持双向。技能给 agent 一个定义（破坏性动作和难以逆转的决策属于单向门），而非核对清单，所以 **Merge Danger 那一行最要仔细读**。两点有帮助：确保 agent 面前有 [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket) 或 [spec](https://www.aihero.dev/ai-coding-dictionary/spec)，而不只是 diff；把你们仓库始终视为单向门的变更（Schema 迁移、任何对外发布或删除的操作）写下来，放在 agent 做判定前能读到的地方。

**它能帮我把 PR 打开吗？**

不能。`pr` 只管正文。[implement](https://aihero.dev/skills-implement) 结束时会提交到当前分支，而把 PR 打开的技能或选项（`/to-pr`、或 `implement` 直接开 PR 而非提交）仍是开放提案；有用户的变通做法是给 `implement` 加一句本地覆盖指令，让它去开 PR。[implement-spec](https://aihero.dev/skills-implement-spec) 是例外：当你的问题追踪器通过 PR 关闭工作项，或你显式要求时，它会打开一个草稿 PR。因为 `pr` 是模型调用的，任何时候你让 agent 开 PR，其描述都会采用这个形态。

**它会不会又产生一堆文本和图表？**

这正是它要防范的失败模式，但依然可能发生。用户对 agent PR 正文的抱怨高度一致：“一大段摘要，我只想知道改了什么、怎么验证、可能坏在哪、安不安全合并。”技能要求 agent 跳过前言、精简文字、选最小视图，通常只用一个视图，极少全上。如果仍得到一堆图，说明 agent 忽略了要求；如果正文巨大是因为 diff 巨大，问题出在 PR 太大，`pr` 不会帮你拆分。

**我仓库已有 PR 模板。听谁的？**

默认谁都不听：`pr` 自带模板，不查找 `.github/pull_request_template.md` 或类似文件。多位用户希望技能遵守仓库模板，若置之不理，agent 就会同时持有两套相互冲突的指令去写同一文档。在你仓库的 agent 文档里把这事定下来，例如：先填满仓库模板，再在其下放 Summary、Evidence、Merge Danger 三节。

**输出是 HTML 吗？为什么 Mermaid 图不渲染？**

输出是 Markdown PR 正文，不是 HTML 页面。GitHub 和 GitLab 会在 PR 描述中渲染 Mermaid 块，但终端不会，所以 agent 在本地展示给你时，图表显示为原始文本。CLI 用户的变通做法：用 ASCII Mermaid 渲染器，或让 agent 把 HTML 版本作为附件加到 PR。Mermaid 只是六种视图之一；调用树、文件树、结构化 diff 在任何地方都能直接阅读。

**它能帮我筛选回来的评审意见吗？**

不能。它写完正文就停了。对其他开发者或审阅机器人的评论进行分拣（哪些值得处理、哪些是非问题）已被多次提出，但不属于本技能范围。

**PR 变更时，它能保持正文最新吗？**

不能。它在某一时间点写下正文，PR 在评审中继续变更会让正文过时。在有实质变更后让 agent 重写正文即可；它又是在写 PR 正文，所以同一形态适用。

**我的变更没有 UI。Evidence 放什么？**

除了截图都放。截图只在视觉变更时是最强证据；对迁移、后台任务、重构，证据是之前失败、现在通过的确切测试，或变化的控制台输出。“测试通过”本身只是断言，不是前后对比。

**它能在正文里标记“由 LLM 撰写”吗？**

自身不带这功能。有用户的做法是在仓库的 agent 文档里立一条规则：agent 写的每个 Issue、评论、PR 结尾都加一行披露声明。这规则属于仓库层面，能覆盖 agent 发布的所有内容，而不应只在一种文档的模板里。

## 如果它起作用了

* 只看 Summary 视图、不开 diff，就能知道 PR 改了什么。
* 正文无前言：直接从 Summary 标题开始。
* Evidence 部分展示前后对比，而不是“测试通过”这类断言。
* 每个 PR 都给出门类型和影响半径，而单向门是你放慢脚步细读的那些。

## 它在系统中的位置

当构建以 Pull Request 形式提交时，`pr` 位于 review 与 retro 之间：`to-spec → to-tickets → implement → code-review → pr → retro`。它是模型调用的，因此在该链路之外、任何 agent 撰写 PR 正文的时刻，它也会自行触发。

* [code-review](https://aihero.dev/skills-code-review) 在它之前运行，因为 PR 正文应描述已被审阅过的 diff。
* [implement](https://aihero.dev/skills-implement) 产出正文所描述的提交。

[ask-matt](https://aihero.dev/skills-ask-matt) 会路由到整个技能集合，当你不确定当前情况需要哪个技能时。
