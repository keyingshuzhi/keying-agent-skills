# Code Refactor（代码重构技能）

> 先建立行为基线，再做最小结构变更；未经验证不得宣称行为保持。

![Version](https://img.shields.io/badge/version-0.1.0-blue) ![License](https://img.shields.io/badge/license-MIT-green)

## 定位

面向代码结构改进和技术债治理的重构技能，通过行为基线、影响分析、小步变更和回归验证降低维护成本，并明确区分重构与功能变更。

## 触发示例

- 保持功能不变，拆分这个过长的函数。
- 清理重复代码并改善可维护性。
- 整理模块边界，控制重构风险。
- 为遗留模块制定分阶段重构计划。

## 使用与安装

将整个 `skill-code-refactor` 目录复制到受支持的 Agent Skills 搜索目录，具体路径以 Agent 宿主产品要求为准。

```bash
cp -r skills/development/skill-code-refactor /path/to/agent/skills/
```

该 Skill 仅提供方法与工作流；文件、终端、Git 和网络工具须由宿主环境提供，不暗示自动获得权限。

## 文件结构

```text
skill-code-refactor/
├── SKILL.md
├── extension.json
├── README.md
├── ANALYSIS_REPORT.md
├── LICENSE
└── references/
    └── guide.md
```

## 标准流程

1. **定义重构契约**：明确哪些可观察行为必须保持：输入输出、异常、API/ABI、持久化格式、权限、时间敏感行为和性能约束。
2. **定位重构目标**：基于具体异味和证据，例如重复逻辑、循环依赖、过大职责；记录范围和不包含的模块。
3. **建立行为基线**：检查已有测试、接口契约和关键边界；无法覆盖时优先建议特征测试（characterization test），未执行则不能声称基线已验证。
4. **规划小步变更**：按依赖顺序列出可独立回滚的变更步骤，每步配套验证与回退方案；避免无关格式化。
5. **执行受控重构**：用户授权修改后才编辑；不改公共接口或数据迁移语义，除非明确批准并单独标记行为变更。
6. **验证差异**：查看 diff，执行相关测试（如安全可用），比较关键输入输出和失败路径；无法验证则列为残余风险。
7. **交付变更说明**：说明改动文件、预期收益、验证结果、兼容性风险、未验证项及回滚点。

## 安全性

- 默认分析优先，显式只读要求必须遵守。
- 报告真实工具执行证据；未执行、未验证、不可访问要标注。
- 高副作用操作需隔离环境与明确授权。

## 评估状态

`ANALYSIS_REPORT.md` 提供待执行 Evaluation Cases。**初稿结构验证不代表 Agent 实测通过。**

## License

MIT © 2026 Keying Dev
