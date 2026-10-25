---
cssclasses:
  - bases-index
---

# 日记索引

> [!info] 示例索引页
> 这个页面配合 `snippets/VisionJam-Bases.css` 使用。把 `cssclasses: bases-index` 放在 frontmatter 里，Dataview 表格就会套上卡片式样式。

```dataview
TABLE file.tags AS "标签"
WHERE contains(file.tags, "Daily")
SORT file.name DESC
```
