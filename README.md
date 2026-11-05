# OGPlay FFmpeg Prebuilt SDK

OGPlay 用于播放游戏 CG 的裁剪版 FFmpeg 二进制包。所有文件只能由
[OGPlay-ffmpeg](../OGPlay-ffmpeg) 的 `tools/build_ffmpeg.py` 生成和验证，不要手工修改
二进制或清单。

## 布局

```text
include/                公共头（各平台共用）+ manifest.json
licenses/               FFmpeg 许可证（各平台共用）+ manifest.json
<platform>/
  manifest.json         源码 tag/commit、configure 参数、组件、库版本、依赖、逐文件 SHA-256
  exports.txt           导出符号列表
  lib/libogplay_ffmpeg.dylib            macOS
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

- macOS：install name 为 `@rpath/libogplay_ffmpeg.dylib`，最低系统版本见清单。
- Windows：MSVC 构建，动态 CRT（`/MD`）。

## 校验

```sh
python3 ../OGPlay-ffmpeg/tools/build_ffmpeg.py verify --prebuilt .
```

## 许可证与源码

FFmpeg 以 LGPL-2.1-or-later 发布，许可证统一放在根目录 `licenses/`；单独分发任一平台时
须一并携带该目录。对应源码为清单中
`ffmpeg_commit` 指向的上游 FFmpeg 提交，构建参数见清单 `configure_args`，构建脚本位于
OGPlay-ffmpeg 仓库。
