# 絆 · 交接速查（HANDOFF）

> **本文件已收敛**：架构 / 编码约定 / 数据管线 / 数据契约 / 编辑台 / Git 协作 / 遗留 / 决策时间线的**唯一出处 = [`docs/DESIGN.md`](./DESIGN.md)**（2026-11-13 与母版仓 `yunji-web` 对齐时同步收口）。
> 本文件只保留「新接手第一天要知道的几件事」；与 DESIGN 冲突时一律以 DESIGN 为准。

## 0. 一句话

**絆 · 绊崽档案**：キズナ（高情厚意 / Kizuna，netkeiba `2010105827`）产驹的个人查阅/检索静态站，GitHub Pages 部署，入口 `/dist/index.html`，`dist/` 自包含 `data/` 副本。

## 1. 当前状态（2026-11-13 对齐后）

| 维度 | 状态 |
|---|---|
| 数据 | 建档 **1221 匹**（生年 2017~2025）· 血统 1221 · 成绩 1213 个 `races/{id}.json` · `nk_id` 空 8 |
| 页面 | 9 个功能页（profile / races / stats / leaderboard / datechart / **entries 出走马表** / timeline / pedigree）+ 编辑台 4 页 |
| 配置 | `config/` 五个 JSON 即站点身份全部旋钮：`sire.json`（种马 + `site` 段）· `features.json`（能力表）· `friend-links.json` · `video-sources.json`（`mid` 空 = 回放关闭）· `grade-table.json` |
| 数据更新 | `python run_update.py <策略>`（10 种，见 README 表）；Actions：`deploy` / `update-data`（每日 `--ci`）/ `update-day` / `status` |
| 本地预览 | `python serve.py` → `http://127.0.0.1:9091/index.html`（端口单源 `serve.config.json`） |
| 门禁 | `python scripts/check_data.py`（含数据体检）+ 6 个 `verify_*.cjs` 全绿 |

## 2. 三条硬性协作约定（原文见 DESIGN §8 / AGENTS.md §0）

- **commit 只在用户明确要求时做**；**永远不 `git push`**。
- **代码随母版、身份不随母版**：`front/` `scripts/` 等代码文件不要手改（下次从 `yunji-web` 同步会冲突），品牌 / 数据 / `config/` 归本站。同步清单见 DESIGN §12。
- **改完必构建**：`cd front && npm run build`，核对 `dist/assets/theme-*.css` 里新类已编译、构建末尾「本站身份」自检无多余 ⚠。

## 3. 与旧版文档的关系

历史过程文档（`REFACTOR.md` 重构方案 / `TESTING.md` 测试排查 / `pipeline.md` 管线流程 / `重复代码审计.md` / `审计报告-工作区全量改动.md` / `友情跳转方案.md`）已于 **2026-11-13 删除**：其中仍然有效的内容都收进了 `docs/DESIGN.md`（§4 编码约定 / §5 管线 / §12 母版同步）与 `README.md`「维护与排障」。要看原文：`git log --diff-filter=D -- docs/` 找到删除提交，`git show <commit>^:<path>` 取回。

现行文档索引见 [`docs/README.md`](./README.md)。
