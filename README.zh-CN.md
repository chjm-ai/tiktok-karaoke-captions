# tiktok-karaoke-captions

[English](README.md) · **中文**

给本地视频烧入 TikTok 风格卡拉 OK 字幕和顶部 headline 横幅，全本地运行，macOS Apple Silicon 专属。

<p align="center">
  <img src="docs/preview.png" alt="加完字幕的视频效果预览 — 顶部 pill 横幅 'VIETNAM BUYER / FIRST VISIT' + 底部卡拉 OK 字幕 'GOT IT.'（'IT.' 被高亮成黄色）" width="320">
</p>

- **逐词黄色高亮** — 词被说到时变黄，没到时白色，TikTok 标志性的卡拉 OK 节奏感
- **原稿强制对齐**（可选） — 给一份脚本 .txt，字幕文字 100% 用原稿，时间戳交给 Whisper，**零错字**
- **持续顶部 headline 横幅** — Archivo Black 粗体配半透明黑底 pill，自动换行 + 自动字号适配
- **自带开源字体**（Roboto Black、Archivo Black） — 任何机器都能跑，商用许可证完全干净
- **ffmpeg 一次烧入** — 不用单独导 SRT 再合成视频

```bash
# 给一个视频，自动加字幕
python3 caption.py my_video.mp4
```

## 流水线

```
输入视频 (.mp4)                ──┐
                                  ├─→ caption.py ──→  video-captioned.mp4
原稿  script.txt   (可选)       ──┤                   video.srt
顶部  HEADLINE     (可选)       ──┘                   video.ass
```

## 安装

前置：
- **macOS Apple Silicon**（M1/M2/M3/M4） — 底层用 `mlx-whisper`，仅支持苹果芯片
- [`uv`](https://github.com/astral-sh/uv) — `brew install uv`

然后 clone：

```bash
git clone https://github.com/chjm-ai/tiktok-karaoke-captions.git
cd tiktok-karaoke-captions
```

完事。首次运行会自动下载 Whisper 模型和依赖（共 ~700MB），之后永久缓存。

## 用法

### 1. 纯自动字幕（无脚本）

```bash
python3 caption.py my_video.mp4
```

用 `whisper-small` 自动转录并出卡拉 OK 字幕。输出落在原视频旁边，叫 `my_video-captioned.mp4`。

### 2. 原稿对齐（推荐 — 零错字）

如果你知道视频里说的是什么，把它写成一个 `.txt`（或 `.md`）。Whisper 还是负责听出每个词的时间戳，但屏幕上显示的文字完全来自你的脚本——任何 ASR 错字都不会上屏。

```bash
python3 caption.py my_video.mp4 --script-file script.txt
```

工作原理：`difflib.SequenceMatcher` 把 Whisper 转录的词序列和脚本词序列做对齐，匹配上的脚本词继承 Whisper 的 `(start, end)`，没匹配上的词从相邻锚点线性插值。

### 3. 完整 TikTok 套装（脚本 + 顶部横幅）

加一个贯穿全片的顶部 banner：

```bash
python3 caption.py my_video.mp4 \
    --script-file script.txt \
    --headline "黑五大促 · 五折"
```

Headline 自动换 1-3 行，字号按 pill 宽度自适应。带自然分隔符（`·` `—` `/` `|`）的长 headline 会断得很合理。

## 字幕风格

```bash
--caption-mode tiktok    # 默认：卡拉 OK ASS，逐词黄色高亮
--caption-mode classic   # 经典：行级 SRT，白字黑边，按句分组
```

## 完整参数

| 参数 | 默认 | 说明 |
|---|---|---|
| `video` | （必填） | 输入视频文件路径 |
| `--script-file PATH` | — | 原稿 txt 文件，用于强制对齐 |
| `--script TEXT` | — | 内联原稿（`--script-file` 的备选） |
| `--headline TEXT` | — | 顶部持续横幅文字 |
| `--headline-font PATH` | 自带 Archivo Black | Headline 用的 TTF 字体 |
| `--caption-mode` | `tiktok` | `tiktok` 或 `classic` |
| `--max-words-per-chunk N` | 3 | 卡拉 OK 模式每屏最多几个词 |
| `--no-uppercase` | 关 | 保留原大小写，不强制全大写 |
| `--max-chars-per-line N` | 42 | 切句时单段字符上限 |
| `--model` | `small` | `tiny`/`base`/`small`/`medium`/`large` |
| `--language` | `en` | Whisper 语言代码 |
| `--out-dir DIR` | 输入视频所在目录 | 输出目录 |
| `--out-name NAME` | `<stem>-captioned.mp4` | 输出视频文件名 |
| `--srt-only` | 关 | 只出字幕文件，不烧进视频 |

## 输出

输出到 `--out-dir`（默认是输入视频所在目录）：

- `<stem>.srt`              — 行级 SRT（每次都写）
- `<stem>.ass`              — 卡拉 OK ASS（仅 `--caption-mode tiktok` 时）
- `<stem>-whisper.json`     — Whisper 原始词级输出（debug 用）
- `<stem>-captioned.mp4`    — 最终烧好字幕的视频

## 性能（M4 / 16 GB 实测）

| 模型 | 首次跑 | 后续（15 秒视频） | 准确率 |
|---|---|---|---|
| tiny | 下载 ~2 分钟 + 5 秒 | ~3 秒 | 错字多 |
| **small** | 下载 ~3 分钟 + 5 秒 | **~5 秒** | 推荐 |
| medium | 下载 ~6 分钟 + 10 秒 | ~10 秒 | 英文近乎完美 |

没有 `--script-file` 时准确率受限于 Whisper；有 `--script-file` 时准确率 100%（文字直接来自脚本）。

## 为什么用开源字体？

视频商用分发意味着字体像素会被烧进每一帧。Mac 系统字体（Helvetica、Arial Black 等）**严格来讲**有再分发限制。我们打包了完全无许可证负担的替代品：

| 用途 | 自带字体 | 许可证 |
|---|---|---|
| Headline | Archivo Black | SIL OFL 1.1 |
| 字幕 | Roboto Black/Bold | Apache 2.0 |

两个许可证都允许免费嵌入、修改、商用。

## 许可证

MIT — 见 [LICENSE](LICENSE)。

## 已知限制

- 仅支持 macOS Apple Silicon（Whisper 后端用的是 `mlx-whisper`）
- 默认按英文风格调字号；中文也能跑，需要换支持中文字形的字体（用 `--headline-font` 指定）
- 仅取首条音轨
