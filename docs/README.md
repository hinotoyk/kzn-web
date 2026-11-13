# docs/ · 文档索引

> 全部人工文档收在本目录；仓库根只留 `README.md`（项目介绍 / 配置 / 维护与排障）与 `AGENTS.md`（AI 助手硬规则）。
> 自动生成报告 `data/check_report.md` / `data/races_report.md` / `data/studbook_report.md` 是脚本产物，留在 `data/`，不属人工文档。

| 文档 | 说明 |
|---|---|
| `docs/DESIGN.md` | **架构与约定的唯一出处**：当前状态 / 前端架构 / 编码约定（含冒烟断言红线）/ 数据管线 / 数据契约要点 / 编辑台 / Git 协作 / 遗留 / 决策时间线 / UI 规格 §索引 / **§12 与母版仓同步** |
| `docs/SCHEMA.md` | 数据产物字段契约与口径（**单一出处**；改字段先改这里） |
| `docs/FORK.md` | **克隆后改成自己的应援站**（给 AI 执行的分步手册：必填/可空字段、品牌文件、首次抓数据）。本仓自身也是按它出生的 |
| `docs/UI优化记录.md` | 各模块 UI 优化的最终方案与成果（文首 = 「★ 全站编码约定 · 复用抽离」；§N 编号与源码注释对应，DESIGN §11 是索引） |
| `docs/HANDOFF.md` | 交接速查：新接手第一天要知道的几件事（细则一律以 DESIGN 为准） |
| `docs/request-path.html` | 请求·数据流路径图（基础 + 竞赛 3 视图，浏览器打开） |

相关入口（仓库根）：[../README.md](../README.md)（项目介绍 / 配置 / 维护与排障）· [../AGENTS.md](../AGENTS.md)（AI 助手硬性规则汇总）。

## 已删除的历史过程文档（2026-11-13，内容并入 DESIGN / README）

| 已删文档 | 原内容 | 现存替身 |
|---|---|---|
| `REFACTOR.md` | 种马配置化 / 复用收口 / docs 目录整理的重构方案（P1–P5 已落地） | 配置契约 → `config/*.json` 键注释 + DESIGN §3；复用约定 → DESIGN §4 |
| `TESTING.md` | 更新策略的测试验证点 / 回归清单 / 排查手册 | README「离线验收」「维护与排障」 |
| `pipeline.md` | 管线流程 / 设计原则 / 离线红线 | DESIGN §5 |
| `重复代码审计.md` | 复用收口清单（P1/F1 已收口、P2/F2 故意保留） | DESIGN §4 |
| `审计报告-工作区全量改动.md` | 上述重构的落地审计（逐项对照表） | 结论已进 `AGENTS.md` / DESIGN |
| `友情跳转方案.md` | 四站互跳组件方案 | `config/friend-links.json` → `window.YJ_FRIENDS`，渲染 `front/public/friend-links.js` |

> 源码注释里若出现这些文件名（如 `docs/重复代码审计.md` A1/B1），属**历史出处指认**，不必再找文件——原文从 git 历史取：`git log --diff-filter=D -- docs/`。
> 以后新增约定请写进 `DESIGN.md` 或 `AGENTS.md`，不要再新起过程文档。
