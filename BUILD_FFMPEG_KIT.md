# FFmpegKit Android/iOS 自定义构建步骤

官方仓库：<https://github.com/arthenica/ffmpeg-kit>

注意：`ffmpeg-kit` 官方已 retired，但源码里的 `android.sh` / `ios.sh` 构建脚本仍可用。

## 1. 进入源码目录

如果已经按当前项目结构拉好了源码：

```bash
cd /Users/luis/Desktop/build-ffmpeg/ffmpeg-kit
```

如果还没有源码：

```bash
cd /Users/luis/Desktop/build-ffmpeg
git clone https://github.com/arthenica/ffmpeg-kit.git ffmpeg-kit
cd ffmpeg-kit
```

## 2. 安装依赖

```bash
brew install automake libtool nasm yasm cmake meson ninja ragel gperf texinfo bison autogen wget doxygen groff gtk-doc libtasn1
```

安装 ffmpeg-kit 模板使用的 Android NDK 版本：

```bash
/Users/luis/Library/Android/sdk/cmdline-tools/latest/bin/sdkmanager --install "ndk;22.1.7171670"
```

## 3. Android 构建

设置环境变量：

```bash
export ANDROID_SDK_ROOT=/Users/luis/Library/Android/sdk
export ANDROID_HOME=/Users/luis/Library/Android/sdk
export ANDROID_NDK_ROOT=/Users/luis/Library/Android/sdk/ndk/22.1.7171670
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
```

SPK App 定制包：只构建 `arm64-v8a`，启用字幕烧录、MP3、H.264：

```bash
./android.sh \
  --disable-arm-v7a-neon \
  --disable-x86 \
  --disable-x86-64 \
  --enable-gpl \
  --enable-x264 \
  --enable-lame \
  --enable-libass
```

说明：`--enable-libass` 会自动拉起 `freetype`、`fontconfig`、`fribidi`、`harfbuzz` 等字幕/字体依赖。`ffmpeg-kit` 的 Android 构建脚本默认会给 FFmpeg configure 添加 `--enable-small`；不要把 `--enable-small` 直接传给 `android.sh`，否则会被当成外部库名解析。

Android 产物：

```bash
prebuilt/bundle-android-aar/ffmpeg-kit.aar
```

## 4. iOS 构建

构建 `arm64` 真机 + `arm64-simulator`，输出 xcframework：

```bash
./ios.sh \
  --xcframework \
  --disable-armv7 \
  --disable-armv7s \
  --disable-i386 \
  --disable-x86-64 \
  --disable-arm64e \
  --disable-arm64-mac-catalyst \
  --disable-x86-64-mac-catalyst
```

iOS 产物：

```bash
prebuilt/bundle-apple-xcframework-ios/
```

## 5. 常用自定义库参数

HTTPS/OpenSSL：

```bash
--enable-openssl
```

音频常用：

```bash
--enable-opus --enable-lame
```

ASS 字幕：

```bash
--enable-libass --enable-fontconfig --enable-freetype --enable-fribidi
```

x264 / x265：

```bash
--enable-gpl --enable-x264 --enable-x265
```

注意：启用 `x264` / `x265` / `libvidstab` / `rubberband` / `xvidcore` 这类 GPL 库后，产物会变成 GPLv3。

## 6. 示例：Android arm64 + HTTPS + opus

```bash
export ANDROID_SDK_ROOT=/Users/luis/Library/Android/sdk
export ANDROID_NDK_ROOT=/Users/luis/Library/Android/sdk/ndk/22.1.7171670

./android.sh \
  --disable-arm-v7a \
  --disable-arm-v7a-neon \
  --disable-x86 \
  --disable-x86-64 \
  --enable-openssl \
  --enable-opus
```

## 7. 查看日志

构建过程大部分详细输出会写入：

```bash
build.log
```

实时查看：

```bash
tail -f build.log
```
