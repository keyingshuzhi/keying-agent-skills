# GitHub CLI (`gh`) 实战手册

## 1. 安装与认证

### 平台安装

```bash
# macOS（Homebrew）
brew install gh
gh --version
gh auth login

# Windows（WinGet）
winget install --id GitHub.cli
# 安装后重新打开终端
gh --version
gh auth login

# Linux：使用对应发行版的官方安装说明
# https://cli.github.com/manual/
```

### 环境变量 token 检测

当 agent 运行在 CI 或受限环境时，token 可能已通过环境变量注入：

```bash
# 检测是否有可用的 token
echo "GITHUB_TOKEN=${GITHUB_TOKEN:-<未设置>}"
echo "GH_TOKEN=${GH_TOKEN:-<未设置>}"

# 验证 gh 当前实际使用的认证来源
gh auth status --show-token 2>/dev/null || gh auth status
```

- 存在 `GITHUB_TOKEN` / `GH_TOKEN` → 可跳过 `gh auth login`，直接进入操作。
- **注意**：此时仍需确认 token 的权限范围（scopes）是否满足目标操作（如发布 Release 需要 `repo`/`write:package` 等）。
- 环境 token 不在日志中展示；如需确认，请使用 `gh auth status` 而非直接 `echo` 完整值。

---

## 2. 工作区探测（gh 找不到时按此排查）

agent 运行在独立工作区（sandbox）内，`gh` 可能安装在宿主机但不在工作区 PATH 中。**必须先探测，不可直接假设其不存在。**

```bash
# 1. 检查命令是否在 PATH 中
command -v gh || which gh

# 2. 检查版本
gh --version

# 3. 搜索常见安装位置
ls -la /usr/local/bin/gh /opt/homebrew/bin/gh \
     /opt/brew/bin/gh /usr/bin/gh /usr/local/sbin/gh \
     ~/.local/bin/gh ~/.cargo/bin/gh 2>/dev/null

# 4. 检查当前 PATH 包含哪些目录
echo "$PATH"
```

排查结论与处理方式：

| 现象 | 结论 | 处理 |
|---|---|---|
| `command -v gh` 无输出，常见目录也无文件 | 工作区确实未安装 | 向用户报告并给出安装指引，等待确认后继续 |
| 常见目录存在 gh 文件 | 已安装但未在 PATH | 告知用户路径，询问是否需将其加入 PATH 或重新打开终端 |
| `gh --version` 输出 | 可用 | 继续后续检查 |

**本手册不会自动安装 `gh`**，安装操作由用户确认后执行。

---

## 3. 仓库上下文自动解析

确定本次操作针对哪个仓库，**优先级从高到低**：

```
显式 -R OWNER/REPO            → 命中（最明确）
    ↓ 未提供
当前目录是 git 仓库           → git remote -v → 取 origin
    ↓ 无 origin / 非 git 仓库
当前登录账户 + 用户已知仓库名  → 请用户显式确认
    ↓ 仍不明确
停止，要求用户显式给出 OWNER/REPO
```

示例：

```bash
# 显式指定（推荐所有跨仓库操作都如此）
gh issue list -R OWNER/REPO --state open --limit 30

# 若在当前仓库内，可通过 git remote 自动识别
git remote -v
# 输出示例：
# origin  https://github.com/OWNER/REPO.git (fetch)
# origin  https://github.com/OWNER/REPO.git (push)
```

---

## 4. 非交互模式强制

gh 在某些场景会进入交互式 prompt（如 `gh repo create` 未传 `--confirm` 时），在 agent 环境中会挂起等待输入。因此所有命令默认使用非交互参数：

```bash
gh repo create REPO --private --description "..." --confirm
gh release create v0.1.0 artifact.zip --draft --verify-tag
gh pr create --base main --head feature/x --title "..." --body "..."
```

常用非交互标志速查：

| 命令 | 非交互标志 | 作用 |
|---|---|---|
| `gh repo create` | `--confirm` | 跳过创建向导确认 |
| `gh release create` | `--draft`、`--verify-tag` | 创建草稿并验证标签 |
| `gh issue create` | `--assignee`、`--label` | 直接指定，跳过交互式输入 |
| `gh api` | `--silent` | 不输出进度信息 |

如确需交互式体验，请用户主动切换上下文并等待人工输入。

---

## 5. 速率限制与分页处理

### 速率限制

