# keying-agent-skills

> 柯影智航（KoyingPilot / DeepSeek Harness）Agent Skill 库 —— 可复用的领域工作流模板集合。

本仓库是 **Keying Dev** 维护的 Agent Skill 开发仓库。每个 Skill 都是一套声明式指令集（`SKILL.md` + `extension.json` + 参考文档），为 AI Agent 提供稳定、可复用的领域工作流。

## Skills 目录

| Skill | 标识 | 版本 | 用途 |
|---|---|---|---|
| [Project Analyst](./skills/development/skill-project-analyst) | `project-analyst` | v0.1.1 | 系统化分析陌生软件项目：技术栈、架构、入口、依赖、构建/测试/部署、风险与未知事项 |
| [GitHub CLI](./skills/development/skill-gh-cli) | `gh-cli` | v0.1.1 | 通过官方 `gh` CLI 安全操作 GitHub 仓库、Issue、PR、Release、Actions |
| [Bug Investigator](./skills/development/skill-bug-investigator) | `bug-investigator` | v0.1.0 | 软件故障根因分析：症状 → 触发条件 → 根因的证据链分级 |
| [Code Reviewer](./skills/development/skill-code-reviewer) | `code-reviewer` | v0.1.0 | 面向代码文件、Git Diff 与 Pull Request 的证据驱动代码审查技能；检查正确性、安全性、可靠性、性能与维护性，定位可操作缺陷，给出风险分级、验证状态和最小修复建议。 |
| [Code Refactor](./skills/development/skill-code-refactor) | `code-refactor` | v0.1.0 | 面向代码结构改进和技术债治理的重构技能，通过行为基线、影响分析、小步变更和回归验证降低维护成本，并明确区分重构与功能变更。 |
| [Dependency Auditor](./skills/development/skill-dependency-auditor) | `dependency-auditor` | v0.1.0 | 面向软件依赖和供应链风险的审计技能，核对 manifest、lockfile、实际安装版本和构建环境，识别兼容性、漏洞证据、许可证和来源可信度风险。 |
| [Architecture Reviewer](./skills/development/skill-architecture-reviewer) | `architecture-reviewer` | v0.1.0 | 面向软件系统架构与设计方案的证据驱动审查，评估模块边界、依赖方向、演进性、可靠性、数据流、可运维性及非功能需求权衡。 |
| [Test Engineer](./skills/development/skill-test-engineer) | `test-engineer` | v0.1.0 | 面向需求、代码变更和故障修复的测试工程技能，建立可追溯测试目标、测试用例、安全执行计划、结果证据及剩余风险报告。 |
| [Workspace Environment Manager](./skills/development/skill-workspace-environment-manager) | `workspace-environment-manager` | v0.1.0 | Diagnose and prepare isolated project environments in restricted AI-agent workspaces. Reuse accessible host tools before considering installation; distinguish missing tools from PATH, permission, sandbox and execution failures; keep project dependencies and generated state inside the workspace; use vetted network mirrors only when needed. |

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