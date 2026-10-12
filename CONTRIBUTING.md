# Contributing

欢迎贡献！本仓库是柯影智航（KoyingPilot）的 Agent Skill 开发仓库。

## 目录结构

```
keying-agent-skills/
├── skills/development/          # 技能开发目录
│   ├── skill-project-analyst/
│   ├── skill-gh-cli/
│   ├── skill-bug-investigator/
│   ├── skill-code-reviewer/
│   ├── skill-code-refactor/
│   ├── skill-dependency-auditor/
│   ├── skill-architecture-reviewer/
│   ├── skill-test-engineer/
│   └── skill-workspace-environment-manager/
├── skills/data-analysis/        # 数据分析技能目录
│   └── skill-plot-generator/
├── catalog.json                 # 技能目录索引
├── README.md
├── LICENSE
└── CONTRIBUTING.md              # 本文件
```

## 创建新 Skill

1. 在 `skills/development/` 下创建新目录，命名格式 `skill-<kebab-case-name>`（或放至其他已有类别如 `skills/data-analysis/`）。
2. 参照现有 Skill 的结构创建以下文件：

| 文件 | 说明 |
|---|---|
| `SKILL.md` | 技能入口。必须以 YAML Frontmatter 开始（`---\nname: ...\ndescription: ...\n---`），后跟行为契约、触发条件、执行策略。 |
| `extension.json` | 机器可读元数据：`name` / `version` / `type: "skill"` / `author` / `license`。 |
| `references/guide.md` | 扩展参考手册，保持主 `SKILL.md` 轻量。 |
| `ANALYSIS_REPORT.md` | 设计决策、风险矩阵与验收清单。 |
| `README.md` | 使用文档与示例。 |
| `LICENSE` | MIT 许可。 |

3. 在根目录 `catalog.json` 的 `skills` 数组中添加新条目（仅适用于希望被 DSH Host 自动识别的 Skill；如仅作内部使用或演示，可省略此步）。

## Skill 结构约定

### SKILL.md

```yaml
---
name: my-skill
description: 一句话描述该技能的用途与触发场景。
---
```

Frontmatter 之后是技能正文，包含：

- **目的**：这个技能解决什么问题。
- **何时使用**：触发条件（用户请求的典型表述）。
- **何时不使用**：边界情况与排除条件。
- **操作流程**：执行步骤。
- **安全边界**：默认只读约束与需要授权的操作。
- **输出格式**：结构化输出模板。

### extension.json

```json
{
  "id": "my-skill",
  "name": "My Skill",
  "version": "0.1.0",
  "description": "详细描述。",
  "type": "skill",
  "author": "Keying Dev",
  "license": "MIT",
  "entry": "SKILL.md",
  "references": ["references/guide.md"],
  "tags": ["标签1", "标签2"]
}
```

## 本地验证

```bash
# 校验所有 extension.json
for f in skills/development/skill-*/extension.json; do
  python3 -c "import json; json.load(open('$f'))" && echo "OK: $f"
done
for f in skills/data-analysis/skill-*/extension.json; do
  python3 -c "import json; json.load(open('$f'))" && echo "OK: $f"
done

# 校验 catalog.json
python3 -c "import json; json.load(open('catalog.json'))" && echo "OK: catalog.json"
```

## 提交规范

- 提交信息使用中文或英文，描述变更目的。
- Skill 版本升级时，同步更新 `extension.json` 中的 `version` 字段。
- 不提交 `.DS_Store`、`node_modules/` 等无关文件（已配置 `.gitignore`）。

## License

所有贡献的 Skill 均采用 MIT 许可，详见各目录下的 `LICENSE` 文件。