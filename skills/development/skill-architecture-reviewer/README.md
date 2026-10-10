# Architecture Reviewer（架构审查技能）

> 架构结论依赖上下文与约束；没有目标就没有唯一最优架构。

![Version](https://img.shields.io/badge/version-0.1.0-blue) ![License](https://img.shields.io/badge/license-MIT-green)

## 定位

面向软件系统架构与设计方案的证据驱动审查，评估模块边界、依赖方向、演进性、可靠性、数据流、可运维性及非功能需求权衡。

## 触发示例

- 审查当前服务拆分和模块边界是否合理。
- 评估单体与微服务的演进取舍。
- 帮我 review 一份架构设计文档。
- 分析系统的单点故障、扩展瓶颈和关键技术债。

## 使用与安装

将整个 `skill-architecture-reviewer` 目录复制到受支持的 Agent Skills 搜索目录，具体路径以 Agent 宿主产品要求为准。

```bash
cp -r skills/development/skill-architecture-reviewer /path/to/agent/skills/
```

该 Skill 仅提供方法与工作流；文件、终端、Git 和网络工具须由宿主环境提供，不暗示自动获得权限。

## 文件结构

```text
skill-architecture-reviewer/
├── SKILL.md
├── extension.json
├── README.md
├── ANALYSIS_REPORT.md
├── LICENSE
└── references/
    └── guide.md
```

## 标准流程

1. **确定架构问题**：抽取业务目标、规模、团队能力、预算、SLO、安全/合规/数据约束以及时间跨度；缺失的输入明确标注。
2. **构建可核验架构视图**：依据代码、配置、部署清单、架构图及接口契约建立组件/依赖/数据流/信任边界；区分事实与假设。
3. **识别关键质量属性**：围绕可用性、性能、可维护性、可扩展性、安全、交付复杂度、可观测性排序，不平均分配关注。
4. **分析架构风险**：审查耦合、循环依赖、失效域、同步链路、状态一致性、权限边界、容量、恢复目标等；联系真实场景。
5. **对比候选方案**：至少提供保持现状方案，在必要时提出可选方案；比较复杂度、迁移成本、收益、风险与适用条件。
6. **形成决策记录**：给出建议、触发条件、假设、不确定性、验证实验和撤销/迁移路径；如关键约束缺失，结论必须有条件。
7. **提供审查产物**：输出架构视图、风险列表、权衡矩阵与 ADR 草案（如适用），不得擅自修改部署结构。

## 安全性

- 默认分析优先，显式只读要求必须遵守。
- 报告真实工具执行证据；未执行、未验证、不可访问要标注。
- 高副作用操作需隔离环境与明确授权。

## 评估状态

`ANALYSIS_REPORT.md` 提供待执行 Evaluation Cases。**初稿结构验证不代表 Agent 实测通过。**

## License

MIT © 2026 Keying Dev
