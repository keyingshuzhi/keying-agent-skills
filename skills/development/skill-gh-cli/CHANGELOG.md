# Changelog

所有 notable 的变更在此记录。

格式基于 [Keep a Changelog](https://keepachangelog.com/)。

---

## [Unreleased]

### 新增
- 新增 `gh` 工作区可用性探测：无法找到时按 PATH / 常见安装位置 / 环境变量 token 顺序排查，并引导用户安装，**不自行安装**。
- 新增仓库上下文自动解析：`-R OWNER/REPO` → `git remote -v`（origin）→ 登录账户 → 请用户显式确认。
- 新增认证状态检查与 token 环境变量检测（`GITHUB_TOKEN` / `GH_TOKEN`）流程。
- 强制非交互模式：写命令默认追加 `--confirm`、`--draft` 等非交互参数，避免 agent 环境被交互式 prompt 挂起。
- 新增速率限制与分页处理指引（`gh api rate_limit`、`--paginate`、`--limit`、403/429 处理）。
- 新增输出解析技巧：`--json` / `--jq` 字段速查与彩色转纯文本切换。
- 新增 `git` 与 `gh` 职责边界决策树。
- 新增高危操作影响预估模板与回滚指引。
- 新增场景化工作流模板（Issue→PR 闭环、发布前检查、PR 批量状态）。
- 新增版本兼容性说明与增强的故障排查表。
- 全文本地化为中文。

### 变更
- `gh-cli` 由 v0.1.0 升级至 v0.1.1。

---

## [0.1.0] — 2026-01-09

### 新增
- 首个版本：GitHub CLI 操作技能，覆盖仓库、Issue、Pull Request、Release、GitHub Actions。
- 建立审批门禁、凭据保护、显式范围解析（`-R OWNER/REPO`）等安全策略。
