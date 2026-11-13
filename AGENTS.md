# AGENTS.md · 絆项目规则（AI 助手必读）

本文件汇总散落在项目各文档中的**硬性规则**，供 AI 助手在任何会话开始时遵守。详细背景与依据见源文档（已注明出处）。

## 0. Git 协作（最高优先级）

- **commit**：仅在用户明确要求提交时执行；未要求不得自行 `git commit`。
  —— `docs/DESIGN.md` §8「Git 协作约定」
- **push**：永远不执行 `git push`；推送只能由用户本人操作。
  —— `docs/DESIGN.md` §8「Git 协作约定」
- `dist/` 在 `.gitignore` 中，构建产物不入库，由 CI/本地构建生成。

## 0.1 与母版仓同步（本站 = 分支站，务必先读）

代码随母版、身份不随母版。母版仓 = `yunji-web`（云迹，同机 `Z:/IdeaProjects/yunji-web`）。

- **不要手改**：`front/**`、`scripts/**`、`run_update.py`、`run_full_test.py`、`serve.py`、`.github/workflows/*` —— 这些归母版，下一次同步会整文件覆盖；发现 bug 先考虑是否为上游问题（`python scripts/smoke_fetch.py` 判定），要改就同步反哺母版。
- **只在本地改**：`config/sire.json`（种马 + `site` 段）、`config/features.json`、`config/friend-links.json`、`config/video-sources.json`、`data/**`、`README.md`/`AGENTS.md` 品牌段、本仓过程文档。
- 同步清单与最近一次对齐（2026-11-13）见 `docs/DESIGN.md` §12。

## 1. 工作流程

- 正式源码在 `front/`（Vite 多页 + Tailwind）；修改 → `npm run build` → 产物到项目根 `dist/` → 验证 `dist/` 生效。
- **每次改样式/类名后必须构建并核对** `dist/assets/theme-*.css` 含新类——编译期框架最大坑：改了类但没编译 → 页面无变化。
- 探索/候选版放项目根 `tests/`，废案放 `tests/_trash/`；旧 `testpage/` 已归档 `tests/_trash/testpage-snapshot`，只读勿直接改。
- UI 优化：每改一个模块，先把「最终思路 + 当前成果」写进 `docs/UI优化记录.md`（不展开废案），再合并进正式页面。
- **架构与约定的单一出处 = `docs/DESIGN.md`**；`docs/HANDOFF.md` 只是速查指针。历史过程文档（REFACTOR / TESTING / pipeline / 审计类 / 友情跳转方案）已于 2026-11-13 删除，需要时从 git 历史取。新增约定写进 DESIGN 或本文件，不要再新起过程文档。

## 2. 前端架构（必须遵守，不得退回旧方式）

