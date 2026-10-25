<%*
// ===== 年份锚定:默认今年,可补写过去/任意年份(回车=今年) =====
let an = moment();   // 锚定时刻:决定四个季度记链接与最终文件名
const rawY = await tp.system.prompt("年份(YY,回车=今年)", an.format("YY"));
const reY = rawY && rawY.match(/^(\d{2})$/);
if (reY) {
  an = moment(`${2000 + Number(reY[1])}-01-01`, "YYYY-MM-DD");
} else if (rawY) {
  new tp.obsidian.Notice(`⚠️ 年份格式无效,已回退今年 ${an.format("YY")}`, 8000);
}
-%>
---
创建日期: <% tp.file.creation_date("YYYY.MM.DD HH:mm") %>
年份: <% an.format("YYYY") %>
aliases:
tags:
  - Yearly
---
最后修改时间：`= dateformat(this.file.mtime, "yyyy.MM.dd HH:mm:ss")`

---
# 本年度季度记

- [[Calendar/Journals/Quarterly/<% an.format("YY") %>.Q1|📅 Q1 季度记]]
- [[Calendar/Journals/Quarterly/<% an.format("YY") %>.Q2|📅 Q2 季度记]]
- [[Calendar/Journals/Quarterly/<% an.format("YY") %>.Q3|📅 Q3 季度记]]
- [[Calendar/Journals/Quarterly/<% an.format("YY") %>.Q4|📅 Q4 季度记]]

---
# 任务结转

>完成任务了吗？如果没有的话，是为什么呢？
>有没有可以改进的地方呢？
---
# 上年度回顾

**总体满意度:** `<% tp.system.prompt("满分10分，你给今年打几分？") %>`

## 关键成果 ✅
- 

## 挑战与反思 🔍
- 

## 经验与教训 🔑
- 

---
# 详情复盘

## 1. 工作与事业
- **完成事项:**
- **未完成与障碍:**
- **明年改进:**
## 2. 学习与成长
- **阅读/课程:**
- **新技能/知识:**
- **灵感瞬间:**
## 3. 生活与健康
- **身体状态:**
- **情绪状态:**
- **人际关系:**

---
# 客观数据分析

## 习惯追踪
- 

## 时间投入分析
- 

---
# 明年计划

## 核心目标
- [ ] 
- [ ] 

## 重点行动
- [ ] 

---
# 相关笔记


---
# 本年度创建的文件
```dataview
TABLE file.ctime AS "创建时间"
WHERE file.cday.year = date(now).year
SORT file.cday DESC
```
# 本年度修改的文件
```dataview
TABLE file.mtime AS "🕐 修改时间"
WHERE file.mday.year = date(now).year
SORT file.mtime DESC
```

<%*
// ===== 内容写完,自动落位 =====
// ⚠️ 本段在 11 个模板中各有一份完全相同的副本;改动必须 11 处同步(见 templates/README.md 的「落位脚本」一节)
// ⚠️ 坑:tp.file.path() 返回「系统绝对路径 + 扩展名」,与 vault 相对路径比较恒为不等 → 必须用 tp.file.path(true)
const TARGET_DIR = "Calendar/Journals/Yearly";   // ← 每个模板改这一行
const RENAME_TO  = an.format("YY");   // ← 有日期命名规则的模板改这里(null = 沿用笔记标题)

const currentRel = tp.file.path(true);          // 当前文件的 vault 相对路径(带 .md)

// ① 决定最终文件名
let finalName = RENAME_TO;
if (!finalName) {
  finalName = tp.file.title;
  if (/^Untitled( \d+)?$/.test(finalName)) {    // 还没起名 → 问
    const ans = await tp.system.prompt("这篇笔记叫什么？");
    finalName = (ans && ans.trim()) ? ans.trim() : "";   // 取消/留空 → 置空
  }
}

if (!finalName) {
  new tp.obsidian.Notice("已取消落位,文件保持在当前位置", 5000);
} else {
  const targetPath = `${TARGET_DIR}/${finalName}`;
  const targetFile = `${targetPath}.md`;

  // ② 已在正确位置 → 什么都不做
  if (currentRel !== targetFile) {
    const existing = tp.file.find_tfile(targetFile);
    if (existing) {
      // ③ 目标已存在 → 提示 + 跳过去 + 清理本次新建的文件
      const stale = tp.file.find_tfile(currentRel);
      new tp.obsidian.Notice(`✅ ${finalName} 已存在,已为你打开`, 5000);
      const cleanup = async () => {
        try {
          await tp.app.workspace.getLeaf(false).openFile(existing);
          if (stale && stale.path !== existing.path) await tp.app.vault.trash(stale, false);
        } catch (e) {
          new tp.obsidian.Notice(`已打开 ${finalName},但临时文件清理失败,请手动删除 ${currentRel}`, 8000);
        }
      };
      // 必须等模板渲染结束后再清理:被删的正是"当前正在渲染的文件"
      if (tp.hooks && tp.hooks.on_all_templates_executed) tp.hooks.on_all_templates_executed(cleanup);
      else setTimeout(cleanup, 500);
    } else {
      // ④ 落位:一步完成「移入目录 + 改名」
      try {
        await tp.file.move(targetPath);
      } catch (e) {
        new tp.obsidian.Notice(`⚠️ 落位失败:${e.message}\n文件仍在 ${currentRel},请手动移动。`, 8000);
      }
    }
  }
}
%>
