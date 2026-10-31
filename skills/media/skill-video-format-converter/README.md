# 视频格式转换技能

> 基于 FFmpeg 的视频格式转换、参数调整与批量处理 Agent Skill。
> **先探查源文件，再转码；不覆盖源文件，不用猜测的编码参数。**

![Version](https://img.shields.io/badge/version-0.1.0-blue)
![Type](https://img.shields.io/badge/type-Agent%20Skill-purple)
![License](https://img.shields.io/badge/license-MIT-green)

## 定位

本技能把"视频格式转换"拆成可复核的步骤：**检查源文件 → 选择目标格式与参数 → 用户确认 → 执行 FFmpeg → 验证输出**。技能本身不含可执行代码，只规定如何调用系统 FFmpeg 与 ffprobe，并要求所有写操作在用户授权后发生。

## 适用请求

- `把这个 MP4 转成 H.265 的 MP4，保持 1080p。`
- `把这一批 AVI 都转成 MP4，保留原文件名。`
- `查看这个视频的分辨率、码率和时长。`
- `压一下这个视频，文件太大了。`

## 不适用

- 视频剪辑、特效、多画面合成（需要专业剪辑工具）。
- 纯音频处理。
- 安装或升级系统 FFmpeg。
- 需要 DRM 解密或绕过版权保护的素材。

## 核心流程

```text
建立上下文 → 检查（只读） → 计划（不改文件） → 应用（需授权） → 验证
```

## 工作模式

| 模式 | 是否写文件 | 说明 |
| --- | --- | --- |
| 检查（默认） | 否 | `ffmpeg -version`、`ffprobe` 查询源文件信息 |
| 计划 | 否 | 选定编码/容器/分辨率/码率，给出命令与批处理顺序，等用户确认 |
| 应用 | 是 | 执行转码，输出写入工作区新文件，需显式授权 |
| 验证 | 否 | 用 `ffprobe` 复核输出时长、编码、分辨率与文件大小 |

## 安装

```bash
cp -r skills/media/skill-video-format-converter /path/to/koying-pilot/skills/
```

前置条件：主机存在 `ffmpeg` 与 `ffprobe`。若当前环境只在宿主缓存中提供，请用其绝对路径调用；技能不会自行安装系统级工具。

## 文件结构

```text
skill-video-format-converter/
├── SKILL.md                # 技能入口：模式、流程与安全边界
├── extension.json          # 机器可读元数据
├── references/
│   └── guide.md            # 容器/编码说明、常用命令与参数表
├── ANALYSIS_REPORT.md      # 设计决策、风险与验收清单
├── README.md               # 本文件
└── LICENSE                 # MIT
```

## 安全边界

- 默认只读：检查模式只查询信息，不生成文件。
- 不覆盖源文件：输出必须是新路径；覆盖已有输出需用户明确同意。
- 不解码受保护内容，不代替用户判断版权授权。
- 不安装系统软件；缺少 FFmpeg 时报告限制并给出安装建议。

## License

MIT，详见 [LICENSE](./LICENSE)。
