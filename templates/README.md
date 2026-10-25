# Templater 模板

这是《我如何使用 Obsidian》里那 11 个 Templater 模板，按文章中的目录结构写：

| 模板 | 落到 | 命名 |
| --- | --- | --- |
| `Daily.md` | `Calendar/Journals/Daily/` | `YY.MM.DD` |
| `Weekly.md` | `Calendar/Journals/Weekly/` | `YY.Www` |
| `Monthly.md` | `Calendar/Journals/Monthly/` | `YY.MM` |
| `Quarterly.md` | `Calendar/Journals/Quarterly/` | `YY.Qn` |
| `Yearly.md` | `Calendar/Journals/Yearly/` | `YY` |
| `Health.md` | `Calendar/Health/` | `YY.MM.DD` |
| `Blogs.md` | `Calendar/Blogs/` | 标题 |
| `Fleetings.md` | `Calendar/Fleetings/` | 标题 |
| `Peoples.md` | `Peoples/` | 标题 |
| `Books.md` | `Calendar/Books/` | 标题 |
| `Videos.md` | `Calendar/Videos/` | 标题 |

## 依赖

- Templater 插件；
- Dataview 插件（模板里的周期链接和健康统计用 Dataview 内联查询）；
- `Daily.md` 末尾调用了 `tp.web.daily_quote()`，会联网取一句引言；不想要就删掉那一行。

## 安装

1. 把 `Templater/` 整个目录复制到你的 vault 根目录（或合并进你已有的 `Templater/`）。
2. 逐个打开模板，把落位脚本里的 `TARGET_DIR` 改成你自己的目录；如果你不用 `YY.MM.DD` 这类命名，把 `RENAME_TO` 一起改。
3. 在 Templater 设置里配置文件夹模板：
   - `Calendar/Journals/Daily` → `Templater/Daily.md`
   - `Calendar/Health` → `Templater/Health.md`
4. 打开 Templater 的 “Trigger Templater on new file creation”（每台设备都要开一次）。

## 落位脚本

每个模板末尾都有一份相同的落位脚本，只有 `TARGET_DIR` 和 `RENAME_TO` 两行不同。它做四件事：

1. 文件已经在正确目录和名字 → 跳过；
2. 目标文件已存在 → 打开它，并清理这次新建的临时文件；
3. 文件名还是 `Untitled` → 弹窗问名字；
4. 落位失败 → 用 Notice 提示，不把错误写进正文。

11 个模板是 11 份副本，改逻辑时要 11 处同步。这是有意的取舍：单文件可读，不依赖共享 user script。

## 已知坑

- **必须用 `tp.file.path(true)`**：`tp.file.path()` 返回系统绝对路径带 `.md`，和 vault 相对路径比较永远不相等。
- **文件夹触发的开关不随 Git 同步**：映射表在 `data.json` 里，会同步；总开关在 Obsidian 的 localStorage 里，每台设备各存一份且默认关闭。换手机后要手动去设置里开一次。
- **`tp.hooks.on_all_templates_executed`**：清理临时文件必须等模板渲染结束后做，否则删的正是当前正在渲染的文件。
- 模板假设你使用文章里的目录结构；目录不同就改 `TARGET_DIR`，不要直接照抄。

## 许可

MIT，和仓库其他原创代码一致。随便改，保留 `LICENSE` 即可。
