# FFmpeg 常用命令

FFmpeg 是一个功能强大的音视频处理工具集。本文档整理了其在常见场景下的实用命令，旨在作为一份快速参考指南。

## 1. 查看媒体文件信息

使用 `ffprobe` 获取详细的媒体信息，包括格式和流信息。

```bash
ffprobe -v error -show_format -show_streams input.mp4
```

将输出格式化为 JSON，便于程序解析和自动化处理。

```bash
ffprobe -v error -show_format -show_streams -print_format json input.mp4
```

## 2. 视频剪辑

视频剪辑的关键在于 `-ss`（起始时间）参数的位置，它决定了定位方式和精确度。

| 参数顺序           | 定位方式 | 速度   | 定位精度       | 说明                                     |
|--------------------|----------|--------|---------------|------------------------------------------|
| `-ss` 在 `-i` 前   | 输入定位 | **快** | 关键帧级      | 快速裁剪，适用于对起始点要求不高的场景     |
| `-ss` 在 `-i` 后   | 输出定位 | **慢** | 逐帧级        | 精确裁剪，速度较慢，但结果精准             |

### 示例命令

1.  **快速裁剪（输入定位）**

    从第 30 秒附近的关键帧开始，快速截取 10 秒。速度最快，但起始位置可能不精确。
    
    ```bash
    ffmpeg -ss 00:00:30 -i input.mp4 -t 10 -c copy output.mp4
    ```
    
2.  **精确裁剪（输出定位）**

    从第 30 秒精确位置开始，截取 10 秒。会重新编码，速度较慢，但结果精准。
    
    ```bash
    ffmpeg -i input.mp4 -ss 00:00:30 -t 10 -c:v libx264 -c:a copy output.mp4
    ```

## 3. 视频合并

### 无损合并（流拷贝）

将视频流和音频流直接合并到新容器，不重新编码，速度极快。

```bash
ffmpeg -i video.mp4 -i audio.mp3 -c copy -map 0:v:0 -map 1:a:0 output.mp4
```

### 重编码合并

合并时重新编码，可用于调整编码参数或解决兼容性问题。

```bash
ffmpeg -i video.mp4 -i audio.mp3 -c:v libx264 -c:a aac output.mp4
```

### 合并多个视频片段

首先，创建一个文本文件（如 `list.txt`），列出所有要合并的视频文件路径：

```text
# list.txt
file 'video1.mp4'
file 'video2.mp4'
file 'video3.mp4'
```

然后使用 `concat` 分离器进行合并：

```bash
ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4
```

## 4. 视频格式转换

### 转换容器格式（流拷贝）

仅更换文件容器（如 MP4 转 MKV），不改变音视频编码，速度快且无损画质。

```bash
ffmpeg -i input.mp4 -c copy output.mkv
```

### 重编码视频

更改视频编码格式（如转换为 H.264），可用于压缩或兼容特定设备。

```bash
ffmpeg -i input.mp4 -c:v libx264 -c:a copy output.mp4
```

## 5. 视频压缩

视频压缩通常涉及在文件大小和画质之间进行权衡。FFmpeg 提供了多种模式来控制这一过程。

### CRF（恒定质量编码）

推荐模式。通过动态分配比特率来保持视觉质量，数值越低画质越好（0-51，推荐 18-28）。

```bash
ffmpeg -i input.mp4 -c:v libx264 -preset medium -crf 23 -c:a copy output.mp4
```

*   `-preset`: 编码速度与压缩率的平衡（`ultrafast`, `fast`, `medium`, `slow`, `veryslow`）。越慢，压缩率越高，文件越小。

### CBR（恒定比特率）

适用于需要固定带宽的场景（如直播推流），通过强制限定比特率保持数据流稳定。

```bash
ffmpeg -i input.mp4 -c:v libx264 -b:v 4000k -maxrate 4000k -minrate 4000k -bufsize 4000k -c:a copy output.mp4
```

*   `-b:v`: 目标视频比特率。
*   `-maxrate` / `-minrate`: 最大/最小比特率。
*   `-bufsize`: 缓冲区大小，通常设为比特率的 1-2 倍。

## 6. 视频帧率调整

### 基础帧率转换

通过直接丢帧或复制帧来改变帧率，**可能导致画面卡顿或动作不连贯**，适用于对流畅度要求不高的场景。

```bash
ffmpeg -i input.mp4 -r 30 output.mp4
```

### 高质量帧率转换（重编码）

通过重新编码来生成新的帧，能获得更平滑的过渡效果，但速度较慢。

```bash
ffmpeg -i input.mp4 -r 30 -c:v libx264 -preset slow -crf 18 -c:a copy output.mp4
```

*   `-preset slow`: 使用较慢的编码预设，以获得更好的压缩效率和质量。

## 7. 提取音频

### 提取音频（流拷贝）

直接复制原始音频流，速度快且无损。

```bash
ffmpeg -i input.mp4 -vn -c:a copy output.aac
```

*   `-vn`: 禁用视频流。

### 提取音频并转换格式

如果原始音频格式不是目标格式（例如，从 MP4 提取为 MP3），则需要重编码音频流。

```bash
ffmpeg -i input.mp4 -vn -c:a mp3 -q:a 2 output.mp3
```

*   `-q:a`: 设置音频质量（适用于 MP3 等有损格式，0-9，值越小质量越高）。

## 8. 使用 NVIDIA GPU 加速 (NVENC)

利用 NVIDIA GPU 的 NVENC 硬件编码器，可以大幅提升编码速度，尤其适合 4K 或高帧率视频的处理。以下是一个兼顾质量和速度的 HEVC (H.265) 编码示例：

```bash
ffmpeg -i input.mp4 -c:v hevc_nvenc -preset p7 -rc vbr -cq 23 -b:v 5M -maxrate 6M -bufsize 12M -c:a copy output.mp4
```

*   `-c:v hevc_nvenc`: 使用 HEVC (H.265) NVENC 编码器。
*   `-preset p7`: 编码预设（p1 最快，p7 最慢最优质）。
*   `-rc vbr`: 可变比特率模式。
*   `-cq`: 恒定质量参数（类似 CRF，值越小质量越高）。
*   `-b:v`, `-maxrate`, `-bufsize`: 配合 VBR 模式使用的比特率控制参数。