- 架构已定：**Vite 多页构建 + Tailwind CSS v3（编译期）**。禁止手写散落 CSS / CDN 运行时 Tailwind / 无构建直接改页面。
- 样式一律用 Tailwind utility 类；复用组件类在 `front/pages/theme.css` 的 `@layer components` 用 `@apply` 封装；不散落原生 CSS、不写死 hex（指定色块除外）。
- 主题 token 统一在 `front/tailwind.config.js`（primary 青绿 `#0aa7a0`、shadcn Neutral 灰阶、Noto Sans SC + Geist）。
- JS 里拼接的 Tailwind 动态类必须能被 `tailwind.config.js` 的 `content` 扫到（当前 globs 含 `./pages/**/*.js` 与 `./public/**/*.js`）。
- 共享 JS（非 module `<script>`）放 `front/public/`，由 Vite 原样拷到 `dist/` 根，页面用 `../xxx.js` 相对引用。
- 响应式：统一 `max-md:`（768px）单断点，不要写多个碎断点。
- i18n：日文字段/枚举 → 中文全部走 `front/public/i18n.js` 字典（`YJ.i18n.t/e/g`），**不改后端 JSON**。
- 数据语义色（全站一致）：1着/胜=`primary` 青绿、2着=`chart-2` 青、3着=`chart-3` 橙、未完走（中止/取消/除外/失格）=`destructive` 红、重赏徽章（GI/GII/GIII/L/OP）用 `primary`/`chart-*` 分级。
- 跨页联动走 `bus.js`（postMessage + **sessionStorage**，按标签页隔离），不另起机制。
- **站点身份全部来自 `config/`（2026-11-12 开源化定稿）**：站名/副标题/徽标/编辑仓库/统计/友链/跨站地址/产驹生年都读 `config/*.json`，**页面里禁止写死**——构建期注入 `dist/site-config.js`（`window.YJ_SITE` / `YJ_CAP` / `YJ_FRIENDS`），HTML 用 `{{YJ_SITE_NAME}}` 等占位符。空值 = 合法关闭态（`repo` 空 → 编辑台整关、`analytics` 空 → 不注入统计、`cross_site` 缺 → stats 隐藏种马榜走势 / entries 显示未配置）。构建末尾会打「本站身份」自检，除「`site.domains` 空」这条预期 ⚠ 外，出现任何 ⚠ 都要回 `config/` 补。
- **页面能力表**：能力 id → 页面映射的唯一出处 = `front/scripts/features.cjs` 的 `CAPS`；开关在 `config/features.json`（只写要关的）。被关闭的能力**仍然构建**（直连 URL 显示「本站未启用」空态），构建入口由 `front/pages/*.html` 目录派生、`vite.config.js` 不再硬编码页面清单。
- **可见性门禁**：只读页引 `front/public/yj-cap.js`（未启用空态）、编辑页引 `front/public/yj-edit-gate.js`（PAT 门禁，且必须在 `yj-util.js` 之后、`editor*.js` 之前）；两者都是可见性门禁，**不是安全边界**。
- **站点级存储键必须带站点前缀（§105 定稿）**：localStorage / sessionStorage / IndexedDB 库名 / Cache API 桶名一律经 **`YJ.util.storeKey(k)`** 构造，前缀 = `YJ_SITE.id`（源 = `config/sire.json` site.id，缺省回退共享 `"yj"`）——禁止裸字面量键；且**加载期不得求值**（部分页面 `site-config.js` 晚于模块脚本，加载期算好会静默退回旧共享键 → 同源双站 PAT/草稿/缓存互串，不报错最难查）。新增键先查 `docs/UI优化记录.md` §105 的键清单。
- **复用抽离（硬性约定，docs/UI优化记录.md §48 定稿）**：公共工具（esc 等）单一出处 `front/public/yj-util.js`（`YJ.util.*`，页面薄别名，禁止复制函数体）；比赛行/徽章/格式化/格级映射一律复用 `front/public/race-rows.js`；着顺浅色三件套 JS 侧 = `race-rows.js` 的 `PLACE_BG`（CSS 侧 = theme.css `.yj-nkm*`，两处互指、改色同步）。**只有全站统一语义或 ≥3 处使用才抽离，仅 2 处不动**；收口只做等值搬移/纯删除，改后照常 build+dist 核对。theme.css `@layer components` 里**不放 JS 动态拼类名、content 无字面量的类**（会被 Tailwind 按候选裁剪，如 `.yj-g*` 必须在 @layer 之外）。细则见 `docs/UI优化记录.md` 文首「★ 全站编码约定 · 复用抽离」。

## 3. 数据与路径

