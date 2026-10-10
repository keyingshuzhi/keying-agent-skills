# Code Reviewer

> 面向代码、Git Diff 与 Pull Request 的证据驱动 Agent Skill。  
> **先确认上下文，再报告缺陷；没有证据，不制造结论。**

![Version](https://img.shields.io/badge/version-0.1.0-blue)
![Type](https://img.shields.io/badge/type-Agent%20Skill-purple)
![License](https://img.shields.io/badge/license-MIT-green)

## 定位

Code Reviewer 是 `keying-agent-skills` 的第四个软件开发技能。它对代码变更进行正确性、安全性、可靠性、性能和维护性审查，要求每个正式 Finding 包含位置、触发条件、影响、证据、风险等级与最小修复建议。

技能**默认只读**，不自动修改代码、创建 PR 评论、提交或合并。

## 适用请求

- `Review 一下这个 PR，找出可能阻塞合并的问题。`
- `审查 src/auth.py 的权限判断，只分析不要修改。`
- `检查这次 Git Diff 是否引入回归。`
- `对这段 Python 代码做证据驱动的 Code Review。`

## 核心流程

```text
确定范围 → 理解契约 → 风险检查 → 证据验证 → 误报排除 → Findings → 总结剩余风险
```

## 证据和风险的两个维度

证据状态：`Confirmed` / `Probable` / `Needs Verification`。

严重度：`Critical` / `High` / `Medium` / `Low`。

请勿将“没有运行测试”描述成“测试通过”，也不要将“没有发现问题”描述成“没有 Bug”。

## 安装

从仓库根目录复制完整目录到支持该技能格式的 Agent Skill 路径：

```bash
cp -r skills/development/skill-code-reviewer /path/to/koying-pilot/skills/
```

安装后，根据 Agent 宿主的加载规则启用该 Skill。实际兼容性以对应宿主的验证结果为准。

## 文件结构

```text
skill-code-reviewer/
├── SKILL.md
├── extension.json
├── README.md
├── ANALYSIS_REPORT.md
├── LICENSE
└── references/
    └── guide.md
```

## 与现有 Skill 协作

| Skill | 回答的问题 |
|---|---|
| Project Analyst | 项目如何组织与运行？ |
| Bug Investigator | 一个具体故障为何发生？ |
| GitHub CLI | 如何安全查询与操作 GitHub？ |
| **Code Reviewer** | 代码/变更是否存在可证实的缺陷与风险？ |

## License

[MIT License](./LICENSE) · **Keying Dev**
