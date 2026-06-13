---
name: tiktok-karaoke-captions
description: Burn TikTok-style karaoke captions (per-word yellow highlight, ALL CAPS, bold sans-serif) + optional headline banner into a local video, fully offline on Apple Silicon (mlx-whisper + ffmpeg). Use when the user wants to add subtitles / captions / 字幕 to a video, asks for "TikTok-style captions" / "karaoke captions" / "word-by-word highlighted captions" / "caption a video" / "burn subtitles into a video" / "auto-subtitle this video" / "给视频加字幕" / "做卡拉OK字幕" / "TikTok字幕风格" / "顶部加 headline 横幅", or provides a video file (and optionally a script.txt and/or headline string) wanting a captioned output. Do NOT use for: generating SRT for an online/streaming video without a local file; editing/translating an existing .srt's text (that's a text task); pure transcription with no burn-in and no styling (just run whisper); non-macOS or non-Apple-Silicon hosts (mlx-whisper won't run — needs a different backend).
---

# TikTok Karaoke Captions

Burn TikTok-style karaoke captions and a persistent headline banner into any local video, fully offline on macOS Apple Silicon.

## When to use

Trigger this skill when the user wants any of:
- 给视频加字幕 / add subtitles to a video
- TikTok 风格字幕 / karaoke captions / per-word highlight captions
- 顶部 headline 横幅 / persistent headline / video title overlay
- 用 Whisper 转录视频 / auto-transcribe a video
- 字幕烧入视频 / burn subtitles into a video
- 用脚本原文校准字幕 / forced-alignment subtitles from a known script

## Three usage tiers

(moved out of the frontmatter — pick based on what the user gives you)

1. **Pure auto-caption** — just a video. Whisper transcribes, captions burned. Fastest, but typos possible.
2. **Script-aligned (zero typos)** — video + `--script-file script.txt`. `difflib` forced-aligns the known script onto Whisper word timestamps, so the burned text is exactly the script. Use whenever the original script exists.
3. **Full TikTok package** — script + `--headline "..."`. Adds a persistent top headline pill on top of tier 2.

## Quick reference

```bash
# 1. Pure auto-caption
python3 ~/Desktop/Repos/AI_Skills/tiktok-karaoke-captions/caption.py video.mp4

# 2. Script-aligned (zero typos)
python3 ~/Desktop/Repos/AI_Skills/tiktok-karaoke-captions/caption.py video.mp4 \
    --script-file script.txt

# 3. Full TikTok package
python3 ~/Desktop/Repos/AI_Skills/tiktok-karaoke-captions/caption.py video.mp4 \
    --script-file script.txt --headline "BLACK FRIDAY · 50% OFF"

# Common flags
--caption-mode classic        # static line-level SRT instead of karaoke
--max-words-per-chunk 4       # 1–4 words per chunk (default 3)
--no-uppercase                # keep original casing
--model small                 # lighter model (480 MB vs default medium 1.5 GB)
--language zh                 # Chinese audio
--srt-only                    # just generate SRT/ASS, no burn-in
--out-dir ./output            # custom output dir
```

## Outputs (in `--out-dir` or alongside the input)

- `<stem>.srt`              — line-level SRT (always written)
- `<stem>.ass`              — karaoke ASS (tiktok mode only)
- `<stem>.whisper.json`     — raw Whisper word timestamps (debug)
- `<stem>-captioned.mp4`    — final video with text burned in

## Mechanism

1. **Audio extract**: ffmpeg → 16 kHz mono WAV
2. **Transcribe**: `uvx --from mlx-whisper mlx_whisper` (Apple Silicon native, ~10 sec for 15s video at medium model after warmup; auto-retries with bigger model if output looks broken)
3. **Align**: if `--script-file` given, `difflib.SequenceMatcher` maps each script word → Whisper word timestamp (forced alignment, zero typos)
4. **Chunk**: split into 1–3 word chunks at sentence/comma boundaries
5. **ASS karaoke**: each chunk → N events, current word highlighted yellow via `{\c&H0000FFFF&}…{\c}` inline tags
6. **Burn**: single ffmpeg pass — `drawbox` + `drawtext` for headline pill + `subtitles` filter for ASS
7. **ffmpeg fallback**: if system ffmpeg lacks libass (Homebrew bottle), auto-uses `static-ffmpeg` from PyPI via `uvx` (no system changes)

## Requirements

- macOS on Apple Silicon (mlx-whisper requirement)
- `uv` installed (`brew install uv`) — only system dependency
- ~1.8 GB disk for first-run downloads (whisper-medium model + deps + static-ffmpeg)

## Optional: Deepgram cloud (faster + more reliable than local)

Set `DEEPGRAM_API_KEY` env var to enable Deepgram Nova-3 — when the key is set, **it becomes the default primary backend** (faster: ~2s vs ~10s, and more reliable than local mlx-whisper). Local Whisper is used as the fallback when Deepgram fails. Pass `--prefer-local` to flip this back to local-first. Free $200 credit at https://console.deepgram.com/signup. Cost ~$0.001 per 15s clip.

## Bundled fonts (open-source, commercial OK)

- **Archivo Black** (OFL 1.1) — headline pill
- **Roboto Black / Bold / Regular** (Apache 2.0) — captions

See [`fonts/README.md`](fonts/README.md) for details and licenses.

## Gotchas（来自真实失败，照这个排错）

- **中文/非英文音频不加 `--language` 会被转成英文乱码。** `--language` 默认 `en`，Whisper 会把中文音轨硬听成英文（满屏拼音/错词）。音频不是英文时**必须**显式传 `--language zh`（或对应语言码）。脚本对齐模式（tier 2）也救不了它——对齐前的转录就已经错了。
- **字幕固定在底部中央（`Alignment=2, MarginV=70`），不会自动避让人脸。** 如果说话人脸在画面下半部，字幕会压在脸/嘴上。本工具不做人脸检测。补救：classic 模式用 `--style "Alignment=8,MarginV=70"`（顶部）或调大 `MarginV` 把字幕抬高；或先确认主体构图在上 2/3 再烧字幕。给竖屏数字人视频时尤其要先看一眼人脸位置。
- **每行字符上限是软上限 42（`--max-chars-per-line`，对齐脚本时生效）。** 超长行会被按句读/逗号切；中文一个字≈一个 char，但占的视觉宽度比英文字母宽，42 对中文偏长，竖屏建议调到 ~16–20，否则字会顶到屏幕边缘。headline 另有独立按宽度自适应换行逻辑。
- **缩略图 / 转场帧会出现"半句字幕"。** 烧字幕是按 Whisper 时间轴的，视频开头的转场帧 / 封面帧那一瞬间可能正好卡在某个 chunk 中间，截图当缩略图时会看到残缺字幕。要干净封面就单独抽一帧没有字幕的原片帧，别从 captioned.mp4 截。
- **系统 ffmpeg 缺 libass 时会自动回退 static-ffmpeg，但首次要联网下 ~60MB。** Homebrew 的 ffmpeg bottle 常被裁掉 libass（`subtitles` filter 用不了）。脚本检测到就用 `uvx --from static-ffmpeg`，不改系统。**离线首跑会失败**——确保第一次有网把 static-ffmpeg 和 whisper 模型拉下来。
- **转录看着"太短/太空"会自动换更大模型重试。** 真很短的视频（语音少）可能误判触发重试，拖慢。确认音频确实稀疏时直接 `--model medium` 跳过 small 的重试循环。

## CLI reference

Run `caption.py --help` for the full list of flags.
