# Konata-Native

Konata 专用原生多媒体播放内核构建配方与预编译运行时（libmpv 0.42.0 + FFmpeg 9.0.2）。

## 概述

本项目为 [Konata](https://github.com/ClainSt/Konata) 提供固化版本的跨端音视频解码与渲染原生支持，包含：
- **Windows x64**：libmpv-2.dll（基于 Direct3D 11 / o=gpu-next，含 HDR/HLG 色彩还原补丁，LGPL-3.0-or-later）；
- **Android ARM64**：libmpv.so 与 FFmpeg 9.0.2 依赖动态库组（支持 Underlay SurfaceView 与 EGL HDR，GPL-3.0-or-later）。

## 预编译产物与 SHA-256

预编译发布包见 [GitHub Releases](https://github.com/ClainSt/Konata-Native/releases)：

| 目标平台 | 发布包文件名 | 校验哈希（SHA-256） |
| :--- | :--- | :--- |
| **Windows x64** | konata-native-windows-x64-r1.zip | 4fc724afb12dbde3463de5d460da0d33355f62987b89bf35b08ce0c86c1b82de |
| **Android ARM64** | konata-native-android-arm64-r1.zip | c7498af5ec521ef752a803b50a4b6f35c05beced55e3ecc693acaff8bd0daa4c |

## 构建要求与流程

构建环境基于 Linux / WSL（Ubuntu 24.04），需要 Python 3.12、Podman、NDK r30 等环境。详细构建指令见 [README.original.md](README.original.md) 与各构建脚本。

## 许可证

- 构建脚本与工程代码遵循 **GNU General Public License v3.0 or later（GPL-3.0-or-later）** 开源；
- Windows 预编译组件遵循 **GNU Lesser General Public License v3.0 or later（LGPL-3.0-or-later）**；
- Android 预编译组件启用 GPL 特性，遵循 **GNU General Public License v3.0 or later（GPL-3.0-or-later）**。
