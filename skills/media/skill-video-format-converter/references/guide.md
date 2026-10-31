# 视频格式转换参考指南

## 支持的功能

FFmpeg 提供了丰富的视频格式转换功能，包括以下几类：

### 1. 容器格式转换
- **文件容器格式**：转换视频文件容器，保持视频内容不变
  - MP4 → MKV、AVI → MP4、MOV → WMV 等
  - 支持所有主要视频容器格式之间的互转

### 2. 视频编码格式转换
- **视频编解码器**：改变视频编码格式，影响视频质量和文件大小
  - H.264 → H.265（HEVC，高压缩率）
  - MPEG-2 → H.264（H.264，广泛兼容）
  - VP9 → AV1（新一代开源编码）
  - ProRes → H.264（电影级编码到网络编码的转换）

### 3. 视频参数调整
- **视频属性**：调整视频的基本参数
  - 分辨率调整：720p → 1080p、4K → 1080p
  - 帧率设置：24fps → 30fps、30fps → 60fps
  - 比特率控制：低比特率（300kbps）→ 高比特率（8Mbps）
  - 采样率：音频采样率调整

### 4. 音频处理
- **音频轨道**：处理音频流
  - 音频编码格式转换（AAC → MP3、FLAC → AAC 等）
  - 音频通道处理（单声道 → 立体声 → 5.1 声道等）
  - 音频时长剪辑

### 5. 元信息处理
- **视频标签**：提取和嵌入视频元信息
  - 视频时长、标题、作者
  - 录制日期、拍摄设备
  - 许可证信息

### 6. 批量处理
- **多文件转换**：同时处理多个视频文件
  - 统一格式转换
  - 批量重命名
  - 批量质量优化

## 实测支持的格式与编码（FFmpeg 9.0.2）

以下范围用宿主 FFmpeg 9.0.2（配置含 `--enable-libx264 --enable-libx265 --enable-libsvtav1 --enable-libvpx --enable-libopus --enable-libmp3lame --enable-libdav1d`）逐条实测确认，**不是**"所有格式都能转"的宣传表述。

### 容器（可写入输出）
MP4、MKV、AVI、MOV、WMV、FLV、WebM、M4V、TS、MPEG、OGG、ASF、3GP/3G2、MXF、DV、FFV1、Matroska、MPEG-TS raw、以及 BMP/PNG/JPEG/TIFF/EXR/SGI 等图像序列管道。

### 视频编码（`-c:v`，全部实测通过）
`libx264` / `libx264rgb`（H.264）、`libx265`（H.265/HEVC）、`libsvtav1`（AV1）、`libvpx`（VP8）、`libvpx-vp9`（VP9）、`mpeg4`、`mpeg1video` / `mpeg2video`、`prores` / `prores_aw` / `prores_ks`、`dvvideo` / `ffv1` / `magicyuv` / `qtrle` / `utvideo` / `rawvideo`（无损）、`wmv1` / `wmv2` / `flv` / `msvideo1`。

输入侧解码器还支持读入 H.264、H.265、AV1、VP9、ProRes、MPEG-2、MPEG-4、MPEG-1、VC-1、RealVideo、Theora、Dirac、MJPG、PNG、GIF 等。

> **VP8 注意**：`-c:v vp8` 对合成测试源（`testsrc`）会失败，需用真实编码过的视频作输入，或直接用 `-c:v libvpx`。

### 音频编码（`-c:a`）
可直接使用：`aac`、`mp3` / `libmp3lame`、`flac`、`pcm_*`、`alac`、`tta`、`wmav1/2`、`aptx`、`libopus`。
需 `-strict -2`：`opus`、`vorbis`（实验编码器，`ffmpeg -encoders` 标记位带 `X`）。
本构建无 `libvorbis`（`Unknown encoder 'libvorbis'`，退出码 8）；要 Opus 直接用 `libopus`，不必加 `-strict -2`。

### 不在范围内
DRM 加密 / 受版权保护的片源、视频剪辑与特效、多画面合成、纯音频处理、安装或升级 FFmpeg 本身。

## ffprobe 可用性（先探测，别假定）

代理沙箱里的 `PATH` 通常只有宿主注入的 `workspace-tools` 目录加 `/usr/bin:/bin:/usr/sbin:/sbin`，**不包含 `/opt/homebrew/bin`**。`ffmpeg` 与 `ffprobe` 都由宿主以软壳脚本注入（内容形如 `exec '/opt/homebrew/bin/ffprobe' "$@"`），但**注入哪些工具取决于宿主版本**——实测出现过两种状态：

