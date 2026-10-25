---
cssclasses:
  - dashboard
---

> [!info] 示例主页
> 这段示例配合 `snippets/VisionJam-Homepage.css` 使用。把 `cssclasses: dashboard` 放在你自己的主页 frontmatter 里，再按需改下面的查询和链接。

```dataviewjs
const now = new Date();
const h = now.getHours();
const greeting = h < 12 ? "🌅 早上好" : h < 18 ? "☀️ 下午好" : "🌙 晚上好";
const dateStr = now.toLocaleDateString("zh-CN", {
  year: "numeric",
  month: "2-digit",
  day: "2-digit",
  weekday: "long"
});
dv.paragraph(
  `<div class="vj-banner">` +
  `<div class="vj-banner-greet">${greeting}，Vision Jam 👋</div>` +
  `<div class="vj-banner-date">${dateStr}</div>` +
  `</div>`
);
```

## 📊 统计概览

```dataviewjs
const all = dv.pages().where(p => !p.file.folder.startsWith("."));
const today = dv.date("today");
const cards = [
  ["总笔记", all.length],
  ["今日新建", all.where(p => p.file.ctime >= today.startOf("day")).length],
  ["本周新建", all.where(p => p.file.ctime >= today.startOf("week")).length],
  ["本月新建", all.where(p => p.file.ctime >= today.startOf("month")).length]
];
dv.paragraph(
  '<div class="vj-stats">' +
  cards.map(([label, value]) =>
    `<div class="vj-stat"><span class="vj-stat-value">${value}</span><span class="vj-stat-label">${label}</span></div>`
  ).join("") +
  "</div>"
);
```

## 📈 数据面板

### 📊 内容分布

```dataviewjs
const pages = dv.pages().where(p => !p.file.folder.startsWith("."));
const counts = {};
for (const p of pages) {
  const parts = p.file.folder.split("/").filter(Boolean);
  const category = parts.length ? parts[0] : "Root";
  counts[category] = (counts[category] || 0) + 1;
}
const rows = Object.entries(counts).sort((a, b) => b[1] - a[1]);
const max = Math.max(...rows.map(r => r[1]), 1);
dv.paragraph(
  '<div class="vj-bars">' +
  rows.map(([label, count]) =>
    `<div class="vj-bar-row">` +
    `<span class="vj-bar-label">${label}</span>` +
    `<div class="vj-bar-track"><div class="vj-bar-fill" style="width:${Math.round(count / max * 100)}%"></div></div>` +
    `<span class="vj-bar-value">${count}</span>` +
    `</div>`
  ).join("") +
  "</div>"
);
```

### 📈 月度新建趋势

```dataviewjs
const pages = dv.pages().where(p => !p.file.folder.startsWith("."));
const counts = {};
for (const p of pages) {
  const month = p.file.ctime.toFormat("yyyy-MM");
  counts[month] = (counts[month] || 0) + 1;
}
const rows = Object.entries(counts).sort((a, b) => a[0].localeCompare(b[0]));
const max = Math.max(...rows.map(r => r[1]), 1);
dv.paragraph(
  '<div class="vj-bars">' +
  rows.map(([label, count]) =>
    `<div class="vj-bar-row">` +
    `<span class="vj-bar-label">${label}</span>` +
    `<div class="vj-bar-track"><div class="vj-bar-fill vj-bar-cyan" style="width:${Math.round(count / max * 100)}%"></div></div>` +
    `<span class="vj-bar-value">${count}</span>` +
    `</div>`
  ).join("") +
  "</div>"
);
```

## 📥 快速入口

[[examples/bases-daily|📅 日记索引]]
[[examples/bases-calendar|🗓️ 日历视图]]

## 🔄 最近 7 天

```dataview
TABLE file.folder AS "位置"
WHERE file.ctime >= date(today) - dur(7 days)
SORT file.ctime DESC
LIMIT 8
```
