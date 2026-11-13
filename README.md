# 絆 · 绊崽档案

「絆 · 绊崽档案」是**单一种马产驹**的个人查阅/检索资料库（当前收录 **キズナ**（高情厚意 / Kizuna，netkeiba id `2010105827`）产驹 **1221 匹**，生年 2017~2025），以 **GitHub Pages 静态站**形式部署。数据由 Python 管线从 JBIS / netkeiba / studbook.jp 抓取维护，前端为 Vite 多页 + Tailwind CSS 构建，全站离线自包含。

> 本站是模板仓 `yunji-web`（云迹）的**分支站**：代码随母版同步，品牌 / 数据 / `config/` 各自维护（同步规则见 `docs/DESIGN.md` §12，改站流程见 `docs/FORK.md`）。

## 配置即站点身份（`config/` 五个文件）

| 文件 | 管什么 | 空值语义 |
|---|---|---|
| `config/sire.json` | 种马档案（ID / 名称 / studbook / 父子制覇 wins / **`progeny_years`**）+ 站点身份（`site.id` / 品牌四件套 / `domains` / **`repo`** / **`analytics`** / **`cross_site`**）+ 台账 URL | `repo` 空 = 编辑台关闭；`analytics` 空 = 统计静默；`cross_site` 空 = 对应跨站板块关闭；`ledger.url` 空 = 台账环节跳过 |
| `config/features.json` | **页面能力表**：哪些功能页挂载、哪些不挂 | 缺文件/缺键 = 全部启用；`"entries": false` = 关掉「出走马表」（侧栏入口消失、直连显示「本站未启用」） |
| `config/friend-links.json` | 侧栏「站群导航」友链数组 | 空数组 = 分组不渲染 |
| `config/video-sources.json` | B 站 UP 主 mid + 匹配正则 | **本站 `mid` 空 = 回放功能关闭**（回放列无图标、`--video` 直接退出） |
| `config/grade-table.json` | 级别（GI/GII/…）归一表 | 必须非空（构建期校验） |

字段契约写在配置文件内（每个键都有 `_about`/行内说明），消费方 = `scripts/core/sire_config.py` 与 `front/scripts/gen-site-config.mjs`。**改配置 ≠ 改代码**：页面里没有写死的站名、仓库名、统计账号、友链或跨站地址。

两处**跨站引用**（`site.cross_site`，都可选）：`jbis_rank` → 倾向统计页「种马榜走势」；`racing_calendar` → 出走马表页（本站两项都已配）。不配 = 该板块整体不出现，不会留半截或 404。

关闭某功能不用删文件、不会 404：侧栏入口消失 + 直连 URL 显示「本站未启用 · 出走马表」说明卡片；页面仍会构建，`npm run build` 的「本站身份」会列出 `capabilities 7/9（关闭: entries, editor）`。验收 = `node scripts/verify_caps.cjs`。编辑台是**双门 AND**：`features.editor` 与 `site.repo` 任一为关即关闭。

## 功能页

| 页面 | 文件 | 说明 |
|---|---|---|
| 基本信息（绊崽档案） | `pages/profile.html` | 头像/名字四槽/信息表/血统/数据概览/内嵌比赛记录；未选马显示全库概览等式 |
| 比赛记录 | `pages/races.html` | 单马履历（内嵌/独立双模式）+ 独立「全库跨马多维组合查询」（8 维筛选 + 分页） |
| 倾向统计 | `pages/stats.html` | 倾向矩阵（15 维切面）× 种马榜走势 × 年龄成绩曲线 |
| 产驹赏金 | `pages/leaderboard.html` | 赏金榜（总榜 / 自然年 / 世代 × 牡牝三态） |
| 日期统计 | `pages/datechart.html` | 天/周/月/年日历聚合（进板数/赏金/重赏标识） |
| 出走马表 | `pages/entries.html` | 跨站引 racing-calendar（`?embed=1&sire=<种马日文名>`）：自家产驹的出走表 |
| 航迹线（时间线） | `pages/timeline.html` | 里程碑事件流（级别首胜/重赏/世代新马首胜/父子制覇…） |
| 血统图 | `pages/pedigree.html` | 简约 2 代 + 完整 5 代弹窗（profile 内嵌） |
| 编辑台 | `pages/edit.html` / `edit-table.html` / `edit-timeline.html` / `edit-drafts.html` | 草稿箱模式人工修正，线上一次性提交 GitHub（PAT 解锁，mb 不开放） |