| 宿主注入状态 | `ffmpeg` | `ffprobe` |
|---|---|---|
| 早期 run（2026-10-31 首次 E2E） | ✅ 可用 | ❌ `command not found` |
| 当前 run（2026-10-31 复测） | ✅ 可用 9.0.2 | ✅ 可用 9.0.2 |

所以**第一步永远是探测**，既不要假定存在，也不要假定不存在：

```bash
command -v ffmpeg ffprobe
```

两者都在就直接用裸命令。若 `ffprobe` 缺失，按以下顺序降级：

```bash
# 1) 先试裸命令
command -v ffprobe || echo "not in PATH"

# 2) 从 ffmpeg 的实现路径反推同目录的 ffprobe
FFMPEG_REAL=$(command -v ffmpeg)
# 若是壳脚本，读出它 exec 的真实路径
grep -oE "/[^'\"]*ffmpeg" "$FFMPEG_REAL" 2>/dev/null
FFPROBE="$(dirname /opt/homebrew/bin/ffmpeg)/ffprobe"

# 3) 实在没有，用 ffmpeg 自身替代（能力受限：只能看基本信息，拿不到结构化 JSON）
ffmpeg -hide_banner -i input.mp4 2>&1 | grep -E "Duration|Stream"
```

要严格校验编码/分辨率/帧率，**优先用 ffprobe**；用 `ffmpeg -i` 兜底时不要声称拿到了精确帧率等字段。

## 常见命令

### 基本转换
```bash
# 简单的格式转换
ffmpeg -i input.mp4 output.mkv

# 带质量参数的转换
ffmpeg -i input.mp4 -c:v libx264 -c:a aac -b:v 2M output.mp4
```

### 分辨率调整
```bash
# 调整分辨率
ffmpeg -i input.mp4 -vf "scale=1280:720" output_720p.mp4

# 保持宽高比调整
ffmpeg -i input.mp4 -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" output_even.mp4
```

### 帧率调整
```bash
# 改变帧率（-r 是输出侧选项，放在 -i 之后）
ffmpeg -i input.mp4 -r 30 output_30fps.mp4

# 设置目标帧率
ffmpeg -i input.mp4 -r 60 output_60fps.mp4
```

> **注意**：`-framerate` 是**输入侧**选项，仅对某些输入格式（如 image2 序列、rawvideo）有效；对普通视频文件放在输出侧**不生效**（实测：`-framerate 60` 后输出仍为原帧率）。改帧率请一律使用输出侧的 `-r`。

### 比特率控制
```bash
# 高质量
ffmpeg -i input.mp4 -b:v 8M -maxrate 9M output_high.mp4

# 低质量（移动设备友好）
ffmpeg -i input.mp4 -b:v 500K -maxrate 600K output_mobile.mp4
```

### 音频处理
```bash
# 只转换视频，保持音频
ffmpeg -i input.mp4 -c:v copy output_video_only.mp4

# 转换视频和音频
ffmpeg -i input.mp4 -c:v libx264 -c:a aac output_with_audio.mp4

# 删除音频
ffmpeg -i input.mp4 -c:v libx264 -an output_no_audio.mp4
```

### 元信息提取
```bash
# 打印视频信息
ffprobe input.mp4

# 提取字幕轨道：源文件确定有字幕用 -map 0:s:0；不确定就加 ? 容忍缺失
ffmpeg -i input.mp4 -map 0:s:0  output_with_subtitles.mp4
ffmpeg -i input.mp4 -map 0:s:0? output_with_subtitles.mp4

# 不要写 -map:s:s 0：这不是合法语法，ffmpeg 会静默忽略该选项，
# 结果是"成功"导出了全部流（音视频），而你以为只导了字幕

# 嵌入封面
ffmpeg -i input.mp4 -i cover.jpg -map 0 -map 1 -c:v copy -c:a copy -c:s copy output_with_cover.mp4
```

### 批量处理
```bash
# 使用 for 循环批量转换（输出必须写入新路径，禁止与输入同名，否则会原地覆盖源文件）
for f in *.mp4; do
    ffmpeg -i "$f" "${f%.mp4}_converted.mp4"
done

# 使用 concat demuxer 拼接多个文件（每行须为 `file '/绝对路径'`，且需 -f concat -safe 0）
ffmpeg -f concat -safe 0 -i files.txt -c copy output_concatenated.mp4

# 用 file: 协议前缀指定列表文件（与上一行等价）
ffmpeg -f concat -safe 0 -i file:files.txt -c copy output_concatenated.mp4
```

