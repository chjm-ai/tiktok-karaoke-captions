---
name: tiktok-karaoke-captions
description: Burn TikTok-style karaoke captions (per-word yellow highlight, ALL CAPS, bold sans-serif) and an optional persistent headline banner into any local video file. Runs fully offline on Apple Silicon via mlx-whisper + ffmpeg, with bundled open-source fonts (Roboto Black, Archivo Black). Three usage tiers — pure auto-caption (just give a video), script-aligned (provide the original script for zero typos via difflib forced alignment), or full TikTok package (script + headline). Use when the user wants to add subtitles / captions / 字幕 to a video, asks for "TikTok-style captions" / "karaoke captions" / "word-by-word highlighted captions" / "caption a video" / "burn subtitles into a video" / "auto-subtitle this video" / "给视频加字幕" / "做卡拉OK字幕" / "TikTok字幕风格" / "顶部加 headline 横幅". Also use when the user provides a video file and a script and wants a captioned output. The skill is fully standalone — no specific workflow assumed; the user just provides a video path (and optionally a script.txt and/or a headline string). Handles English by default; supports any language Whisper supports via `--language`.
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
- ~700 MB disk for first-run downloads (whisper model + deps + static-ffmpeg)

## Bundled fonts (open-source, commercial OK)

- **Archivo Black** (OFL 1.1) — headline pill
- **Roboto Black / Bold / Regular** (Apache 2.0) — captions

See [`fonts/README.md`](fonts/README.md) for details and licenses.

## CLI reference

Run `caption.py --help` for the full list of flags.
