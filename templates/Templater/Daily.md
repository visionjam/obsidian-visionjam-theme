<%*
// ===== 日记日期(默认今天,可补录历史日期) =====
const _isoDefault = tp.date.now("YYYY-MM-DD");
const _raw = await tp.system.prompt("日记日期(YYYY-MM-DD,回车=今天)", _isoDefault);
const _in = (_raw && _raw.trim()) ? _raw.trim() : _isoDefault;
const _m = _in.match(/^(\d{4})-(\d{1,2})-(\d{1,2})$/);
let _iso = _isoDefault;
if (_m) {
  _iso = `${_m[1]}-${_m[2].padStart(2, "0")}-${_m[3].padStart(2, "0")}`;
} else {
  new tp.obsidian.Notice(`⚠️ 日期格式无效,已回退今天 ${_isoDefault}`, 8000);
}
const _an = moment(_iso, "YYYY-MM-DD");     // 锚定时刻:决定全部链接与最终文件名
const _isToday = (_iso === _isoDefault);

// 补录历史日期时跳过三问(那天的天气/心情/地点多半已不记得)
let _mood = "", _weather = "", _place = "";
if (_isToday) {
  _mood = await tp.system.suggester(["😊 开心", "😐 一般", "😔 低落", "😴 疲惫"], ["开心", "一般", "低落", "疲惫"]);
  _weather = await tp.system.prompt("今日天气如何？");
  _place = await tp.system.prompt("所在位置？");
}
-%>
---
创建日期: <% tp.file.creation_date("YYYY.MM.DD HH:mm") %>
心情: <% _mood %>
天气: <% _weather %>
地点: <% _place %>
tags:
  - Daily
---
最后修改时间：`= dateformat(this.file.mtime, "yyyy.MM.dd HH:mm:ss")`
本月对应月记：[[Calendar/Journals/Monthly/<% _an.format("YY") %>.<% _an.format("MM") %>|📅 本月月记]]
本周对应周记：[[Calendar/Journals/Weekly/<% _an.format("GG") %>.W<% _an.format("WW") %>|📅 本周周记]]
当日健康记录：[[Calendar/Health/<% _an.format("YY") %>.<% _an.format("MM") %>.<% _an.format("DD") %>|🥗 今日健康数据]]

---
# 📅 日期信息
**今日**: <% _an.format("YYYY.MM.DD") %>
**周几**: <% _an.format("dddd") %>

# 🎯 今日目标


# 📝 记录与反思
## 1. 重要事件


## 2. 学习收获
- 

## 3. 感恩时刻
- 

## 4. 改进思考
- 

# 🔗 相关笔记


---
<% tp.web.daily_quote() %>
<%*
// ===== 内容写完,自动落位 =====
// ⚠️ 本段在 11 个模板中各有一份完全相同的副本;改动必须 11 处同步(见 templates/README.md 的「落位脚本」一节)
// ⚠️ 坑:tp.file.path() 返回「系统绝对路径 + 扩展名」,与 vault 相对路径比较恒为不等 → 必须用 tp.file.path(true)
const TARGET_DIR = "Calendar/Journals/Daily";   // ← 每个模板改这一行
const RENAME_TO  = _an.format("YY.MM.DD");   // ← 有日期命名规则的模板改这里(null = 沿用笔记标题)

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
