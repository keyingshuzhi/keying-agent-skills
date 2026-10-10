# Dependency Auditor（依赖审计技能）

> 依赖风险必须对应版本、来源和可核验的公告；不以猜测替代漏洞证据。

![Version](https://img.shields.io/badge/version-0.1.0-blue) ![License](https://img.shields.io/badge/license-MIT-green)

## 定位

面向软件依赖和供应链风险的审计技能，核对 manifest、lockfile、实际安装版本和构建环境，识别兼容性、漏洞证据、许可证和来源可信度风险。

## 触发示例

- 审计 Python / npm 项目依赖是否存在风险。
- 检查 lockfile 与实际安装版本是否一致。
- 评估升级某个依赖的兼容性影响。
- 梳理项目许可证与依赖供应链风险。

## 使用与安装

将整个 `skill-dependency-auditor` 目录复制到受支持的 Agent Skills 搜索目录，具体路径以 Agent 宿主产品要求为准。

```bash
cp -r skills/development/skill-dependency-auditor /path/to/agent/skills/
```

该 Skill 仅提供方法与工作流；文件、终端、Git 和网络工具须由宿主环境提供，不暗示自动获得权限。

## 文件结构

```text
skill-dependency-auditor/
├── SKILL.md
├── extension.json
├── README.md
├── ANALYSIS_REPORT.md
├── LICENSE
└── references/
    └── guide.md
```

## 标准流程

1. **确认审计边界**：确定生态系统、工作区、环境、直接与间接依赖范围，以及用户关注的安全/兼容/许可风险。
2. **盘点可重复证据**：读取 manifests、lockfiles、SBOM、依赖树、运行时与系统架构；识别单一项目、多工作区和条件依赖。
3. **核对解析结果**：对比声明约束、锁定版本和实际安装；未能访问安装环境时仅报告锁文件所示版本，不冒充运行时事实。
4. **查证外部风险**：若可访问可信公告/数据库，记录漏洞 ID、影响版本范围、修复版本、查询时间、来源及适用条件；离线时仅报告未核验风险，不能断言最新 CVE 状态。
5. **评估影响**：结合实际使用、可达性、运行时平台、传递路径及许可证条件分类；CVSS 不等于项目实际风险。
6. **制定处置建议**：按必要性、兼容性与迁移成本建议最小版本升级、隔离、替换或保持现状；不自动执行升级。
7. **生成审计报告**：包含范围、时间点、证据、严重度、可信度、建议和遗漏范围；不得声称完成全面供应链审计。

## 安全性

- 默认分析优先，显式只读要求必须遵守。
- 报告真实工具执行证据；未执行、未验证、不可访问要标注。
- 高副作用操作需隔离环境与明确授权。

## 评估状态

`ANALYSIS_REPORT.md` 提供待执行 Evaluation Cases。**初稿结构验证不代表 Agent 实测通过。**

## License

MIT © 2026 Keying Dev
