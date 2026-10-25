<%*
// ===== 健康数据页模板:默认今天,可批量补录历史日期 =====
const isoDefault = tp.date.now("YYYY-MM-DD");
const raw = await tp.system.prompt("数据日期(YYYY-MM-DD,回车=今天)", isoDefault);
const input = (raw && raw.trim()) ? raw.trim() : isoDefault;
const re = input.match(/^(\d{4})-(\d{1,2})-(\d{1,2})$/);
let iso = isoDefault;
if (re) {
  iso = `${re[1]}-${re[2].padStart(2, "0")}-${re[3].padStart(2, "0")}`; // 归一化为 2026-09-07
} else {
  new tp.obsidian.Notice(`⚠️ 日期格式无效,已回退今天 ${iso}`, 8000);
}
const fileDate = `${iso.slice(2,4)}.${iso.slice(5,7)}.${iso.slice(8,10)}`; // 文件名用 YY.MM.DD(与日记一致)
-%>
---
tags: [Health/Data]
日期: <% iso %>
热量: 
蛋白质: 
脂肪: 
碳水: 
体重: 
---
最后修改时间：`= dateformat(this.file.mtime, "yyyy.MM.dd HH:mm:ss")`
# 🥗 健康数据 <% fileDate %>
## 早餐

热量 
蛋白质 
脂肪 
碳水 

## 中餐

热量 
蛋白质 
脂肪 
碳水 

## 晚餐

热量 
蛋白质 
脂肪 
碳水 

数据来源:AI 估算,未经食物库核对(热量/蛋白质/碳水/脂肪 = 当日总摄入;体重 = 晨起空腹称重,可选)

相关日记: [[Calendar/Journals/Daily/<% fileDate %>]]

<%*
// ===== 内容写完,自动落位 =====
// ⚠️ 本段在 11 个模板中各有一份完全相同的副本;改动必须 11 处同步(见 templates/README.md 的「落位脚本」一节)
// ⚠️ 坑:tp.file.path() 返回「系统绝对路径 + 扩展名」,与 vault 相对路径比较恒为不等 → 必须用 tp.file.path(true)
const TARGET_DIR = "Calendar/Health";   // ← 每个模板改这一行
const RENAME_TO  = fileDate;   // ← 有日期命名规则的模板改这里(null = 沿用笔记标题)

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