- 数据源在项目根 `data/`，构建时由 `vite.config.js` 的 `copy-data` 插件复制为 `dist/data/`（跳过 `_tmp/` 与 `*.md`/`*.csv`/`*.jsonl`）；前端一律经 **`YJ_DATA.url('...')`** 解析（`front/public/data-config.js` 为**唯一数据源映射点**，`prefix:'..'`，内部拼 `../data/...`）。**改数据目录/位置只改该文件 `prefix`/`root`**，禁止散落写死数据路径。`data/` 各产物的**字段契约与口径单一出处 = `docs/SCHEMA.md`**（生成脚本文件头只留一行指路），管线说明见 `docs/DESIGN.md` §5。
- **本地预览一律 `python serve.py`**（项目根；服务 `dist/`，端口单源 = `serve.config.json` 的 **9091**，启动即打开 `http://127.0.0.1:9091/index.html`，`--no-open` 只起服务）。禁止再手写 `python -m http.server <port>`（端口散落 + 缺 no-cache/CORS）。`dist/` 已自包含数据副本，务必 http 访问（file:// 下 fetch 失败）；`scripts/` 更新数据后需重新构建才会进 `dist/data/`。
- **端口是约定不是偏好**：四兄弟站 yunji `9090` / kzn `9091` / jbis-sire-rank-history `9092` / racing-calendar `9093`，各仓 `serve.config.json` 为唯一出处（本仓 = 9091，与 yunji 同源部署也不再串 localStorage）。端口被占用时脚本**直接报错退出**——不要改成自动顺延；换端口只改该 JSON，并连带核对跨站引用。
- `data/basic.json` 是后端合并产物，**前端只读，勿手改**。
- **前端数据一律走统一缓存控制器 `front/public/yj-cache.js`（`YJ.cache.fetch(name[, opts])`）**：注册表按产物给 TTL（当前全 12h，可按产品覆写），禁止页面绕过控制器直连 fetch 数据产物（编辑台 no-store 语义除外）。**改数据目录/位置时同步核对 `yj-cache.js` 注册表的 `url/prefix`**。
- `data/races-bundle.json`（比赛压缩包，契约 = `docs/SCHEMA.md` §8）是**构建期派生产物**：只在 `npm run build` 时由 `front/scripts/build-races-bundle.mjs` 生成进 dist（**替换式**——打包成功则 dist 不拷贝 `data/races/*.json`），**`data/` 仓库不落盘、后端管线不经过它**；数据更新后重新构建即得新 bundle。
- `data/fetch_log.jsonl`（抓取观测日志，逐行 JSON）**入库可追踪**：`.gitattributes` 的 `data/fetch_log.jsonl merge=union` 扛并发 rebase（status/update-data/update-day 三条线都会追加同一文件），`scripts/core/net.py::trim_fetch_log` 按 `LOG_RETENTION_DAYS`（60 天）滚动裁剪，检测频率 `TRIM_CHECK_EVERY`（每 N 次写入一次，避免逐请求全量读 4MB）。构建期 `vite copy-data` 跳过 `*.jsonl`，日志不进 `dist/` 与线上包。
- 数据更新：`scripts/basic` 与 `scripts/races` **业务隔离、代码抽离复用**——两管线共享逻辑统一下沉中立层 `scripts/core`（路径/站点/颜色等字面量常量单源 = `core/constants.py`，两个 `common.py` 只做薄再导出，不再各持一份拷贝）；中立层不得出现管线概念，各管线风控口径 `DOMAIN_SLEEP`/`STRIP_BASES` 仍由各 `common.py` 持有并以形参注入 `net.bind()`；业务模块互不 import 对方。数据统一落根 `data/`；日常/定时统一入口 `python run_update.py <策略>`（10 种，表见 `README.md`）。⚠ `--init`/`--races-force` 会删空/覆盖 `data/`（含 `data/photos/` 与人工表，无回收站），`--ci` 会 commit+push（测试必须隔离：别在真库上跑 `--ci`，用临时副本或 `--limit`）。
- **产驹生年只从 `config/sire.json` 的 `sire.progeny_years` 读**（缺/空/非法 → 管线直接退出）。改收录范围只改这个数组，不要在脚本里写死年份。
- **卡死判据 = 有没有输出**：`run_update.py` / `run_full_test.py` 已删掉单步总时长超时，改为「连续 20 分钟无任何输出即整轮中止」；总时长兜底只由 CI 的 `timeout-minutes`（360）承担。不要为了「跑得完」去下调 `DOMAIN_SLEEP`（那是防封 IP 的，不是性能旋钮）。
- **改完/更完数据必跑门禁**：`python scripts/check_data.py`（含「数据体检」：乱码 / 覆盖率骤降 / 字段整批为空 / 着顺未归一）+ 6 个断言 `node scripts/{races/verify_result,races/verify_drill,stats/verify_stats,datechart/verify_datechart,timeline/verify_sire_wins,verify_caps}.cjs`，全绿才算完。断言里**不许写死本库具体值**（马名/条数/年份），一律从产物派生。
- **抓取异常先做冒烟**：`python scripts/smoke_fetch.py`（4~5 个请求、30 秒、只出报告不动数据、不改退出码）——FAIL 说明上游契约漂移（编码 / 跳转 / 页面结构），属母版问题，别在本仓重写管线。
- **派生必须同轮重算**：改了 `sire.wins` → 重跑 `scripts/timeline/sync_sire_wins.py`；改了格口径/数据 → 重跑 `build_timeline` / `build_datechart` / `build_stats`，否则 verify 报漂移。

## 4. 已删除 / 不要复活

- 早期配色方案 A~D、shadcn 探索页、顶部导航布局方案、暗黑模式 已删除，**不要复活**。
- 项目约定：「请求怎么发」可参考旧实现，「业务流程」要自己重新设计。

## 5. 项目定位

- 站点：**絆 · 高情厚意档案**（单一种马产驹的个人查阅/检索用，当前收录キズナ产驹 **1221 匹**，生年 2017~2025）。
- 母版仓：`yunji-web`（云迹）——本仓是它的分支站，代码随母版走、品牌与数据自理（见 §0.1）。
- 部署：GitHub Pages 静态站，**站点根 = 仓库根**，入口 `/dist/index.html`（dist 已自包含 `dist/data/` 副本；后续可改为只发布 `dist/`）；push main 时 `.github/workflows/deploy.yml` 自动 `npm run build` 并发布。本地预览用项目根 `python serve.py`（固定 `http://127.0.0.1:9091/`，服务 `dist/`）。
