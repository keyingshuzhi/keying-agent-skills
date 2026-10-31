# Changelog

所有 notable 的变更在此记录。

格式基于 [Keep a Changelog](https://keepachangelog.com/)。

日期为仓库提交日期（本地时区 Asia/Shanghai）。

---

## [Unreleased]

### 新增
- `skills/media/skill-video-format-converter`：基于 FFmpeg 的视频格式转换技能，补齐 `README.md`、`ANALYSIS_REPORT.md`、`LICENSE` 与 `references/guide.md`。**已注册到 `catalog.json`**（id: `video-format-converter`）。
- `scripts/validate_skills.py`：仓库一致性校验脚本，检查约定文件、`name` 与目录名一致性、`extension.json` / `catalog.json` 字段与路径、Markdown 相对链接、残留备份文件。

### 变更
- 统一全部 12 个技能的 `extension.json` 为单一 schema（`name` / `displayName` / `displayNameZh` / `version` / `description` / `type` / `author` / `publisher` / `license` / `entry` / `references` / `permissions` / `tags`），移除历史上的 `id` 字段与 `skills[]` 包装，并显式声明宿主可读的 `permissions`。
- `gh-cli` 技能标识统一：`SKILL.md` frontmatter 由 `github-cli` 改为 `gh-cli`，与目录 `skill-gh-cli`、`catalog.json` 的 `id` 一致。
- `workspace-environment-manager` 补齐 `README.md`、`ANALYSIS_REPORT.md`、`LICENSE`、`references/guide.md`（此前 `SKILL.md` 引用的 guide 不存在）。
- `plot-generator` 补齐 `LICENSE`，重写 `README.md`（修正安装路径与占位目录说明），删除 `SKILL.md.bak`、`guide.md.bak`。
- `academic-paper-formatter` 的 `references/guide.md` 用 `<MCP_REPO>` / `<UV_PATH>` 占位替代机器专属绝对路径。
- `CONTRIBUTING.md` 重写：补充 `media/` 分类、`scripts/`、各文件必需性、`extension.json` 字段表（标注宿主是否读取）与校验方式。
- 根 `README.md` 同步技能表与工作流说明。
- `.gitignore` 增加 `data-demo/`（本地演示素材，不随仓库分发）。

### 修复
- 修正 `CHANGELOG` 中与提交历史不符的日期（原记录为 2026-01-09，实际首批提交为 2026-10-09）与技能数量描述。
- `video-format-converter` 技能内容修复（用宿主 FFmpeg 9.0.2 逐条实测确认）：
  - 批量转换示例的输入输出同名 → 改为输出新路径，避免原地覆盖源文件。
  - `-i @files.txt` 缺少 `-f concat -safe 0` → 补齐并说明 `file '/绝对路径'` 格式。
  - `-framerate 60` 作为输出选项无效 → 改为 `-r 60`，并注明 `-framerate` 是输入侧选项。
  - `-map:s:s 0` 在无字幕流时静默"成功" → 改为 `-map 0:s:0`（E2E 后进一步订正，见下条）。
  - `SKILL.md` 引用 5 个不存在的文件（`command_list.md` / `code_examples.md` / `convert_video.py` / `batch_convert.py` / 单元测试脚本）→ 删除死链，统一指向 `references/guide.md`。
- `catalog.json` 版本由 `0.1.0` 升至 `0.1.1`，并新增 `video-format-converter` 条目（第 12 个注册技能）。
- `video-format-converter` 端到端（E2E）实测后修复（用 `data-demo/` 真实 1080p 源，FFmpeg 9.0.2，19 项场景见该技能 `ANALYSIS_REPORT.md` 第 8 节）：
  - `ffprobe` 在首轮宿主注入中不在 `PATH`（只注入了 `ffmpeg` 软壳）→ `SKILL.md` 检查步骤新增"先探测 ffprobe"；`references/guide.md` 新增"ffprobe 可用性"一节，给出 `command -v` → 从 ffmpeg 实现路径反推 → `ffmpeg -i` 兜底的三级降级方案。宿主更新后 `ffprobe` 已可用，文档复测后改为"先探测、缺失再降级"，不再断言永久缺失。
  - `-map 0:s:0` 在无字幕时并非"可预期的报错"，而是退出码 234 硬失败；`-map:s:s 0` 被静默忽略并导出全部流 → 改为同时给出 `-map 0:s:0` 与 `-map 0:s:0?`，并说明 `-map:s:s 0` 不是合法语法。
  - 指南误称 `libopus` / `libvorbis` "同样需要 `-strict -2`" → 实测 `libopus` 不需（退出码 0、输出 48kHz 立体声），`libvorbis` 本构建未编入（`Unknown encoder`，退出码 8）；音频编码表两处改正，并推荐用 `libopus` 替代 `opus`。
  - 补充三个"退出码 0 也不代表成功"的陷阱：输出已存在未加 `-y`（打印 `Not overwriting - exiting` 但退出码 0）、编码/容器不匹配会留下几百字节残file（退出码 234）、concat 列表相对路径是相对列表文件所在目录解析（退出码 254）。"转换前检查清单"由 5 条扩为 6 条。
  - 补充 md5 的适用边界：同机同参数下 x265/x264 是逐字节可复现的（x265 连跑 3 轮、x264 连跑 2 轮、全长含音轨连跑 2 轮 md5 各自一致），md5 差异来自命令差异而非多线程非确定性——仅加 `-tag:v hvc1` 就把 md5 从 `5b74ce1e…` 变成 `fe8bffc2…`（大小都是 5060317 B），但剥掉容器只比 HEVC 裸流两者完全一致（`1a22c42a…` / 4703704 B），变的只是 `codec_tag_string`。因此 md5 只能证明"文件没变"，不能证明"转码成功"。
  - 新增"转换后复核（md5 + ffprobe）"一节：给出四种场景的判别表（正常 / 目标已存在未加 `-y` / 编码容器不匹配 / 输入缺失）与可复用的 `verify.sh` 复核脚本（源指纹 → 大小 → ffprobe 解析性 → 编码/分辨率/音频断言 → md5 双侧复核）。实跑 5 个用例，2 PASS / 3 FAIL，三个 FAIL 分别抓到残file、期望值写错、输出=输入，均为预期行为。
  - `ffprobe` 可用后的第二轮 E2E 复测：19 项场景结论零漂移（失败码、文件大小、编码参数两轮完全一致），仅 T1 因宿主注入变化而不同。

---

## [0.1.1] — 2026-10-09

### 变更
- `project-analyst` 升级至 v0.1.1：精简 `references/guide.md`，强化输出预算考量。
- `gh-cli` 升级至 v0.1.1：中文本地化，新增工作区 `gh` 探测、仓库上下文自动解析、认证检查、非交互模式、速率限制处理等 12 项优化。

---

## [0.1.0] — 2026-10-09

### 新增
- 建立仓库结构：`.gitignore`、`catalog.json`、`CONTRIBUTING.md`、`CHANGELOG.md`。
- `project-analyst` v0.1.0：项目分析 8 Phase 流程与 15 节报告模板。
- `gh-cli` v0.1.0：GitHub CLI 操作技能，覆盖仓库 / Issue / PR / Release / Actions。
- `bug-investigator` v0.1.0：证据模型与根因分级（已确认 / 高可能 / 可能 / 未知）。
- 后续补齐 `code-reviewer`、`code-refactor`、`dependency-auditor`、`architecture-reviewer`、`test-engineer`、`workspace-environment-manager`。
