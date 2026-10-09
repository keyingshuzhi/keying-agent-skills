# GitHub CLI Skill

面向 KoyingPilot（柯影智航）的 Agent 技能，用于安全、一致地调用官方 GitHub CLI（`gh`）完成仓库、Issue、Pull Request、Release 和 GitHub Actions 等操作。

## 目录结构

```text
gh-cli-skill/
├── references/
│   └── guide.md          # 命令手册：安装、操作、故障排查
├── ANALYSIS_REPORT.md    # 设计决策、风险矩阵与验收清单
├── extension.json        # 项目元数据（id/name/version/entry/tags）
├── LICENSE               # MIT 许可
├── README.md             # 本文件
└── SKILL.md              # 技能入口：激活条件与执行策略
```

## 安装与配置

1. 安装 GitHub CLI：https://cli.github.com/
2. 完成认证：`gh auth login`
3. 将本目录复制到 Agent 平台的 Skill 目录（具体路径取决于平台）
4. 确保平台加载 `SKILL.md`。`extension.json` 为项目本地元数据，可根据宿主应用按需适配。

## 使用示例

- "查看 OWNER/REPO 最近 10 次 GitHub Actions 运行。"
- "使用这些构建产物为 v0.1.0 创建草稿 Release。"
- "汇总当前打开的 PR 及其失败的检查项。"
- "为本项目创建一个私有仓库。"

## 安全策略

- **先读后写**：变更前先拉取现有状态，变更后验证结果。
- **审批门禁**：对外部资源有实质影响的写操作（发布、删除、合并、改可见性、触发工作流等）需获得用户显式授权。
- **凭据保护**：不在对话或日志中存储、展示或请求原始 GitHub token。
- **范围明确**：所有操作显式指定 `-R OWNER/REPO`，避免误操作到错误仓库。
- **不编造输出**：未实际执行验证的命令，不声称执行成功。

## License

本技能的原始文档与元数据采用 MIT 许可。GitHub CLI 及第三方项目遵循其各自的许可协议。