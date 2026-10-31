# keying-agent-skills

> 柯影智航（KoyingPilot / DeepSeek Harness）Agent Skill 库 —— 可复用的领域工作流模板集合。

本仓库是 **Keying Dev** 维护的 Agent Skill 开发仓库。每个 Skill 都是一套声明式指令集（`SKILL.md` + `extension.json` + 参考文档），为 AI Agent 提供稳定、可复用的领域工作流。

## Skills 目录

| Skill | 标识 | 版本 | 用途 |
|---|---|---|---|
| [项目分析](./skills/development/skill-project-analyst) | `project-analyst` | v0.1.1 | 系统化分析陌生软件项目：技术栈、架构、入口、依赖、构建/测试/部署、风险与未知事项 |
| [GitHub 命令行工具](./skills/development/skill-gh-cli) | `gh-cli` | v0.1.1 | 通过官方 `gh` CLI 安全操作 GitHub 仓库、Issue、PR、Release、Actions |
| [故障调查技能](./skills/development/skill-bug-investigator) | `bug-investigator` | v0.1.0 | 软件故障根因分析：症状 → 触发条件 → 根因的证据链分级 |
| [代码审查技能](./skills/development/skill-code-reviewer) | `code-reviewer` | v0.1.0 | 面向代码文件、Git Diff 与 Pull Request 的证据驱动代码审查技能；检查正确性、安全性、可靠性、性能与维护性，定位可操作缺陷，给出风险分级、验证状态和最小修复建议。 |
| [代码重构技能](./skills/development/skill-code-refactor) | `code-refactor` | v0.1.0 | 面向代码结构改进和技术债治理的重构技能，通过行为基线、影响分析、小步变更和回归验证降低维护成本，并明确区分重构与功能变更。 |
| [依赖审计技能](./skills/development/skill-dependency-auditor) | `dependency-auditor` | v0.1.0 | 面向软件依赖和供应链风险的审计技能，核对 manifest、lockfile、实际安装版本和构建环境，识别兼容性、漏洞证据、许可证和来源可信度风险。 |
| [架构审查技能](./skills/development/skill-architecture-reviewer) | `architecture-reviewer` | v0.1.0 | 面向软件系统架构与设计方案的证据驱动审查，评估模块边界、依赖方向、演进性、可靠性、数据流、可运维性及非功能需求权衡。 |
| [软件测试工程师](./skills/development/skill-test-engineer) | `test-engineer` | v0.1.0 | 面向需求、代码变更和故障修复的测试工程技能，建立可追溯测试目标、测试用例、安全执行计划、结果证据及剩余风险报告。 |
| [工作区环境管理器](./skills/development/skill-workspace-environment-manager) | `workspace-environment-manager` | v0.1.0 | 诊断并准备受限 AI 代理工作区中的隔离项目环境。在考虑安装前优先复用可访问的主机工具；区分 PATH 缺失、权限不足、沙箱限制和执行失败；将项目依赖和生成状态保留在工作区内；仅在必要时使用经过验证的网络镜像。 |
| [学位论文格式化](./skills/academic/skill-academic-paper-formatter) | `academic-paper-formatter` | v0.1.1 | 优先适配柯影智航，同时兼容其他支持 MCP 与 Agent Skill 的智能体。依据学校提供的 Word 模板分析、格式化并校验中国本科、硕士和博士学位论文。 |
| [绘图生成器](./skills/data-analysis/skill-plot-generator) | `plot-generator` | v0.1.0 | 使用 MCP Research-Draw 工具生成统计图表、领域专用图、3D 图和地图可视化。提供可用图表列表、参数说明以及按数据渲染图表的完整指引。 |

> [视频格式转换](./skills/media/skill-video-format-converter)（`video-format-converter`）已注册到 `catalog.json`，见上表。命令示例与端到端流程均用 FFmpeg 9.0.2 在 `data-demo/` 的真实 1080p 源上实测确认，共三轮：19 项 E2E 场景（`ffprobe` 缺失 / 可用两种宿主注入状态，结论零漂移）、md5/ffprobe 复核实录（4 种场景判别表 + 5 个复核用例），见其 `ANALYSIS_REPORT.md` 第 8、9 节。Agent 行为评估用例 VD-E01..E08 已有 7 PASS / 1 PARTIAL：E01/E02/E03/E05/E07 有完整运行时证据，E06/E08 在真实对话中已验证，仅 E04（工具缺失）因当前会话环境已注入 `ffprobe` 而无法在本轮复现。


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

多个 Skill 可组合成完整的软件工程工作流：

```text
Project Analyst        → 理解项目（这个项目是怎么工作的？）
        ↓
Bug Investigator       → 调查故障（为什么坏了？）
        ↓
Coding Agent           → 实施修复（怎么把它修好？）
        ↓
Tests                  → 验证结果
```

## 仓库自检

提交前运行一致性校验（只读，不修改文件）：

```bash
python3 scripts/validate_skills.py
```

脚本检查约定文件是否齐全、`name` 与目录名是否一致、`extension.json` 与 `catalog.json` 的字段与路径是否有效、Markdown 相对链接是否存在、是否残留备份文件。仓库目前**没有**自动化测试与 CI，该脚本只做静态检查。

## License

所有 Skill 均采用 [MIT License](./LICENSE)。

## Author

**Keying Dev**