> **注意**：`-i files.txt` 或 `-i file:files.txt` **必须**搭配 `-f concat -safe 0`，否则会报 `Invalid data found when processing input`。列表文件每行须为 `file '/absolute/path'`。**不要**写成 `-i @files.txt`——FFmpeg 不识别 `@` 前缀，会直接报 `Error opening input file @files.txt`。
>
> **相对路径的坑**：即使加了 `-safe 0`，列表里的相对路径也是**相对列表文件所在目录**解析，而不是相对当前工作目录。实测把 `data-demo/e2e/concat/p1.mp4` 写进同目录的 `files.txt`，会报 `Impossible to open 'data-demo/e2e/concat/data-demo/e2e/concat/p1.mp4'`（路径被拼了两遍）。要么写绝对路径，要么写只相对列表文件目录的裸文件名（`file 'p1.mp4'`）。

## 常见参数说明

### 容器格式
- `-f` 或 `--format`：输出容器格式
  - `mp4`, `mkv`, `avi`, `mov`, `flv`, `webm` 等

### 视频编解码器
- `-c:v` 或 `--video_codec`：视频编解码器
  - `libx264`, `libx265`, `mpeg4`, `vp8`, `vp9`, `av1` 等

### 音频编解码器
- `-c:a` 或 `--audio_codec`：音频编解码器
  - `aac`、`mp3`（或 `libmp3lame`）、`flac`：可直接使用
  - `libopus`：可直接使用（本构建 `--enable-libopus`，实测输出 48kHz 立体声，退出码 0）——**推荐用它替代 `opus`**
  - `opus`、`vorbis`：实验编码器（`ffmpeg -encoders` 中标记位带 `X`），**必须加 `-strict -2`**，否则报 `Experimental feature` 并产出 0 字节文件（实测退出码 88）
  - `libvorbis`：本构建**未编入**（`Unknown encoder 'libvorbis'`，退出码 8），别照抄网络教程

### 视频参数
- `-vf` 或 `--video_filter`：视频滤镜
  - `scale=1280:720`，`crop=width:height` 等
- `-r` 或 `--fps`：帧率
- `-b:v` 或 `--bitrate_video`：视频比特率
- `-maxrate`：最大比特率
- `-bufsize`：缓冲区大小

### 音频参数
- `-ar` 或 `--audio_sample_rate`：音频采样率
- `-b:a` 或 `--bitrate_audio`：音频比特率
- `-ac` 或 `--audio_channels`：音频通道数

### 输出文件
- `-i` 或 `--input`：输入文件
- `{output}`：输出文件路径

## 安全与限制

- **文件路径**：所有文件路径应位于工作区内，避免访问系统根目录
- **参数验证**：在执行前验证参数格式，避免非法参数导致 FFmpeg 错误
- **磁盘空间**：转换过程需要足够的磁盘空间用于临时文件
- **网络访问**：仅在需要下载模板或参考文件时进行网络访问
- **权限**：确保有权限读取输入文件和写入输出文件

## 转换前检查清单（建议逐条过）

执行任何 `-i` 写操作之前，先确认这六条，能避免最常见的几类事故（全部来自实测）：

1. **先确认 `ffprobe` 真的可执行**。宿主环境常只有 `ffmpeg` 在 `PATH` 里（见上文"ffprobe 可用性"），别等到验证阶段才发现命令不存在——那样这一整轮转码就没法判定成败。
2. **输出路径 ≠ 输入路径**。同名会原地覆盖源文件（见上文"批量处理"）。默认用带后缀的新路径，如 `${f%.mp4}_converted.mp4`。
3. **目标编码器在 `ffmpeg -encoders` 里且实测能跑**。`opus`/`vorbis` 需 `-strict -2`；`vp8` 对合成测试源会失败。
4. **输出容器能容纳目标编码**。H.265 一般用 MKV 或 MP4；AV1 常用 MKV/WebM；ProRes 用 MOV。
5. **音频流要显式处理**。只写 `-c:v` 时音频可能被丢弃或重编码出错；不确定就用 `-c:a copy` 保留原音频。
6. **转换后必须用 `ffprobe` 复核**，且不能只看退出码 0。实测有三种"退出码 0 但没成功"或"看着有文件但内容是坏的"：
   - 实验编码器失败（`opus`/`vorbis` 无 `-strict -2`）→ 退出码 88 + 0 字节文件；
   - 输出文件已存在、命令里没写 `-y` → 交互提示被自动拒答，打印 `Not overwriting - exiting`，**退出码仍是 0**，文件根本没更新；
   - 编码/容器不匹配（如 H.265 塞进 WebM）→ 退出码 234，但可能留下一个几百字节的残file，`ls` 看着"有文件"。

   因此判定成功的顺序是：**退出码 → 文件大小 → `ffprobe` 能否解析出预期编码/时长**，三者都过才算成功。

   > **md5 只能证明"没变"，不能证明"转对了"**。详见下节"转换后复核（md5 + ffprobe）"。

