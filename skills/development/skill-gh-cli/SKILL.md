---
name: github-cli
description: 使用官方 GitHub CLI（gh）安全操作 GitHub 仓库、Issue、Pull Request、Release 和 GitHub Actions；支持环境探测、仓库上下文自动解析、认证检查与凭据保护。
---

# GitHub CLI 技能

## 目的

使用官方的 `gh` CLI 操作 GitHub 平台；本地版本控制改用 `git`。对支持的操作优先调用 `gh` 命令，而非手写 REST 请求。

本技能的核心能力：

- 在动手前先确认 **`gh` 是否在当前环境可用**，找不到时引导安装。
- 自动从显式参数 / `git remote` / 登录账户解析**目标仓库上下文**。
- 强制**非交互模式**，避免在 agent 环境中被交互式 prompt 挂起。
- 识别 **GitHub 速率限制、分页、token 来源**等常见问题。
- 明确区分 `git` 与 `gh` 的职责边界，并在执行高风险操作前预估**影响范围**。

## 何时使用

- 用户要求创建、查看、克隆或配置 GitHub 仓库。
- 用户要求管理 Issue、Pull Request、Release 或 Actions 工作流。
- 用户要求检查 GitHub 认证状态、使用 `gh api` 查询，或通过 GitHub 平台完成发布/协作。

## 何时不使用

- 仅操作本地仓库（克隆、提交、分支、合并等）→ 应改用 `git`。
- 用户已明确提供完整的命令（不含 `gh`）→ 应改用 `git`。
- 需要手写 GitHub REST API 特殊参数，而 `gh` 暂不支持 → 可使用 `gh api` 或改用 API。

---

## 前置检查（必须首先执行）

### 第一步：检测 `gh` 是否可用

**注意**：agent 运行在独立工作区（sandbox）内，`gh` 可能安装在宿主机但不在工作区 PATH 中。因此必须先探测，不可直接假设其不存在或不可用。

```bash
# 1. 检查命令是否在 PATH 中
command -v gh || which gh

# 2. 检查版本
gh --version
```

**如果 `gh` 找不到**（无输出或 `command not found`），按以下顺序处理：

```bash
# 3. 搜索常见安装位置
ls -la /usr/local/bin/gh /opt/homebrew/bin/gh \
     /opt/brew/bin/gh /usr/bin/gh /usr/local/sbin/gh \
     ~/.local/bin/gh ~/.cargo/bin/gh 2>/dev/null
```

- 找到可执行文件 → 向用户报告路径，询问是否将目录加入工作区 PATH 或重新打开终端后再执行。
- 仍未找到 → **停止所有 `gh` 操作**，向用户报告：

> 当前工作环境中未找到 `gh` 命令。
>
> 请在本地安装 GitHub CLI：
> - macOS（Homebrew）：`brew install gh`
> - Windows（WinGet）：`winget install --id GitHub.cli`（安装后重新打开终端）
> - Linux：参考官方手册 https://cli.github.com/manual/
> - 已安装在非标准位置：将安装目录加入 PATH 后再试。
>
> 安装完成后请重新发出指令。

本技能**不会自动安装** `gh`，由用户确认后执行。

### 第二步：检查认证状态

`gh` 可用后，先确认是否已登录：

```bash
gh auth status
```

- 已登录 → 继续。
- 未登录 → 引导用户使用浏览器完成认证：

```bash
gh auth login
```

- 存在可用的环境变量 token（`GITHUB_TOKEN` / `GH_TOKEN`）→ 说明其可用，但需确认权限范围（scopes）是否满足目标操作。

### 第三步：解析目标仓库上下文

确定本次操作针对哪个仓库，**优先级从高到低**：

```
显式 -R OWNER/REPO            → 命中（最明确）
    ↓ 未提供
当前目录为 git 仓库           → 读取 git remote -v → origin（若有）
    ↓ 无 origin / 非 git 仓库
当前登录账户 + 已知仓库名     → 提示用户确认
    ↓ 仍不明确
停止，要求用户显式给出 OWNER/REPO
```

示例：

```bash
# 显式指定（推荐所有跨仓库操作都如此）
gh issue list -R OWNER/REPO --state open --limit 30

# 若在当前仓库内，可通过 git remote 自动识别
git remote -v
```

---

## 操作规范

