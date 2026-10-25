<%*
// ===== 周记锚定周:默认本周,可补写上周/任意周(回车=本周) =====
let an = moment();   // 锚定时刻:决定周窗口、每日回溯列表、最终文件名
const rawW = await tp.system.prompt("周记周次(GG.WWW,回车=本周)", an.format("GG.WWW"));
const reW = rawW && rawW.match(/^(\d{2})\.W(\d{1,2})$/);
if (reW) {
  an = moment().isoWeekYear(2000 + Number(reW[1])).isoWeek(Number(reW[2]));
} else if (rawW) {
  new tp.obsidian.Notice(`⚠️ 周次格式无效,已回退本周 ${an.format("GG.WWW")}`, 8000);
}
-%>
---
创建日期: <% tp.file.creation_date("YYYY.MM.DD HH:mm") %>
tags:
  - Weekly
---
最后修改时间：`= dateformat(this.file.mtime, "yyyy.MM.dd HH:mm:ss")`
本月对应月记：[[Calendar/Journals/Monthly/<% an.clone().startOf("isoWeek").format("YY") %>.<% an.clone().startOf("isoWeek").format("MM") %>|📅 本月月记]]

---
# 本周每日

<%*
// 本周每日（周一到周日），用于向下回溯每日日记
const mon = an.clone().startOf("isoWeek");
const dayNames = ["一", "二", "三", "四", "五", "六", "日"];
for (let i = 0; i < 7; i++) {
  const d = mon.clone().add(i, "day");
  tR += `- [[Calendar/Journals/Daily/${d.format("YY.MM.DD")}|📅 ${d.format("MM.DD")} 周${dayNames[i]}]]\n`;
}
%>
---
# 任务结转

>完成任务了吗？如果没有的话，是为什么呢？
>有没有可以改进的地方呢？
---
# 上周回顾

**总体满意度:** `<% tp.system.prompt("满分10分，你给这周打几分？") %>`

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
- **下周改进:**
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

## 营养周报

```dataviewjs
// 周报窗口锚定本笔记标题 GG.WWW(如 26.W37):过期周记重开仍显示该周数据;标题不匹配时回退当前周
const name = dv.current().file.name;
const mt = name && name.match(/^(\d{2})\.W(\d{1,2})$/);
let mon = dv.luxon.DateTime.now().minus({ days: dv.luxon.DateTime.now().weekday - 1 }); // 显式 ISO 周一,不依赖 locale 周起始设置
if (mt) {
  const a = dv.luxon.DateTime.fromObject({ weekYear: 2000 + Number(mt[1]), weekNumber: Number(mt[2]), weekday: 1 });
  if (a.isValid) mon = a;
}
const sun = mon.plus({ days: 6 });
const f = d => d.toFormat("yyyy-LL-dd");
const cfg = dv.page("Calendar/Health/Config");
const kcalT = Number(cfg?.["热量目标"]), protT = Number(cfg?.["蛋白质目标"]);
const rec = dv.pages('"Calendar/Health"')
  .where(p => { const k = String(p["日期"]).slice(0, 10); return p["热量"] && k >= f(mon) && k <= f(sun); });
const n = rec.length;
if (n === 0) {
  dv.paragraph("本周暂无健康数据。录入见 [[Calendar/Health/Config|🎯 目标]] 页说明。");
} else {
  let kcalSum = 0;
  rec.forEach(p => { kcalSum += Number(p["热量"]); });
  const kcalAvg = Math.round(kcalSum / n);
  const w = rec.filter(p => p["体重"] != null);
  let wSum = 0;
  w.forEach(p => { wSum += Number(p["体重"]); });
  const wAvg = w.length ? (wSum / w.length).toFixed(1) : "—";
  const protHit = rec.filter(p => p["蛋白质"] != null && Number(p["蛋白质"]) >= protT).length;
  const kcalHit = rec.filter(p => Math.abs(Number(p["热量"]) - kcalT) <= kcalT * 0.1).length;
  dv.paragraph(
    `**平均摄入**: ${kcalAvg} kcal(目标 ${kcalT},±10% 内 ${kcalHit}/${n} 天)` +
    ` ｜ **平均体重**: ${wAvg} kg` +
    ` ｜ **蛋白质达标**: ${protHit}/${n} 天` +
    ` ｜ 📊 详见 [[Bases/Health]]`
  );
}
```

## 习惯追踪
- 

## 时间投入分析
- 

---
# 下周计划

## 核心目标
- [ ] 
- [ ] 

## 重点行动
- [ ] 

---
# 相关笔记


---
# 本周创建的文件
```dataview
TABLE file.ctime AS "创建时间"
WHERE file.cday.weekyear = date(now).weekyear
  AND file.cday.week = date(now).week
SORT file.cday DESC
```
# 本周修改的文件
```dataview
TABLE file.mtime AS "修改时间"
WHERE file.mday.weekyear = date(now).weekyear
  AND file.mday.week = date(now).week
SORT file.mday DESC
```

<%*
// ===== 内容写完,自动落位 =====
// ⚠️ 本段在 11 个模板中各有一份完全相同的副本;改动必须 11 处同步(见 templates/README.md 的「落位脚本」一节)
// ⚠️ 坑:tp.file.path() 返回「系统绝对路径 + 扩展名」,与 vault 相对路径比较恒为不等 → 必须用 tp.file.path(true)
const TARGET_DIR = "Calendar/Journals/Weekly";   // ← 每个模板改这一行
const RENAME_TO  = an.format("GG") + ".W" + an.format("WW");   // ← 有日期命名规则的模板改这里(null = 沿用笔记标题)

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
