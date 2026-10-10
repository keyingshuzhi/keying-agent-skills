# Test Engineer（软件测试工程师）

> 测试结论必须来自真实执行证据；未运行不等于通过。

![Version](https://img.shields.io/badge/version-0.1.0-blue) ![License](https://img.shields.io/badge/license-MIT-green)

## 定位

面向需求、代码变更和故障修复的测试工程技能，建立可追溯测试目标、测试用例、安全执行计划、结果证据及剩余风险报告。

## 触发示例

- 给这个模块设计单元测试 / 集成测试。
- 评估这个 PR 的回归测试范围。
- 在安全环境中运行已有测试并分析失败。
- 为这次修复制定验收测试。

## 使用与安装

将整个 `skill-test-engineer` 目录复制到受支持的 Agent Skills 搜索目录，具体路径以 Agent 宿主产品要求为准。

```bash
cp -r skills/development/skill-test-engineer /path/to/agent/skills/
```

该 Skill 仅提供方法与工作流；文件、终端、Git 和网络工具须由宿主环境提供，不暗示自动获得权限。

## 文件结构

```text
skill-test-engineer/
├── SKILL.md
├── extension.json
├── README.md
├── ANALYSIS_REPORT.md
├── LICENSE
└── references/
    └── guide.md
```

## 标准流程

1. **定义测试契约**：确定被测对象、预期行为、可观察输出、成功判据及非目标；缺少需求时从代码与文档提取候选预期并标注待确认。
2. **发现测试环境**：识别语言、框架、测试目录、manifest/lockfile、现有命令、fixture、mock、依赖服务及CI配置；不臆造运行条件。
3. **设计测试矩阵**：按风险选择单元/集成/契约/E2E/回归；包含正常、边界、异常、状态转换与权限场景，显式区分输入、预期、依据。
4. **制定执行计划**：先检查工具权限、测试写入范围、数据隔离、外部请求、密钥与清理方案；无法隔离或可能影响生产时仅提供计划。
5. **安全执行（条件允许）**：优先运行最小相关测试集，再按需要扩大；记录原始命令、工作目录、环境、退出码、测试报告位置与关键输出。用户要求只分析时不得执行有写入的测试。
6. **解释结果**：按用例标记 PASS/FAIL/BLOCKED/SKIPPED/NOT RUN；区分产品缺陷、测试缺陷、环境失败和不稳定测试；不能把断言失败直接定性为产品 Bug。
7. **报告与移交**：记录覆盖目标、证据、失败复现方式、未执行项和剩余风险；仅在证据足够时向 bug-investigator 移交具体故障。

## 安全性

- 默认分析优先，显式只读要求必须遵守。
- 报告真实工具执行证据；未执行、未验证、不可访问要标注。
- 高副作用操作需隔离环境与明确授权。

## 评估状态

`ANALYSIS_REPORT.md` 提供待执行 Evaluation Cases。**初稿结构验证不代表 Agent 实测通过。**

## License

MIT © 2026 Keying Dev