页面可开关：`config/features.json` 的能力表（关掉的页侧栏不出现、直连显示「本站未启用」，但页面仍构建）。

外壳 `index.html`：左侧导航 + iframe 内容区，响应式单断点 768px（手机收成 ☰ 抽屉）。

## 技术栈

- **前端**：Vite 多页构建 + Tailwind CSS v3（编译期），产物输出到仓库根 `dist/`；共享 JS 放 `front/public/`（Vite 原样拷贝），页面相对引用。
- **数据**：全部 JSON 产物在仓库根 `data/`（`basic.json` + `pedigree/{id}.json` + `races/{id}.json` + `timeline.json` + `datechart.json` + `stats.json`），构建时复制为 `dist/data/`，前端经 `YJ_DATA.url()` 统一解析。
- **数据管线**：Python（`scripts/`），JBIS 建档 → netkeiba/studbook 详情与成绩 → 合并回 `data/basic.json`；统一入口 `python run_update.py <策略>`。
- **部署**：GitHub Pages（`dist/` 自包含数据副本），push main 时 CI 自动 `npm run build` 并发布。

## 目录结构

```
kzn-web/
├── config/                # 站点身份全部旋钮：sire.json（种马 + site 段）· features.json（能力表）
│                          # friend-links.json（友链）· video-sources.json（B 站源，mid 空 = 关闭）· grade-table.json
├── data/                  # ★ 全部数据产物（basic.json / pedigree/ / races/ / timeline.json / datechart.json / stats.json）
├── docs/                  # ★ 全部人工文档（索引见 docs/README.md；架构单源 = docs/DESIGN.md）
├── scripts/               # Python 数据管线（core 中立层 / basic 建档 / races 竞赛 / timeline / datechart / stats / status / check_data / smoke_fetch）
├── front/                 # ★ 前端正式源码（Vite 多页 + Tailwind）
│   ├── index.html         # 外壳：左侧导航 + iframe 内容区
│   ├── pages/             # 功能子页面 + theme.css（Tailwind 输入）
│   ├── public/            # 共享 JS（i18n / selector / race-rows / yj-cache / yj-util / yj-cap / yj-edit-gate / bus / device / pedigree / editor …）
│   └── scripts/           # 构建期工具（gen-font.mjs / gen-site-config.mjs / build-races-bundle.mjs / features.cjs）
├── dist/                  # 构建产物（gitignore，CI 现场构建）
├── run_update.py          # 数据更新统一入口（10 种策略）
├── run_full_test.py       # 从 0 全量自测
├── AGENTS.md              # AI 助手工作规则（硬性约定汇总）
├── LICENSE / NOTICE       # 代码许可（MIT）/ 数据与照片版权声明
└── README.md              # 本文件
```

## 快速开始

```bash
# 前端：安装依赖 → 构建（产物到根 dist/）
cd front
npm install
npm run build

# 本地预览：端口固定（单源 = 项目根 serve.config.json），服务 dist/，自动打开首页
cd ..
python serve.py             # → http://127.0.0.1:9091/index.html
python serve.py --no-open   # 只起服务，不弹浏览器
```

> 必须用 http 访问（file:// 下 fetch 失败）；端口被占用时 `serve.py` **直接报错退出，不会换号**——四兄弟站约定 yunji `9090` / kzn `9091` / jbis-sire-rank-history `9092` / racing-calendar `9093`。
> Vite dev server（Vite 默认 `:5173`，根是 `front/`、取不到 `../data`）只做热更新，**不占本站约定端口**；看数据一律走 `python serve.py`。
> 每次改 `front/` 后 `npm run build` 再刷新验证；改数据后同样需重新构建才会进 `dist/data/`。

## 离线验收（不联网，改完就跑）