1. **识别意图**：只读、创建、更新、发布或删除。
2. **解析范围**：确认目标仓库、分支、Issue/PR 编号、Workflow 或 Release 标签。
3. **变更前检查**：先用 `gh ... view/list` 拉取现有状态。
4. **计划命令**：使用非交互、显式 `-R OWNER/REPO` 和结构化 `--json` 输出。
5. **批准门禁**：执行发布、删除、合并、改可见性、触发工作流等影响外部资源的写操作前，必须先向用户说明**影响范围**并获得显式授权。
6. **执行与验证**：用 `gh ... view/list` 验证执行结果，报告标识符或 URL。
7. **永不展示凭据**：不在对话或日志中打印 token、secret 值或私有仓库内容。

---

## 命令模式

### 认证与状态

```bash
command -v gh
gh --version
gh auth status
gh config list   # 查看当前主机与 token 配置
```

### 仓库

```bash
# 查看仓库（显式范围）
gh repo view OWNER/REPO --json nameWithOwner,description,visibility,url

# 列出仓库（分页、避免截断）
gh repo list --limit 30 --jq '.[].name_with_owner'

# 克隆
gh repo clone OWNER/REPO

# 创建私有仓库（高风险，先预估影响）
gh repo create REPO --private --description "Project description" --confirm
```

### Issue

```bash
gh issue list -R OWNER/REPO --state open --limit 30
gh issue view 123 -R OWNER/REPO --json number,title,state,body,url
gh issue create -R OWNER/REPO --title "Bug: ..." --body "Reproduction steps..." --assignee @me
```

### Pull Request

```bash
gh pr list -R OWNER/REPO --state open --limit 30
gh pr view 42 -R OWNER/REPO --json number,title,state,baseRefName,headRefName,url
gh pr checks 42 -R OWNER/REPO
gh pr diff 42 -R OWNER/REPO
gh pr create --base main --head feature/example --title "Add feature" --body "Summary"
```

### Releases 与制品

```bash
gh release list -R OWNER/REPO --limit 30
gh release view v0.1.0 -R OWNER/REPO
gh release create v0.1.0 ./dist/app.zip -R OWNER/REPO --title "v0.1.0" --notes "Notes" --draft
gh release upload v0.1.0 ./dist/checksums.txt -R OWNER/REPO --clobber
```

### GitHub Actions

```bash
gh workflow list -R OWNER/REPO
gh run list -R OWNER/REPO --limit 20
gh run view RUN_ID -R OWNER/REPO --log-failed
gh workflow run build.yml -R OWNER/REPO --ref main
```

### API 与 JSON 自动化

```bash
# 结构化查询 + jq 过滤
gh api repos/OWNER/REPO --jq '{full_name: .full_name, private: .private, default_branch: .default_branch}'
gh repo view OWNER/REPO --json nameWithOwner,visibility,url --jq '.url'
gh api --paginate repos/OWNER/REPO/issues --jq '.[].number'
```

---

## 安全边界与防护（Guardrails）

- **绝不自行安装** `gh`，找不到时只引导用户。
- 不在对话或日志中打印 token、secret 或私有仓库内容；绝不执行 `gh auth token` 输出凭据。
- 绝不假定当前目录指向目标仓库；跨仓库操作必须显式 `-R OWNER/REPO`。
- 所有写命令默认使用非交互参数（`--confirm`、`--no-input`、`--draft` 等），避免交互式 prompt 挂起 agent。
- 视为高危操作：`gh repo delete`、`gh release delete`、`gh secret set`、`gh pr merge`、`gh workflow run`、`gh repo edit --visibility`、release 发布。执行前必须预估影响并获批准。
- 区分本地操作（`git add/commit`）与远程影响（`git push`、`gh` 写入）。
- 识别速率限制（403/429）与 API 截断，数据量大时强制分页（`--paginate` / `--limit`）。
- 不编造命令输出，未实际执行验证时不声称成功。
- CI 日志含凭证时先脱敏再分享。

---

## 参考资料

详细说明见 [references/guide.md](references/guide.md)，包含：

- `gh` 安装与认证指南
- 工作区探测与仓库上下文解析
- 非交互模式、速率限制与分页处理
- token 环境变量检测
- 输出解析技巧（`--json` / `--jq`）
- `git` 与 `gh` 职责决策树
- 操作影响预估模板
- 场景化工作流模板
- 版本兼容性说明与故障排查表

---

## 完成标准

- [ ] 已确认 `gh` 在当前环境可用（或明确引导用户安装）。
- [ ] 已确认认证状态（或明确引导用户登录）。
- [ ] 已解析目标仓库上下文，必要时已请用户显式确认。
- [ ] 所有命令已使用非交互参数。
- [ ] 高风险写操作前已预估影响并获用户授权。
- [ ] 执行后已用 `view/list` 验证结果并报告 URL/标识符。
- [ ] 未在任何输出中泄露凭据或私有内容。
