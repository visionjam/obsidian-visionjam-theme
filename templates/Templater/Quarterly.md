<%*
// ===== 季度锚定:默认本季度,可补写过去/任意季度(回车=本季度) =====
let an = moment();   // 锚定时刻:决定季内月记链接与最终文件名
const rawQ = await tp.system.prompt("季度(YY.Qn,回车=本季度)", an.format("YY") + ".Q" + an.quarter());
const reQ = rawQ && rawQ.match(/^(\d{2})\.Q([1-4])$/);
if (reQ) {
  const yy = 2000 + Number(reQ[1]);
  const mm = (Number(reQ[2]) - 1) * 3 + 1;
  an = moment(`${yy}-${String(mm).padStart(2, "0")}-01`, "YYYY-MM-DD");
} else if (rawQ) {
  new tp.obsidian.Notice(`⚠️ 季度格式无效,已回退本季度 ${an.format("YY")}.Q${an.quarter()}`, 8000);
}
-%>
---
创建日期: <% tp.file.creation_date("YYYY.MM.DD HH:mm") %>
季度: <% an.format("YYYY") %> Q<% an.quarter() %>
aliases:
tags:
  - Quarterly
---
最后修改时间：`= dateformat(this.file.mtime, "yyyy.MM.dd HH:mm:ss")`
本年对应年记：[[Calendar/Journals/Yearly/<% an.format("YY") %>|📅 本年年记]]

<%*
// 本季度三个月份(用于下方月记链接)
const q = an.quarter();
const yy = an.format("YY");
const startMonth = (q - 1) * 3 + 1;
const q1 = yy + "." + String(startMonth).padStart(2, "0");
const q2 = yy + "." + String(startMonth + 1).padStart(2, "0");
const q3 = yy + "." + String(startMonth + 2).padStart(2, "0");
%>

---
# 本季度月记

- [[Calendar/Journals/Monthly/<% q1 %>|📅 <% startMonth %> 月月记]]
- [[Calendar/Journals/Monthly/<% q2 %>|📅 <% startMonth + 1 %> 月月记]]
- [[Calendar/Journals/Monthly/<% q3 %>|📅 <% startMonth + 2 %> 月月记]]

---
# 任务结转

>完成任务了吗？如果没有的话，是为什么呢？
>有没有可以改进的地方呢？
---
# 上季度回顾

**总体满意度:** `<% tp.system.prompt("满分10分，你给这个季度打几分？") %>`

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
- **下季度改进:**
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
# 下季度计划

## 核心目标
- [ ] 
- [ ] 

## 重点行动
- [ ] 

---
# 相关笔记


---
# 本季度创建的文件
```dataview
TABLE file.ctime AS "创建时间"
WHERE file.cday.year = date(now).year
  AND floor((file.cday.month - 1) / 3) = floor((date(now).month - 1) / 3)
SORT file.cday DESC
```
# 本季度修改的文件
```dataview
TABLE file.mtime AS "🕐 修改时间"
WHERE file.mday.year = date(now).year
  AND floor((file.mday.month - 1) / 3) = floor((date(now).month - 1) / 3)
SORT file.mtime DESC
```

<%*
// ===== 内容写完,自动落位 =====
// ⚠️ 本段在 11 个模板中各有一份完全相同的副本;改动必须 11 处同步(见 templates/README.md 的「落位脚本」一节)
// ⚠️ 坑:tp.file.path() 返回「系统绝对路径 + 扩展名」,与 vault 相对路径比较恒为不等 → 必须用 tp.file.path(true)
const TARGET_DIR = "Calendar/Journals/Quarterly";   // ← 每个模板改这一行
const RENAME_TO  = an.format("YY") + ".Q" + an.quarter();   // ← 有日期命名规则的模板改这里(null = 沿用笔记标题)

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
