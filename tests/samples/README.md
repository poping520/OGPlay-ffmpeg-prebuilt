# 冒烟样本

全部由 FFmpeg 合成源（`testsrc2` 画面、`sine` 音调）生成。期望结果见 `samples.json`，
由 `tools/build_ffmpeg.py smoke` 判定。纯音频样本的 `expect` 省略视频字段，检查编码名、
采样率、声道数、`min_audio_samples`（每声道 PCM 样本数下限）和 seek。seek 后须能解码
所选流的一帧，有视频时选视频流，纯音频时选音频流。

| 文件 | 覆盖点 |
| --- | --- |
| `h264-high-aac.mp4` | H.264 High profile + B 帧重排，AAC-LC 立体声，faststart |
| `mpeg4-aac-moov-at-end.mp4` | MPEG-4 Part 2 Simple Profile，moov 在文件尾（custom AVIO seek） |
| `h263-aac.3gp` | 3GP 容器，H.263（mpeg4 解码器的隐含依赖） |
| `hevc-aac.mp4` | 未编入的编码：必须按名称报告 `unsupported`，不得崩溃或伪造成功 |
| `mp3-id3-tags.mp3` | 约 0.3 秒的 44.1 kHz 立体声 CBR，含 ID3v2.3 头标签与 ID3v1 尾标签 |
| `mp3-vbr-xing.mp3` | 约 2 秒的 22.05 kHz 单声道 VBR，含 Xing/LAME 头，覆盖纯音频 seek |
| `vorbis.ogg` | 约 2 秒的 44.1 kHz 立体声 Ogg Vorbis |
| `aac-audio-only.m4a` | 约 1 秒的 44.1 kHz 单声道 AAC/MP4 |
| `opus.ogg` | Ogg 可解封装，Opus 必须报告 `unsupported`（`stream=audio`、`codec=opus`） |

在本目录生成样本。使用 Homebrew FFmpeg 8.1（含 libx264、libx265、libmp3lame、libopus）。
Vorbis 样本用 FFmpeg 原生编码器，Xing/LAME 样本用 Homebrew 的 `lame` 写入 LAME 标签。

```sh
F() { ffmpeg -hide_banner -loglevel error -y "$@"; }
F -f lavfi -i testsrc2=s=160x90:r=12:d=2 -f lavfi -i sine=f=440:r=44100:d=2 \
  -c:v libx264 -profile:v high -bf 2 -g 12 -pix_fmt yuv420p \
  -c:a aac -b:a 64k -ac 2 -movflags +faststart h264-high-aac.mp4
F -f lavfi -i testsrc2=s=160x90:r=12:d=2 -f lavfi -i sine=f=660:r=22050:d=2 \
  -c:v mpeg4 -q:v 6 -g 12 -c:a aac -b:a 32k -ac 1 mpeg4-aac-moov-at-end.mp4
F -f lavfi -i testsrc2=s=176x144:r=12:d=2 -f lavfi -i sine=f=330:r=22050:d=2 \
  -c:v h263 -q:v 8 -g 12 -c:a aac -b:a 32k -ac 1 -f 3gp h263-aac.3gp
F -f lavfi -i testsrc2=s=160x96:r=12:d=1 -f lavfi -i sine=f=440:r=44100:d=1 \
  -c:v libx265 -x265-params log-level=error -tag:v hvc1 -pix_fmt yuv420p \
  -c:a aac -b:a 32k -ac 1 hevc-aac.mp4
F -f lavfi -i sine=f=440:r=44100:d=0.3 -c:a libmp3lame -b:a 128k -ac 2 \
  -metadata title="ID3 smoke test" -id3v2_version 3 -write_id3v1 1 -write_xing 0 mp3-id3-tags.mp3
F -f lavfi -i sine=f=660:r=22050:d=2 -c:a pcm_s16le -f wav - | \
  lame --silent -V 4 - mp3-vbr-xing.mp3
F -f lavfi -i sine=f=330:r=44100:d=2 -c:a vorbis -strict experimental -q:a 3 -ac 2 vorbis.ogg
F -f lavfi -i sine=f=550:r=44100:d=1 -c:a aac -b:a 48k -movflags +faststart aac-audio-only.m4a
F -f lavfi -i sine=f=440:r=48000:d=1 -c:a libopus -b:a 32k opus.ogg
```