GitHub API 有严格的速率限制，`gh` 返回常见状态码：

| 状态码 | 含义 | 处理方式 |
|---|---|---|
| `403`（含 "rate limit"） | 超出速率限制 | 等待后重试；查看 `gh api rate_limit` 确认剩余配额；降低并发或改用分页分批 |
| `404` | 资源不存在 | 核对 OWNER/REPO、编号、标签、资产名 |
| `401` / `403`（权限类） | 认证失效或无权限 | 检查 `gh auth status` 与 token scopes |

检查当前速率配额：

```bash
gh api rate_limit --jq '.resources.main'
```

### 分页与截断

- 部分 `gh` 子命令默认只返回一页；大数据量查询时强制分页：

```bash
# 分页拉取所有 Issue 编号
gh api --paginate repos/OWNER/REPO/issues --jq '.[].number'

# 或使用 --limit 显式指定上限
gh repo list --limit 50
```

---

## 6. 输出解析技巧（--json / --jq）

agent 需要解析 `gh` 输出以做后续决策，应优先使用结构化输出：

```bash
# --json 指定需要的字段
gh repo view OWNER/REPO --json nameWithOwner,description,visibility,url,defaultBranchRefName

# --jq 做进一步过滤（支持 jq 表达式）
gh api repos/OWNER/REPO --jq '{full_name: .full_name, private: .private, default_branch: .default_branch}'
gh api --paginate repos/OWNER/REPO/issues --jq '.[].number'

# 只取单个值
gh repo view OWNER/REPO --json url --jq '.url'
```

常用字段速查（以 `--json` 形式使用）：

| 资源 | 常用字段 |
|---|---|
| Repository | `nameWithOwner`, `description`, `visibility`, `url`, `defaultBranchRefName`, `createdAt` |
| Issue | `number`, `title`, `state`, `body`, `url`, `createdAt`, `author`, `labels` |
| Pull Request | `number`, `title`, `state`, `url`, `baseRefName`, `headRefName`, `mergeStateStatus`, `reviews` |
| Release | `tagName`, `name`, `url`, `isDraft`, `isPrerelease`, `publishedAt` |
| Workflow Run | `status`, `conclusion`, `createdAt`, `url`, `headBranch`, `headSha` |

**输出切换**：`gh` 默认向终端输出彩色文本，解析困难。可在环境变量中强制 JSON 或禁用颜色：

```bash
export GH_FORCE_TTY=
gh repo view OWNER/REPO --json ...
# 或
gh --template '{{.url}}' ...
```

---

## 7. `git` 与 `gh` 职责边界（决策树）

```text
要操作本地仓库（文件、分支、提交历史）？
    └─ 是 → 使用 git
    └─ 否，要操作 GitHub 平台（仓库/Issue/PR/Release/Actions）？
        ├─ 是 → 使用 gh
        └─ 仍不清楚？

创建 PR 的完整链路：
    1. git: 分支 → 修改 → 提交 → git push
    2. gh: 创建 PR  → gh pr create --base main --head <分支>

推送本地分支到远程？
    └─ 是 → git push

在 GitHub 上创建仓库并克隆？
    1. gh: gh repo create OWNER/REPO --private --confirm
    2. git: git clone <url>
```

---

## 8. 操作影响预估模板

执行高风险写操作前，必须先向用户说明影响范围。示例模板：

```
拟执行：gh release create v0.1.0 ./dist/app.zip -R OWNER/REPO --draft

该操作的影响：
  + 在 OWNER/REPO 上创建 Release v0.1.0（草稿状态）
  + 上传 app.zip 作为 Release 附件
  - 不会推送到任何分发渠道，不会通知用户
  - 后续可通过 gh release delete v0.1.0 回滚

是否确认执行？
```

常见高危操作清单：

| 操作 | 影响 | 回滚方式 |
|---|---|---|
| `gh repo create` | 创建远程仓库 | `gh repo delete` |
| `gh release create` | 创建 Release 并上传资产 | `gh release delete` + `gh release delete-asset` |
| `gh pr create` | 创建 PR，可能通知相关人员 | `gh pr close` |
| `gh pr merge` | 合并到目标分支，改变仓库历史 | 需人工 revert 或 force push |
| `gh release delete` | 永久删除 Release 及所有附件 | 不可逆（除非有备份） |
| `gh repo edit --visibility public` | 公开私有仓库 | 可改回 private |
| `gh workflow run` | 触发工作流（可能产生费用/部署） | 无法撤销已运行步骤 |

