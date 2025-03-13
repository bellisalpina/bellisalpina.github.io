# FFmpeg 常用命令参考

FFmpeg 是一个强大的音视频处理工具。本文档提供一些常见使用场景的简单命令参考。

## 查看媒体文件信息

使用 `ffprobe` 获取流信息：

```bash
ffprobe -v error -show_format -show_streams input.mp4
```

以 JSON 格式输出信息：

```bash
ffprobe -v error -show_format -show_streams -print_format json input.mp4
```

## 视频剪辑

| 参数顺序           | 定位方式 | 速度   | 精确度       | 说明                                     |
|--------------------|----------|--------|--------------|------------------------------------------|
| `-ss` 在 `-i` 前   | 输入定位 | **快** | 关键帧级精度 | 快速裁剪，适用于不需要精确帧的场景       |
| `-ss` 在 `-i` 后   | 输出定位 | **慢** | 逐帧级精度   | 精确到帧的裁剪或处理，速度较慢             |

#### 示例命令

1. **输入定位（快速裁剪）**

   ```bash
   ffmpeg -ss 00:00:30 -i input.mp4 -t 10 -c copy output.mp4
   ```

   *从视频大约 30 秒处开始，快速截取 10 秒内容（可能存在几秒的偏差）。*

   ```bash
   ffmpeg -ss 00:01:00 -i input.mp4 -to 00:01:30 -c copy output.mp4
   ```

   *截取视频从 1 分 00 秒到 1 分 30 秒的片段（可能存在几秒的偏差）。*

2. **输出定位（精确裁剪）**

   ```bash
   ffmpeg -i input.mp4 -ss 00:00:30 -t 10 -c copy output.mp4
   ```

   *精确截取从 30 秒处开始的 10 秒内容。*

   ```bash
   ffmpeg -i input.mp4 -ss 00:01:00 -to 00:01:30 -c copy output.mp4
   ```

   *精确截取视频从 1 分 00 秒到 1 分 30 秒的片段。*

## 视频合并

合并视频与音频（不重新编码，速度快）：

```bash
ffmpeg -i video.mp4 -i audio.mp3 -c copy -map 0:v:0 -map 1:a:0 output.mp4
```

合并视频与音频（重新编码，可调整编码参数）：

```bash
ffmpeg -i video.mp4 -i audio.mp3 -c:v libx264 -c:a aac output.mp4
```

合并多个视频文件（需要先创建一个文本文件列表 `list.txt`）：

```bash
# list.txt 示例
# file 'video1.mp4'
# file 'video2.mp4'
# file 'video3.mp4'

ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4
```

## 视频格式转换

更换容器格式（不改变编码，速度快）：

```bash
ffmpeg -i input.mp4 -c copy output.mkv
```

更改视频编码：

```bash
ffmpeg -i input.mp4 -c:v libx264 -c:a copy output.mp4
```

## 视频压缩

### CRF（恒定质量编码）

使用 CRF（Constant Rate Factor）方式压缩视频，数值范围 0-51（值越大压缩率越高，画质越低，18-28 是推荐范围）。该模式通过动态分配比特率实现质量优先的压缩：

```bash
ffmpeg -i input.mp4 -c:v libx264 -crf 23 -preset medium -c:a copy output.mp4
```

*   `-preset`：编码预设，影响编码速度和质量。`medium` 是一个平衡的选择。

### CBR（恒定比特率）

适用于需要固定带宽的场景（如直播推流），通过强制限定比特率保持数据流稳定。需配合 `bufsize` 参数控制缓冲区大小：

```bash
ffmpeg -i input.mp4 -c:v libx264 -b:v 4000k -maxrate 4000k -minrate 4000k -bufsize 2000k -c:a copy output.mp4
```

*   `-b:v`：视频比特率。
*   `-maxrate`：最大比特率。
*   `-minrate`：最小比特率。
*   `-bufsize`：缓冲区大小。

## 视频帧率调整

### 简单更改帧率

```bash
ffmpeg -i input.mp4 -r 30 output.mp4
```

这会将视频帧率设置为 30fps，通过丢弃或重复帧来实现。**注意：可能导致视频播放不流畅。**

### 保持质量的帧率转换

```bash
ffmpeg -i input.mp4 -r 30 -c:v libx264 -preset slow -crf 18 -c:a copy output.mp4
```

这种方式可以获得较好的质量，使用 `libx264` 编码器，质量参数 `crf` 值越小质量越高（18-23 通常是比较好的选择）。

*   `-preset slow`：使用较慢的编码预设，以获得更好的质量。

## 提取音频

从视频中提取音频：

```bash
ffmpeg -i input.mp4 -vn -c:a copy output.mp3
```

*   `-vn`：禁用视频流。

如需不同格式：

```bash
ffmpeg -i input.mp4 -vn -c:a aac output.aac
```

## 使用硬件加速

使用 NVIDIA GPU 加速（NVENC）：

```bash
ffmpeg -i input.mp4 -c:v h264_nvenc -c:a copy output.mp4
```

使用 Intel Quick Sync：

```bash
ffmpeg -i input.mp4 -c:v h264_qsv -c:a copy output.mp4
```

使用 AMD GPU 加速：

```bash
ffmpeg -i input.mp4 -c:v h264_amf -c:a copy output.mp4
```

**注意：** 硬件加速需要相应的硬件和驱动支持。