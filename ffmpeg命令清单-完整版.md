# SPK App FFmpeg 命令完整清单

本文档收集了 SPK App 项目中所有使用的 FFmpeg 命令和功能，用于自定义构建最小化的 FFmpeg。

---

## 一、编解码器使用情况

### 视频编码器
- **mpeg4**: 当前所有视频输出使用此编码器（内置）
  - 参数：`-c:v mpeg4 -q:v 3`
  - 使用场景：画中画、加水印、去字幕、旋转、智能打包、文章导入、字幕烧录
  - 优点：内置、无需外部库、LGPL 许可
  - 缺点：压缩率较低、文件体积较大

- **libx264**: H.264 编码器（需要外部库）
  - 参数：`-c:v libx264 -preset medium -crf 23`
  - 使用场景：所有视频输出（可替换 mpeg4）
  - 优点：压缩率高、质量好、兼容性强
  - 缺点：需要 libx264 库、GPL 许可、增加约 2-3 MB 体积

### 音频编码器
- **aac**: 音频合成和重编码
  - 参数：`-c:a aac -b:a 128k`
  - 使用场景：音频合成、背景音乐混合
- **copy**: 音频流复制（不重编码）
  - 参数：`-c:a copy`
  - 使用场景：大部分视频处理保留原音频

### 复制模式
- **-c copy**: 视频和音频流复制
  - 使用场景：视频裁剪、音频复制、背景音乐添加

---

## 二、视频滤镜使用情况

### 基础滤镜（内置）

#### 1. scale - 视频缩放
**使用频率**: 极高（几乎所有功能）
**命令示例**:
```bash
# 基础缩放
scale=1280:720

# 保持宽高比，限制最大高度
scale=-2:min(720\,ih)

# 缩放到偶数尺寸（编码器要求）
scale=trunc(iw/2)*2:trunc(ih/2)*2

# 画中画缩放
scale=640:360
```
**使用场景**:
- 画中画：缩放小窗口视频
- 去字幕：抽帧时限制高度
- 智能打包：标准化尺寸
- 文章导入：素材标准化、背景模糊
- 所有视频输出：确保偶数尺寸

#### 2. overlay - 视频叠加
**使用频率**: 高
**命令示例**:
```bash
# 固定位置叠加
overlay=x=10:y=10

# 动态位置（画中画）
overlay=x=100:y=100:shortest=1

# 居中叠加
overlay=(W-w)/2:(H-h)/2

# 右下角叠加
overlay=W-w-24:H-h-24

# 动态移动水印
overlay=x='mod(t*55\,W-w)':y='mod(t*35\,H-h)'
```
**使用场景**:
- 画中画：叠加小窗口视频
- 加水印：叠加水印图片/文字
- 智能打包：动态水印、自动画中画
- 文章导入：叠加水印图片

#### 3. format - 像素格式转换
**使用频率**: 极高
**命令示例**:
```bash
# 转换为 YUV420P（移动端兼容）
format=yuv420p

# 转换为 RGBA（支持透明度）
format=rgba

# 自动格式
format=auto
```
**使用场景**:
- 所有视频输出：确保 yuv420p 格式兼容性
- 水印处理：rgba 格式支持透明度

#### 4. colorchannelmixer - 颜色通道混合
**使用频率**: 中
**命令示例**:
```bash
# 调整透明度
colorchannelmixer=aa=0.5

# 调整透明度（水印）
colorchannelmixer=aa=0.06
```
**使用场景**:
- 画中画：调整小窗口透明度
- 加水印：调整水印透明度
- 智能打包：自动画中画透明度

#### 5. delogo - 去除 logo/字幕
**使用频率**: 中
**命令示例**:
```bash
# 去除指定区域
delogo=x=100:y=500:w=800:h=100

# 多个区域（链式）
delogo=x=10:y=10:w=100:h=50,delogo=x=200:y=200:w=150:h=60
```
**使用场景**:
- 去字幕：擦除字幕区域

#### 6. fps - 帧率调整
**使用频率**: 中
**命令示例**:
```bash
# 固定帧率
fps=30

# 动态帧率（抽帧）
fps=1

# 根据时长计算帧率
fps=0.5
```
**使用场景**:
- 去字幕：抽帧生成预览图
- 文章导入：素材标准化为 30fps

