# Konata-Native

Konata 专用原生多媒体播放内核构建配方、补丁集合与预编译运行时（`libmpv 0.42.0` + `FFmpeg 9.0.2`）。

## 概述

本仓库为 [Konata](https://github.com/ClainSt/Konata) 提供固化版本的跨端音视频解码与渲染原生依赖：

- **Windows x64**（`libmpv-2.dll`、`libmpv.dll.a`、C 头文件）：基于 `mpv 0.42.0`（`-Dgpl=false`）与静态链接 `FFmpeg 9.0.2`（`--enable-version3`），含 Direct3D 11 / `vo=gpu-next` 与 HDR/Dolby Vision HLG 回退补丁，有效协议 **LGPL-3.0-or-later**。
- **Android ARM64**（`libmpv.so`、`libav*.so`、`libsw*.so`、`libmediakitandroidhelper.so`、`libc++_shared.so`）：基于 `mpv-android`（`mpv 0.42.0` + `FFmpeg 9.0.2` `--enable-gpl --enable-version3`），含 EGL HDR 色彩空间选择与 16KB 页对齐，有效协议 **GPL-3.0-or-later**。

## 预编译产物（Release `r1-mpv0.42.0-ff9.0.2`）

预编译包发布于 [GitHub Releases](https://github.com/ClainSt/Konata-Native/releases/tag/r1-mpv0.42.0-ff9.0.2)，由 Konata 主仓执行 `dart run tool/fetch_native.dart` 自动拉取并校验 SHA-256：

| 目标平台 | 归档文件名 | 字节数 | SHA-256 |
| :--- | :--- | ---: | :--- |
| **Windows x64** | `konata-native-windows-x64-r1.zip` | `29016042` | `ccddc4bc2f445bfb0fd8f3f5f9ce881170f3857948daa02b03bd888774b06e96` |
| **Android ARM64** | `konata-native-android-arm64-r1.zip` | `14599121` | `578506f1007050485780f4729bdbc41722d05e86045899585adce43d96a27da2` |

## 源码清单与锁定校验

所有构建脚本在解包前均强制校验源码压缩包的 SHA-256：

- `mpv-v0.42.0.tar.gz`：`https://codeload.github.com/mpv-player/mpv/tar.gz/b0841e8c4d156b8403dc1876c4cbd9011eb583f3`
- `ffmpeg-n9.0.2.tar.gz`：`https://codeload.github.com/FFmpeg/FFmpeg/tar.gz/946fcce07b6dcd0331c8cc609192aeff5e1924f8`
- Windows 交叉编译工具链容器：`ghcr.io/btbn/ffmpeg-builds/win64-lgpl@sha256:d2290020ecfd15e95366c828f236c7e3d8066fd6a92507d93b6b119bc3aed48a`
- Android 构建脚本基线：`mpv-android` commit `255901dd586fa750000fa0075f69d6ffdd26cd9c`（SHA-256: `10750e25fb2f1097f5df55a409df6d962279a0923d7c31ed25701120190176f4`）
- Android Helper JAR：`https://github.com/media-kit/libmpv-android-video-build/releases/download/v1.1.7/default-arm64-v8a.jar`（仅提取 `libmediakitandroidhelper.so`）

## 复现构建步骤（Linux / WSL Ubuntu 24.04）

请将 `KONATA_BUILD_ROOT` 放在 Linux 本地文件系统（如 `/opt/konata-build`），避免放在 NTFS 挂载目录。需准备 Python 3.12、`build-essential`、Git、`curl`、`wget`、`unzip`、`autoconf`、`automake`、`libtool`、`pkg-config`、CMake、Ninja、NASM、`gperf`、`python3-venv` 与 Podman。

```sh
export KONATA_BUILD_ROOT=/opt/konata-build
export KONATA_SOURCES=/path/to/source-archives
export KONATA_NATIVE_ROOT=/path/to/Konata/native/konata-native
export KONATA_JOBS=2

# 1. Windows x64 构建
python3 prepare-windows.py --root "$KONATA_BUILD_ROOT" --sources "$KONATA_SOURCES"
podman run --name konata-windows-build --cpus=2 --memory=4g \
  -e KONATA_JOBS=2 -v "$KONATA_BUILD_ROOT/windows:/mpv" -w /mpv \
  ghcr.io/btbn/ffmpeg-builds/win64-lgpl@sha256:d2290020ecfd15e95366c828f236c7e3d8066fd6a92507d93b6b119bc3aed48a \
  bash ci/build-konata-windows.sh
python3 stage.py windows --root "$KONATA_BUILD_ROOT" --output "$KONATA_NATIVE_ROOT"

# 2. Android ARM64 构建（NDK r30 由 prepare-ndk.sh 校验并解压）
bash prepare-ndk.sh
bash prepare-android.sh
export PATH="$KONATA_BUILD_ROOT/venv/bin:$PATH"
export cores=2
(cd "$KONATA_BUILD_ROOT/mpv-android-255901dd586fa750000fa0075f69d6ffdd26cd9c/buildscripts" && ./buildall.sh --arch arm64 mpv)
python3 stage.py android --root "$KONATA_BUILD_ROOT" --output "$KONATA_NATIVE_ROOT" \
  --android-helper-jar /path/to/default-arm64-v8a.jar
```

## 补丁说明

- `dovi-hlg-fallback.patch`：在 Dolby Vision 映射前保留原始 HDR 元数据，使用 HLG 基础层时还原 HLG 场景光与元数据（双端共用）。
- `android-hdr.patch`：依据还原后的基础层色彩参数自动选择 EGL HDR 输出空间。
- `windows-dovi-base-metadata.patch` / `windows-d3d11-strict-float.patch` / `windows-d3d11-low10-upload.patch` / `windows-p010-repack.patch`：Windows Direct3D 11 10-bit / P010 / 严格浮点渲染管线修复补丁。

## 许可证

- 本仓库构建脚本与补丁遵循 **GNU General Public License v3.0 or later（GPL-3.0-or-later）**，全文见 [`LICENSE`](LICENSE)。
- Windows x64 预编译运行时有效协议为 **LGPL-3.0-or-later**；Android ARM64 预编译运行时有效协议为 **GPL-3.0-or-later**。
