# keying-agent-skills

> 柯影智航（KoyingPilot / DeepSeek Harness）Agent Skill 库 —— 可复用的领域工作流模板集合。

本仓库是 **Keying Dev** 维护的 Agent Skill 开发仓库。每个 Skill 都是一套声明式指令集（`SKILL.md` + `extension.json` + 参考文档），为 AI Agent 提供稳定、可复用的领域工作流。

## Skills 目录

| Skill | 标识 | 版本 | 用途 |
|---|---|---|---|
| [Project Analyst](./skills/development/skill-project-analyst) | `project-analyst` | v0.1.1 | 系统化分析陌生软件项目：技术栈、架构、入口、依赖、构建/测试/部署、风险与未知事项 |
| [GitHub CLI](./skills/development/skill-gh-cli) | `gh-cli` | v0.1.1 | 通过官方 `gh` CLI 安全操作 GitHub 仓库、Issue、PR、Release、Actions |
| [Bug Investigator](./skills/development/skill-bug-investigator) | `bug-investigator` | v0.1.0 | 软件故障根因分析：症状 → 触发条件 → 根因的证据链分级 |

## 安装

每个 Skill 目录可直接复制到 KoyingPilot 的 Skill 目录中使用。详细安装步骤见各 Skill 的 `README.md`。

```bash
# 示例：安装 Project Analyst
cp -r skills/development/skill-project-analyst /path/to/koying-pilot/skills/
```

## Skill 结构约定

每个 Skill 遵循以下结构：

```text
skill-name/
├── SKILL.md              # 技能入口：YAML Frontmatter + 行为契约 + 触发条件
├── extension.json        # 机器可读元数据（name / version / type: skill）
├── references/
│   └── guide.md          # 扩展参考手册
├── ANALYSIS_REPORT.md     # 设计决策与评估基线
├── README.md              # 使用文档
└── LICENSE                # MIT 许可
```

## 工作流串联

三个 Skill 可组合成完整的软件工程工作流：

```text
Project Analyst        → 理解项目（这个项目是怎么工作的？）
       ↓
Bug Investigator       → 调查故障（为什么坏了？）
       ↓
Coding Agent           → 实施修复（怎么把它修好？）
       ↓
Tests                  → 验证结果
```

## License

所有 Skill 均采用 [MIT License](./LICENSE)。

## Author

**Keying Dev**