```bash
cd front && npm run build && cd ..    # 构建尾部会打印「本站身份」（repo / 统计 / 域名 / 友链 / 能力 / 种马）

node scripts/races/verify_drill.cjs           # 比赛记录页下钻（统计矩阵 → races?f= 全链路）
node scripts/races/verify_result.cjs          # 着顺三态口径（完赛 / 未完赛 / 未出走 分账）
node scripts/stats/verify_stats.cjs           # 统计总览（产物契约 + 源对账 + 页面冒烟）
node scripts/datechart/verify_datechart.cjs   # 日期图（同上）
node scripts/timeline/verify_sire_wins.cjs    # 父子制覇数据源
node scripts/verify_caps.cjs                  # 页面能力表（配置 ↔ 导航 ↔ 空态脚本 ↔ dist 对账）

python scripts/basic/build_registry.py --print-urls   # 只打印抓取 URL（不改盘、不联网）
```

六条全绿 = 代码与数据契约都没被破坏。改过 `config/features.json`、增删页面后**必须**跑 `verify_caps`。

## 新项目：换种马（单配置源）

换种马 / 新开一个应援站 = **只改 `config/` 下的 JSON，不动代码**。完整分步手册（必填/可空字段、品牌文件清单、首次抓数据、部署前置、红线）见 **[`docs/FORK.md`](./docs/FORK.md)**，把下面两行发给 AI 即可：

```text
按 docs/FORK.md 帮我把这个站改成应援 <小马日文名> 的站。
马页：https://db.netkeiba.com/horse/<马页 URL 里那串数字>/
```

最小要点：`config/sire.json` 的 `sire` 段（`name_ja`/`name_cn`/`jbis_id`/`netkeiba_id`/`studbook`/**`progeny_years`**/`wins`）与 `site` 段（`id` 必须唯一——存储键前缀；`repo` 指向自己的仓库，空 = 编辑台关闭；`domains`/`analytics` 空 = 统计关闭；`cross_site` 缺 = 跨站板块不出现）。

## 数据更新

```bash
python run_update.py <策略>
```

| 策略 | 做什么 |
|---|---|
| `--init` | 删空 data/ 从 0 全量重建 |
| `--basic [--year …]` | 新马对账建档 + 补缺 + 合并（日常基本数据） |
| `--races` | 详情更新+判变 → 成绩增量 → 台账海外 → 合并（赛后日常） |
| `--horse 1,2,3` | 只处理指定 id 的马 |
| `--races-force` | 全部马重抓成绩页（覆盖式重建） |
| `--day 20261020[,..]` | 按天快速更新（SP 结果页；可配 `--jra-only` 跳过地方） |
| `--check [--fix]` | 数据一致性校验（+ 自动补跑修复）；含「数据体检」（乱码/覆盖率骤降/字段整批为空） |
| `--ledger` | 只拉台账海外并入（本站 `ledger.url` 空 = 未启用） |
| `--video` | 视频回放：抓 B 站 UP 主清单 → `data/video-map.json`（本站 `mid` 空 = 关闭；只在本地跑） |
| `--ci [--year …]` | 基本+比赛+校验+派生+断言+git 提交（GitHub Actions 每日） |

**真跑之前先冒烟**：`python scripts/smoke_fetch.py`（4~5 个请求、30 秒、只出报告不动数据）——判断上游（JBIS / netkeiba / studbook）今天还认不认这套脚本。抓取卡死判据 = **连续 20 分钟无输出**即整轮中止，没有总时长上限。

详细验证点 / 回归清单 / 排查指南见上节「维护与排障」；管线内部流程见 `docs/DESIGN.md` §5。

## 维护与排障

**日常三条线**（本仓启用 4 条 workflow，串联不抢文件）：`update-data` 每日 UTC 22:00 跑 `--ci`（有变化才 commit，末步 dispatch 部署）→ 串联 `status`（登録/抹消马簿）；`update-day` 每日北京 21:00 用 `--day` 快更当天赛果（幂等，与全量互不覆盖）。视频回放未启用，**没有** `video-task`。

**管线三条硬规矩**（动脚本前先看）：

