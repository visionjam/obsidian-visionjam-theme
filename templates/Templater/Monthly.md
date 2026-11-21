<%*
// ===== 月记锚定月份:默认本月,可补写上上月/任意月(回车=本月) =====
let an = moment();   // 锚定时刻:决定月窗口、周回溯、最终文件名
const rawM = await tp.system.prompt("月记月份(YY.MM,回车=本月)", an.format("YY.MM"));
const reM = rawM && rawM.match(/^(\d{2})\.(\d{2})$/);
if (reM) {
  an = moment(`${2000 + Number(reM[1])}-${reM[2]}-01`, "YYYY-MM-DD");
} else if (rawM) {
  new tp.obsidian.Notice(`⚠️ 月份格式无效,已回退本月 ${an.format("YY.MM")}`, 8000);
}
%>
---
创建日期: <% tp.file.creation_date("YYYY.MM.DD HH:mm") %>
月份: <% an.format("YYYY.MM") %>
tags:
  - Monthly
---
最后修改时间：`= dateformat(this.file.mtime, "yyyy.MM.dd HH:mm:ss")`
本年对应年记：[[Calendar/Journals/Yearly/<% an.format("YY") %>|📅 本年年记]]

---
# 本月周记

<%*
// 本月包含的周（含跨月边界周），用于向下回溯周记
const monthStart = an.clone().startOf("month");
const monthEnd = an.clone().endOf("month");
const cur = monthStart.clone().startOf("isoWeek");
while (cur.isSameOrBefore(monthEnd)) {
  const gg = cur.format("GG");
  const ww = cur.format("WW");
  const range = `${cur.format("MM.DD")}-${cur.clone().add(6, "day").format("MM.DD")}`;
  tR += `- [[Calendar/Journals/Weekly/${gg}.W${ww}|📅 ${range} W${ww}]]\n`;
  cur.add(7, "day");
}
%>
---
# 笔记主体：


---

# 任务结转

>完成任务了吗？如果没有的话，是为什么呢？
>有没有可以改进的地方呢？
---
# 上月回顾

**总体满意度:** `<% tp.system.prompt("满分10分，你给这个月打几分？") %>`

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
- **下月改进:**
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

## 营养月报

```dataviewjs
// 月报窗口锚定本笔记标题 YY.MM(如 26.09):过期月记重开仍显示该月数据;标题不匹配时回退当前月
const mt = dv.current().file.name.match(/^(\d{2})\.(\d{2})$/);
let mStart = dv.luxon.DateTime.now().startOf("month");
if (mt) {
  const a = dv.luxon.DateTime.fromObject({ year: 2000 + Number(mt[1]), month: Number(mt[2]) });
  if (a.isValid) mStart = a;
}
const mEnd = mStart.endOf("month");
const f = d => d.toFormat("yyyy-LL-dd");
const cfg = dv.page("Calendar/Health/Config");
const kcalT = Number(cfg?.["热量目标"]), protT = Number(cfg?.["蛋白质目标"]);
const rec = dv.pages('"Calendar/Health"')
  .where(p => { const k = String(p["日期"]).slice(0, 10); return p["热量"] && k >= f(mStart) && k <= f(mEnd); })
  .sort(p => String(p["日期"]), "asc");
const n = rec.length;
if (n === 0) {
  dv.paragraph("本月暂无健康数据。录入见 [[Calendar/Health/Config|🎯 目标]] 页说明。");
} else {
  let kcalSum = 0;
  rec.forEach(p => { kcalSum += Number(p["热量"]); });
  const kcalAvg = Math.round(kcalSum / n);
  const w = rec.filter(p => p["体重"] != null).sort(p => String(p["日期"]), "asc");
  let wSum = 0;
  w.forEach(p => { wSum += Number(p["体重"]); });
  const wAvg = w.length ? (wSum / w.length).toFixed(1) : "—";
  const wDelta = w.length >= 2 ? (Number(w[w.length - 1]["体重"]) - Number(w[0]["体重"])).toFixed(1) : "—";
  const kcalHit = rec.filter(p => Math.abs(Number(p["热量"]) - kcalT) <= kcalT * 0.1).length;
  const protHit = rec.filter(p => p["蛋白质"] != null && Number(p["蛋白质"]) >= protT).length;
  dv.paragraph(
    `**记录**: ${n} 天 ｜ **日均摄入**: ${kcalAvg} kcal(目标 ${kcalT},±10% 内 ${kcalHit}/${n} 天)` +
    ` ｜ **平均体重**: ${wAvg} kg` +
    (wDelta !== "—" ? `(月内净变 ${wDelta} kg)` : "") +
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
# 下月计划

## 核心目标
- [ ] 
- [ ] 

## 重点行动
- [ ] 

---
# 相关笔记


---

# 本月创建的文件
```dataview
TABLE file.ctime AS "创建时间"
WHERE file.cday.month = date(now).month
  AND file.cday.year = date(now).year