## 转换后复核（md5 + ffprobe）

退出码、文件大小、md5、ffprobe 四者各自只能回答一个问题，必须合起来用。实测四种场景的判别表：

| 场景 | 退出码 | 文件在? | 大小 | md5（与转换前比） | ffprobe | 真实结论 |
|---|---|---|---|---|---|---|
| 正常转换 | 0 | 是 | 6147869 | **变了** | 通过 | ✅ 成功 |
| 目标已存在、命令没写 `-y` | **0** | 是 | 6147869（未变） | **没变** | 通过（验的是旧文件） | ❌ **根本没转** |
| 编码/容器不匹配（H.265→WebM） | 234 | 是 | 262 B（残file） | 变了 | **失败，exit 1** | ❌ 失败 |
| 输入文件缺失 | 254 | 否 | — | — | — | ❌ 未产生输出 |

结论：**只有退出码会骗人，只有大小会骗人，只有"文件存在"更会骗人。** md5 专门负责抓"没变"，ffprobe 专门负责抓"坏了"。

### 可复制的复核片段

```bash
#!/bin/sh
# 用法: sh verify.sh <源文件> <输出文件> <期望编码> <期望宽> <期望高>
SRC="$1"; OUT="$2"; WANT_V="$3"; WANT_W="$4"; WANT_H="$5"
FFPROBE="${FFPROBE:-ffprobe}"   # 不在 PATH 时按上文"ffprobe 可用性"兜底

# 复核前先取源文件指纹
SRC_MD5_BEFORE=$(md5 -q "$SRC")

# ---- 1. 大小 ----
[ -f "$OUT" ] || { echo "FAIL 输出不存在"; exit 1; }
SIZE=$(stat -f%z "$OUT" 2>/dev/null || stat -c%s "$OUT")
[ "$SIZE" -gt 1024 ] || { echo "FAIL 输出仅 $SIZE 字节，疑似残file"; exit 1; }

# ---- 2. ffprobe 结构化校验（这一步才是真裁判） ----
"$FFPROBE" -v error -show_entries format=duration -of default=nk=1:nw=1 "$OUT" >/dev/null 2>&1 \
  || { echo "FAIL ffprobe 无法解析，文件损坏"; exit 1; }

GOT_V=$("$FFPROBE" -v error -select_streams v:0 -show_entries stream=codec_name -of default=nk=1:nw=1 "$OUT")
GOT_W=$("$FFPROBE" -v error -select_streams v:0 -show_entries stream=width  -of default=nk=1:nw=1 "$OUT")
GOT_H=$("$FFPROBE" -v error -select_streams v:0 -show_entries stream=height -of default=nk=1:nw=1 "$OUT")
GOT_A=$("$FFPROBE" -v error -select_streams a:0 -show_entries stream=codec_name -of default=nk=1:nw=1 "$OUT")
GOT_D=$("$FFPROBE" -v error -show_entries format=duration -of default=nk=1:nw=1 "$OUT")

[ "$GOT_V" = "$WANT_V" ] || { echo "FAIL 视频编码 $GOT_V != 期望 $WANT_V"; exit 1; }
[ "$GOT_W" = "$WANT_W" ] && [ "$GOT_H" = "$WANT_H" ] || { echo "FAIL 分辨率 ${GOT_W}x${GOT_H} != 期望 ${WANT_W}x${WANT_H}"; exit 1; }
[ -n "$GOT_A" ] || echo "WARN 输出无音频流（若源有音频则属丢轨）"

# ---- 3. md5 双侧复核 ----
SRC_MD5_AFTER=$(md5 -q "$SRC")
[ "$SRC_MD5_BEFORE" = "$SRC_MD5_AFTER" ] || { echo "FAIL 源文件被改动！"; exit 1; }
OUT_MD5=$(md5 -q "$OUT")
[ "$OUT_MD5" != "$SRC_MD5_AFTER" ] || { echo "FAIL 输出与源 md5 相同，说明根本没转换"; exit 1; }

echo "PASS  vcodec=$GOT_V ${GOT_W}x${GOT_H} acodec=${GOT_A:-none} duration=$GOT_D size=$SIZE"
echo "      src_md5=$SRC_MD5_AFTER (未变)  out_md5=$OUT_MD5"
```