#### 7. transpose - 视频旋转
**使用频率**: 中
**命令示例**:
```bash
# 顺时针旋转 90 度
transpose=1

# 逆时针旋转 90 度
transpose=2

# 旋转 180 度
transpose=1,transpose=1
```
**使用场景**:
- 视频旋转：90°/180°/270° 旋转

#### 8. hflip / vflip - 视频翻转
**使用频率**: 低
**命令示例**:
```bash
# 水平翻转
hflip

# 垂直翻转
vflip
```
**使用场景**:
- 视频翻转：水平/垂直镜像

#### 9. drawtext - 文字水印
**使用频率**: 低
**命令示例**:
```bash
# 基础文字水印
drawtext=text='水印文字':fontsize=24:fontcolor=white:x=10:y=10

# 右下角文字水印
drawtext=text='水印':fontsize=24:fontcolor=white:x=w-text_w-10:y=h-text_h-10
```
**使用场景**:
- 水印处理器：添加文字水印

#### 10. boxblur - 高斯模糊
**使用频率**: 中
**命令示例**:
```bash
# 轻度模糊
boxblur=luma_radius=1:luma_power=1

# 重度模糊（背景）
boxblur=18:2
```
**使用场景**:
- 智能打包：视频模糊效果
- 文章导入：背景模糊效果

#### 11. eq - 色彩调整
**使用频率**: 低
**命令示例**:
```bash
# 调整亮度、对比度、饱和度
eq=brightness=0.03:contrast=1.05:saturation=1.06
```
**使用场景**:
- 智能打包：色彩微调

#### 12. rotate - 旋转（角度）
**使用频率**: 低
**命令示例**:
```bash
# 轻微旋转
rotate=0.015:c=black
```
**使用场景**:
- 智能打包：轻微旋转效果

#### 13. crop - 视频裁剪
**使用频率**: 低
**命令示例**:
```bash
# 裁剪到指定尺寸
crop=1920:1080
```
**使用场景**:
- 文章导入：裁剪视频到目标尺寸

#### 14. split - 视频分流
**使用频率**: 中
**命令示例**:
```bash
# 分成两路
[0:v]split=2[main][pip]

# 分成两路用于背景模糊
[0:v]split=2[bg][fg]
```
**使用场景**:
- 智能打包：自动画中画
- 文章导入：背景模糊效果

#### 15. color - 生成纯色
**使用频率**: 低
**命令示例**:
```bash
# 生成半透明白色
color=c=white@0.12:s=80x36
```
**使用场景**:
- 智能打包：动态水印背景

### 音频滤镜（内置）

#### 16. concat - 音频连接
**使用频率**: 中
**命令示例**:
```bash
# 连接多个音频
[0:a][1:a][2:a]concat=n=3:v=0:a=1[outa]
```
**使用场景**:
- 文章导入：合并多段配音

#### 17. amix - 音频混合
**使用频率**: 中
**命令示例**:
```bash
# 混合两路音频（人声+背景音乐）
[0:a][bgm]amix=inputs=2:duration=first:dropout_transition=0[aout]
```
**使用场景**:
- 文章导入：添加背景音乐

### 字幕滤镜（需要 libass）

#### 18. subtitles - 字幕烧录
**使用频率**: 中
**命令示例**:
```bash
# 烧录 ASS 字幕
subtitles=/path/to/subtitle.ass:charenc=UTF-8

# 烧录 SRT 字幕
subtitles=/path/to/subtitle.srt
```
**使用场景**:
- 文章导入：字幕烧录

**重要说明**: 
- 此滤镜依赖 **libass** 库
- libass 依赖链会自动包含 libfreetype、libfribidi、libharfbuzz
- 构建时必须添加 `--enable-libass` 参数

---

## 三、其他 FFmpeg 功能

### 媒体信息探测
- **FFprobeKit**: 获取视频信息（宽高、时长、旋转角度等）
- 使用场景：去字幕、所有视频处理前的信息获取

### 流控制参数
- **-stream_loop -1**: 循环输入流（画中画、文章导入、背景音乐）
- **-shortest**: 以最短流为准（画中画、文章导入）
- **-map**: 流映射（所有多输入场景）

