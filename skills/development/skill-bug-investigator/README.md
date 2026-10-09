# Bug Investigator

> 面向软件故障调查与根因分析的 Agent Skill。  
> **先调查，再修复；先证据，再结论。**

![Version](https://img.shields.io/badge/version-0.1.0-blue)
![Type](https://img.shields.io/badge/type-Agent%20Skill-purple)
![License](https://img.shields.io/badge/license-MIT-green)

## 简介

**Bug Investigator（故障调查技能）** 是一个面向软件开发场景的 Agent Skill，用于调查程序异常、运行错误、启动失败、API 故障、配置问题、依赖异常以及其他非预期行为。

它的核心目标不是让 Agent 在看到错误后立即修改代码，而是建立一套相对稳定、以证据为基础的故障调查流程：

```text
发现故障
   ↓
定义问题
   ↓
收集证据
   ↓
定位故障边界
   ↓
建立候选原因
   ↓
验证与排除
   ↓
确定根因
   ↓
给出下一步建议
```

## 目录结构

```text
bug-investigator/
├── ANALYSIS_REPORT.md
├── LICENSE
├── extension.json
├── README.md
├── SKILL.md
└── references/
    └── guide.md
```

## 根因分级

| 状态 | 含义 |
|---|---|
| **已确认** | 存在直接证据建立原因与故障之间的因果关系 |
| **高可能** | 多个证据共同支持，但仍缺少一个重要验证环节 |
| **可能** | 原因合理，但证据不足或仍存在其他无法排除的解释 |
| **未知** | 当前信息不足以形成可靠结论 |

## 与其他 Skill 的区别

| | Project Analyst | Bug Investigator |
|---|---|---|
| 中文定位 | 项目分析技能 | 故障调查技能 |
| 分析对象 | 整个软件项目 | 一个具体故障 |
| 核心问题 | 项目是怎么工作的？ | 为什么发生这个故障？ |
| 主要范围 | Project | Bug / Incident |
| 默认行为 | 建立项目认知 | 建立故障证据链 |
| 主要结果 | 项目分析 | 根因诊断 |

## 推荐工作流

```text
Project Analyst      ↓ 理解项目
Bug Investigator     ↓ 调查故障
Coding Agent         ↓ 实施修复
Tests                ↓ 验证结果
```

## License

本项目采用 [MIT License](./LICENSE)。

## Author

**Keying Dev**

Bug Investigator 是面向 Agent 软件工程工作流构建的故障调查技能。

> **Diagnose first. Fix second.**