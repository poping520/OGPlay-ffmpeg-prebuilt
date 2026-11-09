# OGPlay FFmpeg Prebuilt SDK

此仓库保存 OGPlay 用于播放游戏 CG 的裁剪版 FFmpeg SDK，以及生成和验证 SDK 的
构建工具。OGPlay 通过 `third_party/ffmpeg-prebuilt` 使用 SDK。二进制和清单由
`tools/build_ffmpeg.py` 生成，构建流程见[维护手册](tools/README.md)。

## 布局

```text
include/                公共头（各平台共用）+ manifest.json
licenses/               FFmpeg 许可证（各平台共用）+ manifest.json
<platform>/
  manifest.json         源码 tag/commit、configure 参数、组件、库版本、依赖、逐文件 SHA-256
  exports.txt           导出符号列表
  lib/libogplay_ffmpeg.dylib            macOS
  lib/libogplay_ffmpeg.so               Android
  bin/ogplay_ffmpeg.dll                 Windows 运行时
  lib/ogplay_ffmpeg.lib                 Windows 导入库
```

已发布平台以各平台目录是否存在为准，详细事实以对应 `manifest.json` 为准。
平台清单的 `shared_licenses` 指向根目录 `licenses/`。

## 内容

单一共享库 `ogplay_ffmpeg` 内含 avformat、avcodec、swscale、swresample、avutil，只导出
FFmpeg 公共 API；编译期直接链接使用，不需要也不支持运行时按名称加载 FFmpeg。
组件仅限 `mov` demuxer 与 `h264`/`mpeg4`/`h263`/`aac` 解码器（另含对应 parser），无任何
protocol：媒体必须经自定义 `AVIOContext` 读入。依赖只有系统库。

- macOS 的 install name 为 `@rpath/libogplay_ffmpeg.dylib`，最低系统版本见清单。
- Windows 使用 MSVC 动态 CRT（`/MD`）。
- Android 支持 `arm64-v8a`，最低 API 23，ELF 加载段按 16 KiB 对齐。

## 校验

```sh
python3 tools/build_ffmpeg.py verify
```

## 许可证与源码

FFmpeg 以 LGPL-2.1-or-later 发布，许可证统一放在根目录 `licenses/`；单独分发任一平台时
须一并携带该目录。对应源码为 [`ffmpeg.json`](ffmpeg.json) 固定的上游 FFmpeg
提交，构建参数见平台清单的 `configure_args`。维护者可用 `sync` 将源码取到
本地 `ffmpeg/`。源码和 `.local/` 下的构建输出均由 Git 忽略。