SORT file.cday DESC
```
# 本月修改的文件
```dataview
TABLE file.mtime AS "🕐 修改时间"
WHERE file.mday.month = date(now).month
  AND file.mday.year = date(now).year
SORT file.mtime DESC
```
<%*
// ===== 落位：把新笔记移到目标目录，并按规则改名 =====
// ⚠️ 本段在 11 个模板中各有一份完全相同的副本；改动必须 11 处同步（见 README/模板系统-设计 §3）
// ⚠️ 坑：tp.file.path() 返回「系统绝对路径 + 扩展名」，与 vault 相对路径比较恒为不等 → 必须用 tp.file.path(true)
const TARGET_DIR = "Calendar/Journals/Monthly";   // ← 每个模板只改这一行
const RENAME_TO  = an.format("YY.MM");   // ← 有日期命名规则的模板改这里（null = 沿用笔记标题）

const currentPath = tp.file.path(true);   // 当前文件在 vault 内的相对路径（含 .md）

// ① 决定最终文件名
let finalName = RENAME_TO;
if (!finalName) {
  finalName = tp.file.title;
  if (/^Untitled( \d+)?$/.test(finalName)) {   // 标题还是默认的 Untitled → 问用户
    const answer = await tp.system.prompt("这篇笔记叫什么？");
    finalName = (answer && answer.trim()) ? answer.trim() : "";   // 取消或留空 → 放弃落位
  }
}

if (!finalName) {
  new tp.obsidian.Notice("已取消落位，文件保持在当前位置", 5000);
} else {
  const targetPath = TARGET_DIR + "/" + finalName;   // 不含扩展名，给 tp.file.move 用
  const targetFile = targetPath + ".md";             // 含扩展名，给路径查询用

  // ② 已经在正确位置 → 不再移动，交给 Templater 写内容
  if (currentPath === targetFile) {
    // 什么都不做
  } else {
    // ⚠️ 用 getAbstractFileByPath 而不是 find_tfile：
    //    find_tfile 是「链接解析」语义（按文件名找候选再比对路径），
    //    存在性判断需要精确路径，避免同名文件造成误判。
    const existing = tp.app.vault.getAbstractFileByPath(targetFile);

    if (!existing) {
      // ③ 目标不存在 → 直接移入 + 改名
      await moveTo(targetPath);
    } else if (existing.stat && existing.stat.size === 0) {
      // ④ 目标是 0 字节空壳（上次模板中断留下的）→ 删掉空壳再落位
      //    0 字节没有内容可丢，直接永久删除，不制造 .trash 残骸
      try {
        await tp.app.vault.delete(existing);
        await moveTo(targetPath);
      } catch (e) {
        new tp.obsidian.Notice("⚠️ 空壳清理失败：" + e.message + "\n请手动删除 " + targetFile + " 后重试", 8000);
      }
    } else {
      // ⑤ 目标已有内容 → 提示 + 跳过去 + 把本次新建的文件丢进回收站
      new tp.obsidian.Notice("✅ " + finalName + " 已存在，已为你打开", 5000);
      const currentFile = tp.app.vault.getAbstractFileByPath(currentPath);
      const cleanup = async () => {
        try {
          await tp.app.workspace.getLeaf(false).openFile(existing);
          if (currentFile && currentFile.path !== existing.path) {
            await tp.app.vault.trash(currentFile, false);
          }
        } catch (e) {
          new tp.obsidian.Notice("已打开 " + finalName + "，但临时文件清理失败，请手动删除 " + currentPath, 8000);
        }
      };
      // 必须等模板渲染结束后再清理：要删的正是「当前正在渲染的文件」
      if (tp.hooks && tp.hooks.on_all_templates_executed) tp.hooks.on_all_templates_executed(cleanup);
      else setTimeout(cleanup, 500);
    }
  }
}

// 统一的落位动作：一步完成「移入目录 + 改名」
async function moveTo(destination) {
  try {
    await tp.file.move(destination);
  } catch (e) {
    new tp.obsidian.Notice("⚠️ 落位失败：" + e.message + "\n文件仍在 " + currentPath + "，请手动移动。", 8000);
  }
}
%>