- `scripts/basic` 与 `scripts/races` **业务隔离**、共享逻辑下沉 `scripts/core` 中立层（中立层不出现管线概念）；`status` / `video` 是独立支线，不挂在 `--ci` 里。
- **能离线就不联网**：`--print-urls`、`merge_*`、六个 verify 全部离线可跑；抓取一律经 `core/net.py`（按域限速 `DOMAIN_SLEEP`，别调低 —— 调低等于加速被封）。
- **人工值只写人工表**：`data/basic.json` 是合并产物，手改会被下一轮 merge 覆盖。正确姿势 = 改 `data/manual_overrides.json`（马字段）/ `races_manual.json`（配图）/ `timeline_manual.json`（航迹线节点）→ 重跑对应 `merge_*`。

**发版前 10 分钟**：`git status`（不该有 `dist/`）→ `cd front && npm run build`（「本站身份」段除 `domains` 空这一条预期 ⚠ 外无其他 ⚠）→ 六件套冒烟 → 肉眼过首页 / 基本信息 / 比赛记录 / 统计 / 日期统计，并把窗口拉窄到 768px 以下看抽屉。

**疑难速查**：

| 症状 | 先查 |
|---|---|
| 改了样式没生效 | 没 build；或动态拼接的 Tailwind 类被裁剪（去 `dist/assets/theme-*.css` 搜类名） |
| 线上侧栏高亮全灭 | CF 把 `/pages/x.html` 307 剥成 `/pages/x`，`location.pathname` 比对必须先经外壳 `pageKey()` 归一 |
| 编辑台打不开 / 403 | `site.repo` 与 PAT 所属仓库不一致（双站防串是故意的） |
| 访客数不涨 | 本仓 `site.domains` 为空 = 统计整体关闭（预期），要开就先填域名并在 GoatCounter 注册 |
| 某匹马数据一直缺 | `python run_update.py --horse <id>` 定向跑；再看 `data/fetch_log.jsonl` 里该 host 的状态码 |
| 全量跑到一半被掐断 | 日志出现 `✗ [步骤] 判为卡死（连续 1200s 无输出）` → 站点假死/请求挂住（看 `fetch_log.jsonl` 该 host 的状态码与 dur）；整轮退出是**有意的**：不完整缓存不许合并提交 |
| 抓完发现马名乱码/整批空 | 先跑 `python scripts/smoke_fetch.py`（4~5 个请求，30 秒）判断是解码/跳转/结构哪一处变了；再看 `--check` 报告「数据体检」段 |
| 删了旧图后老访客 404 | 缓存桶要 +1（`front/public/yj-cache.js` 的 `data-vN`） |

> ⚠ 已知上游状况：`race.netkeiba.com` 对部分出口 IP 整域回 **400 + 空响应**（本机实测命中，`db.netkeiba.com` 正常）。SP 富化会因此失败，新代码只记账并汇总「SP 富化失败 N 场」，不影响记录入库。

## 部署

- **GitHub Actions**：push main 时 `.github/workflows/deploy.yml` 自动 `npm run build`，把仓库根（含 `dist/` + `data/`）发布为 Pages 站点根，访问 `https://<user>.github.io/<repo>/dist/index.html`。
- **数据自动更新**：`.github/workflows/update-data.yml` 每天 UTC 22:00 跑 `--ci`（数据仓库模式，跑完有变化自动 commit+push），也可手动触发选策略；`update-day.yml` 每天北京时间 21:00 按天快速补当天赛果；`status.yml` 在 update-data 成功后串联跑登録状態任务（抹消/去向）。
- 前置：Settings → Pages → Source 选 **GitHub Actions**；Settings → Actions → Workflow permissions 勾选读写。

## 文档导航

| 文档 | 内容 |
|---|---|
| `docs/DESIGN.md` | **架构与约定的单一出处**：状态 / 前端架构 / 编码约定 / 管线 / 契约要点 / 编辑台 / Git 协作 / 遗留 / 决策 / UI §索引 / **母版同步规则** |
| `docs/FORK.md` | 克隆后改成自己的应援站（给 AI 执行的分步手册） |
| `docs/SCHEMA.md` | 数据产物字段契约（单一出处） |
| `docs/UI优化记录.md` | 各模块 UI 优化的最终方案与当前成果（含文首全站编码约定） |
| `docs/HANDOFF.md` | 交接速查（指针 + 当前状态；细则以 DESIGN 为准） |
| `docs/request-path.html` | 请求·数据流路径图 |
| `docs/README.md` | docs/ 目录索引 |
