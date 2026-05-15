# tiktok-karaoke-captions

Burn TikTok-style karaoke captions and a persistent headline banner into any local video, fully offline on macOS Apple Silicon.

- **Per-word yellow highlight** as words are spoken — the signature TikTok karaoke effect
- **Forced-alignment from a script file** when you have one — text comes from your script verbatim, timing comes from Whisper (zero typos)
- **Bundled open-source fonts** (Roboto Black, Archivo Black) — works on any machine, commercial-use safe
- **One-pass ffmpeg burn** — no separate `.srt`/`.mp4` mux step

```bash
# Just give it a video — captions auto-generated
python3 caption.py my_video.mp4
```

## Demo flow

```
input video (.mp4)              ──┐
                                  ├─→ caption.py ──→  video-captioned.mp4
script.txt   (optional)         ──┤                   video.srt
"HEADLINE"   (optional)         ──┘                   video.ass
```

## Install

Prerequisites:
- **macOS Apple Silicon** (M1/M2/M3/M4) — the underlying ASR (`mlx-whisper`) is Apple Silicon only
- [`uv`](https://github.com/astral-sh/uv) — `brew install uv`

Then clone:

```bash
git clone https://github.com/chjm-ai/tiktok-karaoke-captions.git
cd tiktok-karaoke-captions
```

That's it. The first run downloads the Whisper model and other deps automatically and caches them forever after (~700 MB total).

## Usage

### 1. Pure auto-caption (no script)

```bash
python3 caption.py my_video.mp4
```

Uses `whisper-small` to transcribe and emit karaoke captions. Output lands beside the input video as `my_video-captioned.mp4`.

### 2. Script-aligned (recommended — zero typos)

If you know what was said, give it the script as a `.txt` (or `.md`) file. Whisper still provides the word-level timestamps, but the displayed text comes from your script — so no transcription errors ever reach the screen.

```bash
python3 caption.py my_video.mp4 --script-file script.txt
```

How the alignment works: `difflib.SequenceMatcher` aligns the Whisper word sequence against the script word sequence; matched script words inherit the matched Whisper word's `(start, end)`; unmatched words are linearly interpolated from neighbors.

### 3. Full TikTok package (script + headline)

Add a persistent top banner that lasts the whole video:

```bash
python3 caption.py my_video.mp4 \
    --script-file script.txt \
    --headline "BLACK FRIDAY · 50% OFF"
```

The headline auto-wraps onto 1–3 lines and auto-fits its font size to the pill width. Long headlines or natural separators (`·` `—` `/` `|`) get split nicely.

## Caption styles

```bash
--caption-mode tiktok    # default: karaoke ASS, per-word yellow highlight
--caption-mode classic   # static line-level SRT, white-on-black, sentence-grouped
```

## All flags

| Flag | Default | Meaning |
|---|---|---|
| `video` | (required) | Input video file path |
| `--script-file PATH` | — | Script text file for forced alignment |
| `--script TEXT` | — | Inline script (alternative to `--script-file`) |
| `--headline TEXT` | — | Persistent top-banner text |
| `--headline-font PATH` | bundled Archivo Black | TTF for headline |
| `--caption-mode` | `tiktok` | `tiktok` or `classic` |
| `--max-words-per-chunk N` | 3 | Words per karaoke chunk |
| `--no-uppercase` | off | Keep original casing in karaoke |
| `--max-chars-per-line N` | 42 | Soft cap for sentence-split |
| `--model` | `small` | `tiny`/`base`/`small`/`medium`/`large` |
| `--language` | `en` | Whisper language code |
| `--out-dir DIR` | input dir | Where to write outputs |
| `--out-name NAME` | `<stem>-captioned.mp4` | Output video filename |
| `--srt-only` | off | Generate subtitle files only, skip burn-in |

## Outputs

Files written to `--out-dir` (default = same dir as input video):

- `<stem>.srt`              — line-level SRT (always)
- `<stem>.ass`              — karaoke ASS (only when `--caption-mode tiktok`)
- `<stem>.whisper.json`     — raw Whisper output for debugging
- `<stem>-captioned.mp4`    — final video with text burned in

## Performance (M4, 16 GB)

| Model | First run | Subsequent runs (15-sec video) | Quality |
|---|---|---|---|
| tiny | ~2 min download + 5 sec | ~3 sec | Many typos |
| **small** | ~3 min download + 5 sec | **~5 sec** | Recommended |
| medium | ~6 min download + 10 sec | ~10 sec | Near-perfect for English |

Without `--script-file`, accuracy is bounded by Whisper. With `--script-file`, accuracy is 100% (text comes from your script).

## Why open-source fonts?

Captioning videos for commercial distribution means embedding fonts into the rendered pixels. Mac system fonts (Helvetica, Arial Black, etc.) have license terms that *technically* restrict redistribution. We bundle replacements that are unambiguously commercial-OK:

| Use | Bundled font | License |
|---|---|---|
| Headline | Archivo Black | SIL OFL 1.1 |
| Captions | Roboto Black/Bold | Apache 2.0 |

Both allow free embedding, modification, and commercial use.

## License

MIT — see [LICENSE](LICENSE).

## Limitations

- macOS Apple Silicon only (Whisper backend is `mlx-whisper`)
- Currently English caption styling is the default; CJK works but may need a CJK-glyph font swap via `--headline-font`
- Single audio track only (uses first audio stream)