### 输出优化
- **-movflags +faststart**: MP4 快速启动（所有视频输出）
- **-threads 4**: 多线程处理（去字幕）
- **-pix_fmt yuv420p**: 像素格式（所有视频输出）

### 时间控制
- **-ss**: 起始时间（视频裁剪）
- **-to**: 结束时间（视频裁剪）

---

## 四、完整命令示例

### 1. 画中画
```bash
-i "main.mp4" -stream_loop -1 -i "pip.mp4" \
-filter_complex "[1:v]scale=640:360[overlay];[0:v][overlay]overlay=x=100:y=100:shortest=1,format=yuv420p[vout]" \
-map "[vout]" -map 0:a? \
-c:v mpeg4 -q:v 3 -pix_fmt yuv420p \
-c:a copy -shortest -movflags +faststart output.mp4
```

### 2. 视频加水印（动态）
```bash
-y -i "video.mp4" -i "watermark.png" \
-filter_complex "[1:v]scale=120:54,format=rgba,colorchannelmixer=aa=0.8[wm];[0:v][wm]overlay=x='(W-w)/2+sin(t*1.37)*(W-w)*0.28':y='(H-h)/2+cos(t*1.11)*(H-h)*0.28':format=auto,format=yuv420p[vout]" \
-map "[vout]" -map 0:a? \
-c:v mpeg4 -q:v 3 -pix_fmt yuv420p -c:a copy -movflags +faststart output.mp4
```

### 3. 视频去字幕
```bash
-y -i "video.mp4" \
-vf "delogo=x=100:y=500:w=800:h=100,scale=trunc(iw/2)*2:trunc(ih/2)*2,format=yuv420p" \
-c:v mpeg4 -q:v 3 -c:a copy \
-threads 4 -movflags +faststart output.mp4
```

### 4. 视频裁剪
```bash
-i "video.mp4" -ss 00:00:10 -to 00:01:30 -c copy output.mp4
```

### 5. 视频旋转 90 度
```bash
-i "video.mp4" -vf "transpose=1,format=yuv420p" -c:v mpeg4 -q:v 3 -c:a copy output.mp4
```

### 6. 智能打包（色彩+模糊+动态水印）
```bash
-y -i "video.mp4" \
-filter_complex "[0:v]eq=brightness=0.03:contrast=1.05:saturation=1.06,rotate=0.015:c=black,boxblur=luma_radius=1:luma_power=1,scale=trunc(iw/2)*2:trunc(ih/2)*2,format=yuv420p[base0];color=c=white@0.12:s=80x36[water];[base0][water]overlay=x='mod(t*45\,W-w)':y='mod(t*28\,H-h)'[outv]" \
-map "[outv]" -map 0:a? -c:v mpeg4 -q:v 3 -c:a copy output.mp4
```

### 7. 字幕烧录
```bash
-y -i "video.mp4" -vf "subtitles=/path/to/subtitle.ass:charenc=UTF-8" \
-c:v mpeg4 -q:v 3 -c:a copy -movflags +faststart output.mp4
```

### 8. 音频合成（多段配音）
```bash
-y -i "audio1.m4a" -i "audio2.m4a" -i "audio3.m4a" \
-filter_complex "[0:a][1:a][2:a]concat=n=3:v=0:a=1[outa]" \
-map "[outa]" -c:a aac -b:a 128k output.m4a
```

### 9. 添加背景音乐
```bash
-y -i "video.mp4" -stream_loop -1 -i "bgm.mp3" \
-filter_complex "[1:a]volume=0.3[bgm];[0:a][bgm]amix=inputs=2:duration=first:dropout_transition=0[aout]" \
-map 0:v -map "[aout]" -c:v copy -c:a aac -b:a 128k -shortest output.mp4
```

### 10. H.264 编码示例

#### 画中画（H.264）
```bash
-i "main.mp4" -stream_loop -1 -i "pip.mp4" \
-filter_complex "[1:v]scale=640:360[overlay];[0:v][overlay]overlay=x=100:y=100:shortest=1,format=yuv420p[vout]" \
-map "[vout]" -map 0:a? \
-c:v libx264 -preset medium -crf 23 -pix_fmt yuv420p \
-c:a copy -shortest -movflags +faststart output.mp4
```

