<%*
// ===== 日记日期：默认今天，也可以补录历史日期 =====
const todayISO = tp.date.now("YYYY-MM-DD");
const dateInput = await tp.system.prompt("日记日期（YYYY-MM-DD，回车=今天）", todayISO);
const rawDate = (dateInput && dateInput.trim()) ? dateInput.trim() : todayISO;

// 统一成 YYYY-MM-DD；格式不对就回退今天
let dateISO = todayISO;
const dateMatch = rawDate.match(/^(\d{4})-(\d{1,2})-(\d{1,2})$/);
if (dateMatch) {
  dateISO = dateMatch[1] + "-" + String(dateMatch[2]).padStart(2, "0") + "-" + String(dateMatch[3]).padStart(2, "0");
} else {
  new tp.obsidian.Notice("⚠️ 日期格式无效，已回退今天 " + todayISO, 8000);
}

// 锚定日期：后面所有链接、文件名、每日一句都从它派生
const anchor = moment(dateISO, "YYYY-MM-DD");
const isToday = (dateISO === todayISO);

// 补录历史日期时跳过三问（那天的天气/心情/地点多半已不记得）
let mood = "", weather = "", place = "";
if (isToday) {
  mood = await tp.system.suggester(["😊 开心", "😐 一般", "😔 低落", "😴 疲惫"], ["开心", "一般", "低落", "疲惫"]);
  weather = await tp.system.prompt("今日天气如何？");
  place = await tp.system.prompt("所在位置？");
}
-%>
---
创建日期: <% tp.file.creation_date("YYYY.MM.DD HH:mm") %>
心情: <% mood %>
天气: <% weather %>
地点: <% place %>
tags:
  - Daily
---
最后修改时间：`= dateformat(this.file.mtime, "yyyy.MM.dd HH:mm:ss")`
本月对应月记：[[Calendar/Journals/Monthly/<% anchor.format("YY") %>.<% anchor.format("MM") %>|📅 本月月记]]
本周对应周记：[[Calendar/Journals/Weekly/<% anchor.format("GG") %>.W<% anchor.format("WW") %>|📅 本周周记]]
当日健康记录：[[Calendar/Health/<% anchor.format("YY") %>.<% anchor.format("MM") %>.<% anchor.format("DD") %>|🥗 今日健康数据]]

---
# 📅 日期信息
**今日**: <% anchor.format("YYYY.MM.DD") %>
**周几**: <% anchor.format("dddd") %>

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
<%*
// ===== 每日一句：本地名言库，不联网 =====
// 想加句子就在下面数组里追加一行 { text: "……", author: "……" }；同一天固定同一句
const DAILY_QUOTES = [
  { text: "认识你自己。", author: "德尔斐神庙箴言" },
  { text: "未经省察的人生不值得过。", author: "苏格拉底" },
  { text: "我思故我在。", author: "笛卡尔" },
  { text: "人是万物的尺度。", author: "普罗泰戈拉" },
  { text: "上善若水，水善利万物而不争。", author: "老子" },
  { text: "知人者智，自知者明。", author: "老子" },
  { text: "千里之行，始于足下。", author: "老子" },
  { text: "君子求诸己，小人求诸人。", author: "孔子" },
  { text: "学而不思则罔，思而不学则殆。", author: "孔子" },
  { text: "知之者不如好之者，好之者不如乐之者。", author: "孔子" },
  { text: "己所不欲，勿施于人。", author: "孔子" },
  { text: "君子和而不同，小人同而不和。", author: "孔子" },
  { text: "吾生也有涯，而知也无涯。", author: "庄子" },
  { text: "不积跬步，无以至千里；不积小流，无以成江海。", author: "荀子" },
  { text: "锲而不舍，金石可镂。", author: "荀子" },
  { text: "天行健，君子以自强不息。", author: "《周易》" },
  { text: "路漫漫其修远兮，吾将上下而求索。", author: "屈原" },
  { text: "采菊东篱下，悠然见南山。", author: "陶渊明" },
  { text: "长风破浪会有时，直挂云帆济沧海。", author: "李白" },
  { text: "会当凌绝顶，一览众山小。", author: "杜甫" },
  { text: "山重水复疑无路，柳暗花明又一村。", author: "陆游" },
  { text: "纸上得来终觉浅，绝知此事要躬行。", author: "陆游" },
  { text: "博观而约取，厚积而薄发。", author: "苏轼" },
  { text: "人生如逆旅，我亦是行人。", author: "苏轼" },
  { text: "问渠那得清如许？为有源头活水来。", author: "朱熹" },
  { text: "穷则独善其身，达则兼济天下。", author: "孟子" },
  { text: "生于忧患，死于安乐。", author: "孟子" },
  { text: "静以修身，俭以养德。", author: "诸葛亮" },
  { text: "非淡泊无以明志，非宁静无以致远。", author: "诸葛亮" },
  { text: "海纳百川，有容乃大。", author: "林则徐" },
  { text: "世上无难事，只怕有心人。", author: "中国谚语" },
  { text: "Knowing yourself is the beginning of all wisdom.", author: "Aristotle" },
  { text: "The unexamined life is not worth living.", author: "Socrates" },
  { text: "Simplicity is the ultimate sophistication.", author: "Leonardo da Vinci" },
  { text: "Well begun is half done.", author: "Aristotle" },
  { text: "The only thing we have to fear is fear itself.", author: "Franklin D. Roosevelt" },
  { text: "It always seems impossible until it is done.", author: "Nelson Mandela" },
  { text: "Imagination is more important than knowledge.", author: "Albert Einstein" },
  { text: "We are what we repeatedly do. Excellence, then, is not an act, but a habit.", author: "Will Durant" },
  { text: "Waste no more time arguing about what a good man should be. Be one.", author: "Marcus Aurelius" },
  { text: "The best way out is always through.", author: "Robert Frost" }
];

// 用「年份 × 372 + 当年第几天」做种子，保证同一天固定、跨年轮换
const quoteOfDay = DAILY_QUOTES[(anchor.year() * 372 + anchor.dayOfYear()) % DAILY_QUOTES.length];
tR += "> [!quote] " + quoteOfDay.text + "\n> — " + quoteOfDay.author;
%>
<%*
// ===== 落位：把新笔记移到目标目录，并按规则改名 =====
// ⚠️ 本段在 11 个模板中各有一份完全相同的副本；改动必须 11 处同步（见 README/模板系统-设计 §3）
// ⚠️ 坑：tp.file.path() 返回「系统绝对路径 + 扩展名」，与 vault 相对路径比较恒为不等 → 必须用 tp.file.path(true)
const TARGET_DIR = "Calendar/Journals/Daily";   // ← 每个模板只改这一行
const RENAME_TO  = anchor.format("YY.MM.DD");   // ← 有日期命名规则的模板改这里（null = 沿用笔记标题）

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
