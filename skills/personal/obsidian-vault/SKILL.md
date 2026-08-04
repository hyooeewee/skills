---
name: obsidian-vault
description: 使用维基链接和索引笔记在 Obsidian 仓库中搜索、创建和管理笔记。当用户想要在 Obsidian 中查找、创建或整理笔记时使用。

---

# Obsidian 仓库

## 仓库位置

`/mnt/d/Obsidian Vault/AI Research/`

主要保持根目录下的扁平化结构。

## 命名约定

* **索引笔记**：聚合相关主题（例如 `Ralph Wiggum Index.md`、`Skills Index.md`、`RAG Index.md`）
* 所有笔记名称使用首字母大写
* 不使用文件夹进行组织 - 而是使用链接和索引笔记

## 链接

* 使用 Obsidian 的 `[[wikilinks]]` 语法：`[[Note Title]]`
* 笔记在底部链接到依赖项/相关笔记
* 索引笔记只是 `[[wikilinks]]` 的列表

## 工作流程

### 搜索笔记

```bash
# Search by filename
find "/mnt/d/Obsidian Vault/AI Research/" -name "*.md" | grep -i "keyword"

# Search by content
grep -rl "keyword" "/mnt/d/Obsidian Vault/AI Research/" --include="*.md"
```

或者直接在仓库路径上使用 Grep/Glob 工具。

### 创建新笔记

1. 文件名使用 **首字母大写**
2. 将内容作为学习单元编写（遵循仓库规则）
3. 在底部添加 `[[wikilinks]]` 到相关笔记
4. 如果属于编号序列的一部分，请使用分层编号方案

### 查找相关笔记

在整个仓库中搜索 `[[Note Title]]` 以查找反向链接：

```bash
grep -rl "\\[\\[Note Title\\]\\]" "/mnt/d/Obsidian Vault/AI Research/"
```

### 查找索引笔记

```bash
find "/mnt/d/Obsidian Vault/AI Research/" -name "*Index*"
```