### md5 的边界（实测）

md5 是**内容指纹**，不是**质量指标**，用错会得出相反结论：

- ✅ **能证明**：源文件没被覆盖（覆盖保护的核心证据）；输出文件没被本次运行改写（抓"退出码 0 但没转"）；输出 ≠ 源（抓"命令写成了原地复制"）。
- ❌ **不能证明**：转码成功。任何参数变动都会改 md5——实测同一源、同一次编码，仅加 `-tag:v hvc1` 就把 md5 从 `5b74ce1e…` 变成 `fe8bffc2…`（大小都是 5060317 B），但把容器剥掉只比 HEVC 裸流，两者**完全一致**（`1a22c42a…` / 4703704 B）——变的只是 `codec_tag_string`（`hvc1` vs `hev1`），画面数据一个字节没差。
- ❌ **不能证明**：可复现性。本构建 x265/x264 在同机重复跑是逐字节可复现的（x265 连跑 3 轮、x264 连跑 2 轮、全长含音轨连跑 2 轮，md5 各自一致），但这依赖版本、线程数与参数完全相同；换机器/换 FFmpeg 版本就可能整个变。**不要**把 md5 当成跨环境的质量基线。

一句话：**md5 回答"变没变"，ffprobe 回答"对不对"。**

## 示例工作流程

### 1. 检查 FFmpeg 是否可用
```bash
ffmpeg -version
# 检查 ffprobe 是否可用（不在 PATH 时按"ffprobe 可用性"一节的顺序兜底）
command -v ffprobe || /opt/homebrew/bin/ffprobe -version
```

### 2. 检查源视频信息
```bash
ffprobe -v error -show_entries format=duration,size -of default=noprint_wrappers=1:nokey=1 input.mp4
```

### 3. 执行转换
```bash
ffmpeg -i input.mp4 -c:v libx264 -c:a aac -b:v 2M output.mp4
```

### 4. 验证结果
```bash
# 检查输出文件信息（能解析出预期编码/时长才算过）
ffprobe -v error -show_entries format=duration,size -show_entries stream=codec_type,codec_name,width,height -of compact=p=0 output.mp4

# 检查文件大小（0 字节或几百字节基本都是失败残file）
ls -lh output.mp4
```

## 常见问题

### 1. 容器格式不兼容
**问题**：目标容器不支持特定编解码器
**解决方案**：确保编解码器与容器格式兼容，如使用 MP4 时使用 H.264 或 AAC 编解码器

### 2. 内存不足
**问题**：大文件转换时内存不足
**解决方案**：分片处理或使用更低的比特率

### 3. 音频轨道丢失
**问题**：转换后音频轨道丢失
**解决方案**：确保指定正确的 `-c:a` 参数，或使用 `-map 0:a` 保留音频

### 4. 输出文件过大
**问题**：输出文件远大于输入文件
**解决方案**：使用更低的比特率，或使用无损压缩格式

## 快速参考

### 基本转换
```bash
ffmpeg -i input.mp4 -c:v libx264 -c:a aac output.mp4
```

### 质量优先
```bash
ffmpeg -i input.mp4 -c:v libx265 -c:a aac -b:v 5M output.mp4
```

### 移动设备优化
```bash
ffmpeg -i input.mp4 -c:v libx264 -preset fast -b:v 1M output_mobile.mp4
```

### 分辨率调整
```bash
ffmpeg -i input.mp4 -vf "scale=1280:720" output_720p.mp4
```

### 音频处理
```bash
ffmpeg -i input.mp4 -c:v copy -an output_no_audio.mp4
```

### 批量处理
```bash
for f in *.mp4; do ffmpeg -i "$f" "${f%.mp4}_converted.mp4"; done
```

## 工具

### FFmpeg
- **官方网站**：https://ffmpeg.org/
- **文档**：https://ffmpeg.org/documentation.html
- **下载**：https://ffmpeg.org/download.html

### ffprobe
- **用途**：查询视频文件信息
- **常用选项**：`-show_entries format=*`、`-select_streams v:0` 等

### 手册
- **视频转换手册**：`-i input.mp4 -c:v libx264 -c:a aac output.mp4`
- **批量处理手册**：`-i @files.txt -c copy output.mp4`

## 总结

视频格式转换技能提供了完整的 FFmpeg 工具使用指导，包括检查、计划、应用和验证等阶段，支持各种视频格式转换、参数调整和批量处理。通过本技能，代理可以高效地完成视频格式转换任务，同时确保安全性和可靠性。