#### 视频加水印（H.264）
```bash
-y -i "video.mp4" -i "watermark.png" \
-filter_complex "[1:v]scale=120:54,format=rgba,colorchannelmixer=aa=0.8[wm];[0:v][wm]overlay=x=W-w-24:y=H-h-24:format=auto,format=yuv420p[vout]" \
-map "[vout]" -map 0:a? \
-c:v libx264 -preset medium -crf 23 -pix_fmt yuv420p -c:a copy -movflags +faststart output.mp4
```

#### 字幕烧录（H.264）
```bash
-y -i "video.mp4" -vf "subtitles=/path/to/subtitle.ass:charenc=UTF-8" \
-c:v libx264 -preset medium -crf 23 -c:a copy -movflags +faststart output.mp4
```

**H.264 参数说明**:
- `-preset medium`: 编码速度（ultrafast/fast/medium/slow/veryslow）
- `-crf 23`: 质量控制（18-28，越小质量越好，23 为推荐值）

---

## 五、构建建议

### 最小化构建配置

基于以上分析，SPK App 需要以下 FFmpeg 组件：

#### 方案一：仅 mpeg4（最小体积，LGPL 许可）
```bash
--enable-base          # 包含所有内置编解码器和滤镜
--enable-libass        # 字幕烧录（自动包含 freetype/fribidi/harfbuzz）
--enable-small         # 排除非必要库，进一步缩小体积
```
- **体积**: 最小
- **许可**: LGPL（可商用）
- **编码器**: mpeg4（内置）

#### 方案二：mpeg4 + H.264（推荐，GPL 许可）
```bash
--enable-base          # 包含所有内置编解码器和滤镜
--enable-libass        # 字幕烧录
--enable-gpl           # 启用 GPL 许可
--enable-libx264       # H.264 编码器
--enable-small         # 排除非必要库
```
- **体积**: 增加约 2-3 MB
- **许可**: GPL（商用需注意）
- **编码器**: mpeg4（内置）+ H.264（libx264）
- **优势**: 更好的压缩率和视频质量

#### 不需要的外部库（可节省大量空间）
- ⚠️ libx264 (H.264 编码器) - **如需更好压缩率则保留**
- ❌ libx265 (H.265 编码器)
- ❌ libvpx (VP8/VP9 编码器)
- ❌ libaom (AV1 编码器)
- ❌ libopus (Opus 音频编码器)
- ❌ libvorbis (Vorbis 音频编码器)
- ❌ libmp3lame (MP3 编码器)

#### 完整构建命令

**方案一：仅 mpeg4（LGPL）**
```bash
# arm64
sudo ./runner.sh \
  --host=android \
  --arch=arm64 \
  --enable-base \
  --enable-libass \
  --enable-small \
  -y

# armv7
sudo ./runner.sh \
  --host=android \
  --arch=armv7 \
  --enable-base \
  --enable-libass \
  --enable-small \
  -y
```

**方案二：mpeg4 + H.264（GPL，推荐）**
```bash
# arm64
sudo ./runner.sh \
  --host=android \
  --arch=arm64 \
  --enable-base \
  --enable-libass \
  --enable-gpl \
  --enable-libx264 \
  --enable-small \
  -y

# armv7
sudo ./runner.sh \
  --host=android \
  --arch=armv7 \
  --enable-base \
  --enable-libass \
  --enable-gpl \
  --enable-libx264 \
  --enable-small \
  -y
```

### 预期效果

**方案一（仅 mpeg4）**:
- **体积减少**: 约 60-70%（相比完整构建）
- **功能完整**: 覆盖项目所有视频处理需求
- **兼容性**: 保持 LGPL 许可，可商用
- **视频质量**: 中等（mpeg4 压缩率较低）

**方案二（mpeg4 + H.264）**:
- **体积减少**: 约 55-65%（相比完整构建，比方案一多 2-3 MB）
- **功能完整**: 覆盖项目所有视频处理需求
- **兼容性**: GPL 许可，商用需注意
- **视频质量**: 优秀（H.264 压缩率高、质量好）
- **文件体积**: 输出视频体积更小（相同质量下）

