# Examples

## Try it on your own video

```bash
# 1. Just auto-caption a video — Whisper transcribes from scratch
python3 ../caption.py /path/to/your/video.mp4
```

Output lands beside your video as `video-captioned.mp4`.

## With a script (recommended for accuracy)

Create a plain text file with what's spoken in the video:

```bash
cat > my-script.txt <<'EOF'
Hello everyone. Today I'm reviewing the new iPhone.
The camera is incredible. The battery lasts all day.
Don't miss this one!
EOF

python3 ../caption.py /path/to/your/video.mp4 \
    --script-file my-script.txt
```

Whisper still detects WHEN each word is spoken, but the displayed text comes from your script verbatim — zero typos.

## Add a TikTok-style headline banner

```bash
python3 ../caption.py /path/to/your/video.mp4 \
    --script-file my-script.txt \
    --headline "iPhone 16 PRO REVIEW"
```

The headline gets a black semi-transparent pill at the top of the frame, persistent for the whole video. Long headlines auto-wrap onto 2–3 lines.

## Test with a sample audio (no video)

If you only have an audio file, wrap it into a video first:

```bash
ffmpeg -loop 1 -i black.png -i audio.mp3 -shortest -tune stillimage \
    -c:v libx264 -c:a copy test.mp4
python3 ../caption.py test.mp4 --script-file test-script.txt
```

## Common combos

```bash
# More accurate but slower (medium model)
python3 ../caption.py video.mp4 --script-file script.txt --model medium

# Chinese audio
python3 ../caption.py video.mp4 --language zh

# Up to 4 words per chunk
python3 ../caption.py video.mp4 --max-words-per-chunk 4

# Keep original casing
python3 ../caption.py video.mp4 --no-uppercase

# Just generate SRT, don't burn into video
python3 ../caption.py video.mp4 --srt-only

# Classic line-level subtitles instead of karaoke
python3 ../caption.py video.mp4 --caption-mode classic
```
