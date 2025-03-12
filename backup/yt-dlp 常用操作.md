# yt-dlp 常用操作

## 更新
```bash
# 更新到最新版本
yt-dlp -U
```

## 基础使用
```bash
# 下载视频（自动选择最佳质量）
yt-dlp <URL>

# 列出所有可用格式
yt-dlp -F <URL>
```

## 常用格式选择器

- `best`/`b`: 选择同时包含视频和音频的最佳质量格式
- `bestvideo`/`bv`: 选择最佳质量的纯视频格式
- `bestaudio`/`ba`: 选择最佳质量的纯音频格式
- `bestvideo*`/`bv*`: 选择包含视频的最佳质量格式（可能也包含音频）

## 格式选择技巧

### 合并音视频流
```bash
# 下载并合并最佳视频和音频（默认行为）
yt-dlp -f "bv*+ba" <URL>

# 手动指定格式合并（格式号来自-F列表）
yt-dlp -f 137+140 <URL>
```

### 比较运算符
- 数值: `<`, `<=`, `>`, `>=`, `=`, `!=`
- 字符串: `=`(等于), `^=`(开头是), `$=`(结尾是), `*=`(包含), `~=`(正则匹配)
- 前缀`!`表示否定

### 过滤条件
```bash
# 下载指定编码的视频（例如H.265优先，其次H.264）
yt-dlp -f "bv*[vcodec^=hev1]+ba/bv*[vcodec^=avc1]+ba" <URL>

# 下载720p及以下的最佳视频
yt-dlp -f "bv*[height<=720]+ba/b[height<=720]" <URL>

# 下载小于500MB的MP4文件
yt-dlp -f "b[ext=mp4][filesize<500M]" <URL>

# 下载最佳音频（m4a格式优先）
yt-dlp -f "ba[ext=m4a]/ba" <URL>
```

## 特殊场景
```bash
# 需要cookie的网站
yt-dlp --cookies <FILE> <URL>

# 使用浏览器的cookie（以火狐为例）
yt-dlp --cookies-from-browser firefox <URL>
```