---

## 六、功能覆盖清单

### ✅ 已覆盖的功能
- [x] 视频编码输出 (mpeg4)
- [x] 音频编码/提取 (aac)
- [x] 视频裁剪 (-c copy)
- [x] 视频缩放 (scale)
- [x] 视频叠加 (overlay)
- [x] 画中画 (overlay + scale)
- [x] 视频旋转 (transpose)
- [x] 视频翻转 (hflip/vflip)
- [x] 视频模糊 (boxblur)
- [x] 去除字幕/水印 (delogo)
- [x] 添加图片水印 (overlay)
- [x] 添加文字水印 (drawtext)
- [x] 字幕烧录 (subtitles + libass)
- [x] 色彩调整 (eq)
- [x] 帧率调整 (fps)
- [x] 格式转换 (format)
- [x] 透明度调整 (colorchannelmixer)
- [x] 音频合成 (concat)
- [x] 音频混合 (amix)
- [x] 媒体信息探测 (FFprobeKit)

### 📊 使用频率统计
- **极高频**: scale, format, mpeg4, aac
- **高频**: overlay, colorchannelmixer
- **中频**: delogo, fps, transpose, boxblur, split, concat, amix, subtitles
- **低频**: hflip, vflip, drawtext, eq, rotate, crop, color

---

## 七、代码文件映射

### 工具功能 → 代码文件
| 功能 | 文件路径 | 主要命令 |
|------|---------|---------|
| 画中画 | `lib/pages/tools/picture_in_picture/picture_in_picture_page.dart` | overlay, scale, colorchannelmixer |
| 视频加水印 | `lib/pages/tools/video_add_watermark/video_add_watermark_page.dart` | overlay, scale, colorchannelmixer |
| 视频去字幕 | `lib/pages/tools/video_remove_subtitle/video_remove_subtitle_processor.dart` | delogo, scale, fps |
| 视频裁剪 | `lib/pages/video_edit/video_trim_processor.dart` | -c copy |
| 视频旋转 | `lib/pages/video_edit/video_rotate_processor.dart` | transpose, hflip, vflip |
| 水印处理器 | `lib/pages/video_edit/video_watermark_processor.dart` | overlay, drawtext |
| 智能打包 | `lib/pages/tools/smart_packaging/smart_package_processor.dart` | eq, rotate, boxblur, color, split |
| 文章导入 | `lib/pages/tools/import_article/import_article_processor.dart` | scale, fps, overlay, crop, split, concat, amix |
| 字幕烧录 | `lib/pages/tools/import_article/import_article_subtitle_edit_page.dart` | subtitles (libass) |

---

## 八、总结

### 核心结论
SPK App 的所有视频处理功能**只需要 FFmpeg 内置能力 + libass**（+ 可选的 libx264）。

### 构建配置选择

**方案一：最小体积（LGPL）**
```bash
--enable-base --enable-libass --enable-small
```
- 适合：对 APK 体积敏感、需要 LGPL 许可的项目
- 编码器：mpeg4（内置）

**方案二：推荐配置（GPL）**
```bash
--enable-base --enable-libass --enable-gpl --enable-libx264 --enable-small
```
- 适合：追求视频质量和压缩率的项目
- 编码器：mpeg4（内置）+ H.264（libx264）
- 增加体积：约 2-3 MB
- 许可证：GPL（商用需注意）

### 体积优化
- 去除 libx265/libvpx/libaom 等大型编解码库
- 方案一：预计减少 APK 体积 **60-70%**
- 方案二：预计减少 APK 体积 **55-65%**

### 功能完整性
- ✅ 所有 18 个滤镜均为内置
- ✅ mpeg4 和 aac 编码器均为内置
- ✅ 仅 subtitles 滤镜需要 libass（已包含）
- ⚠️ libx264 为可选外部库（推荐添加）

### 代码修改建议
如果选择方案二（H.264），建议将代码中的所有 `mpeg4` 替换为 `libx264`：
```dart
// 修改前
'-c:v mpeg4 -q:v 3'

// 修改后
'-c:v libx264 -preset medium -crf 23'
```

### 下一步
参考 `FFMPEG_CUSTOM_BUILD.md` 文档执行构建流程。