---

## 9. 场景化工作流模板

### 9.1 从 Issue 到修复 PR 的闭环

```bash
# 1. 确认 Issue
gh issue view 123 -R OWNER/REPO --json number,title,state,body,url

# 2. 创建分支并开发（git）
git checkout -b fix/issue-123
# ... 编写代码 ...
git add -A && git commit -m "fix: 修复 #123"

# 3. 推送到远程
git push -u origin fix/issue-123

# 4. 创建 PR，并关联 Issue
gh pr create --base main --head fix/issue-123 \
  --title "fix: 修复 #123" \
  --body "Closes #123"
```

### 9.2 发布前的检查清单

```bash
# 1. 确认目标分支清洁
git status && git log HEAD..origin/main

# 2. 确认版本号与 changelog 已更新

# 3. 确认 CI 全部通过
gh run list -R OWNER/REPO --limit 5 --json status,conclusion,createdAt,url

# 4. 生成并校验制品 checksum
shasum -a 256 ./dist/app.zip > ./dist/checksums.txt

# 5. 创建草稿 Release（先不发布）
gh release create v0.1.0 ./dist/app.zip ./dist/checksums.txt \
  -R OWNER/REPO --title "v0.1.0" --notes "Release notes" --draft --verify-tag

# 6. 人工确认后发布
gh release edit v0.1.0 -R OWNER/REPO --draft=false --verify-tag
```

### 9.3 批量获取打开的 PR 及其状态

```bash
gh api --paginate repos/OWNER/REPO/pulls \
  --jq '.[] | select(.state=="open") | {number, title, state, url}'
```

---

## 10. 版本兼容性说明

`gh` 的命令与 flag 可能随版本变化，编写脚本前请先确认：

```bash
gh --version         # 查看当前版本
gh <command> --help  # 查看该命令可用 flag
gh <command> --help | grep -- -
```

已知的常见版本差异：

| 命令 | 旧写法 | 新写法（推荐） | 影响 |
|---|---|---|---|
| `gh repo create` 可见性 | `--private` / `--public` | `--visibility private` / `public` | `--private` 仍保留但逐步废弃 |
| `gh release create` | 标签可省略（基于当前 tag） | 建议显式传 `TAG` + `--verify-tag` | 显式更安全 |
| `gh issue create` 关联 | `--issue` 等 | `--body` 中写 `Closes #123` | 两种方式均可，建议明确指定 |

---

## 11. 故障排查表

| 症状 | 检查 | 处理 |
|---|---|---|
| `gh: command not found` | `command -v gh`；常见安装目录；`echo "$PATH"` | 按「工作区探测」章节排查，引导安装 |
| `Not logged in` | `gh auth status` | 运行 `gh auth login` 完成浏览器认证 |
| `404 Not Found` | OWNER/REPO、Issue/PR 编号、Release 标签、资产名 | 逐一核对，确认仓库名大小写与权限 |
| `401 Unauthorized` | `gh auth status` | token 过期或失效，重新登录或换 token |
| `403`（rate limit） | `gh api rate_limit` | 等待后重试；改用分页分批；降低并发 |
| 权限不足（403） | token scopes | 确认 token 拥有 `repo`/`read:org` 等对应权限 |
| 错误仓库 | 当前目录 / git remote | 显式 `-R OWNER/REPO` 指定目标 |
| Release 资产上传失败 | 标签是否存在、文件路径、文件名、写权限 | `gh release view TAG` 核对；确认文件存在 |
| PR 创建失败 | 分支是否存在、base 分支名 | 先 `git push` 并核对 `--base` |
| 命令挂起无响应 | 是否缺少非交互参数（`--confirm` 等） | 补上非交互标志或中断后重试 |

---

## 12. 安全与隐私

- 使用最小权限凭证：token 仅申请目标操作所需的 scopes。
- 不将 token 持久化到仓库文件或 shell 历史；优先使用环境变量注入。
- 永不记录或分享原始 token、secret 值。
- CI 日志含凭证时先脱敏再分享；`--log-failed` 输出需人工审查。
- 私有仓库数据默认不向用户披露，除非用户显式授权。
- 执行破坏性命令前必须获得明确的「目标 + 确认」。

官方文档：https://cli.github.com/manual/
