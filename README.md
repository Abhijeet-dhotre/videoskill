# videoskill

A skill for building and rendering videos with [HyperFrames](https://hyperframes.heygen.com) — HeyGen's open-source HTML/CSS/GSAP-to-video framework.

Write a video as an HTML composition, and the HyperFrames CLI renders it frame-by-frame in headless Chrome to MP4/WebM. If you can build a web page, you can build the video: motion graphics, explainers, Shorts, ads, and captioned videos.

## What's in this repo

- `SKILL.md` — the full skill: the HyperFrames contract, data attributes, sub-compositions and variables, the work loop, render options, formats, and troubleshooting.
- `README.md` — this file.

## Install the skill

```bash
npx skills add Abhijeet-dhotre/videoskill
```

Or copy `SKILL.md` into your agent's skills folder manually.

## Install HyperFrames

Requires Node.js, FFmpeg on PATH, and Chrome/Chromium (the CLI manages its own headless shell).

```bash
npm install -g hyperframes      # or use npx hyperframes everywhere
hyperframes doctor              # verify Chrome + FFmpeg are reachable
```

## Quick start

```bash
npx hyperframes init my-video        # scaffold a project
# ... write/edit index.html and compositions/ ...
npm run check                        # lint + runtime + layout + motion + contrast — fix ALL errors
npx hyperframes snapshot --at 1,3,5  # PNG frames at those seconds — LOOK at them before rendering
npx hyperframes render               # MP4 out (add -q draft while iterating)
```

Preview for a human: `npx hyperframes preview --background`, verify with `preview --status`, stop with `preview --stop`.

## The contract, in short

1. Root element carries `data-composition-id`, `data-width`, `data-height`, `data-start="0"`, `data-duration="<total seconds>"`.
2. Every timed element has `data-start` and `data-duration` (seconds).
3. Timed visual elements get `class="clip"`.
4. One paused GSAP timeline per composition, registered on `window.__timelines` by composition id.
5. Deterministic only — no `Date.now()`, no `Math.random()`, no network fetches at render time.
6. Vendor libraries locally (`vendor/`), never a CDN.
7. Audio lives on the media element: footage with sound uses `<video data-has-audio="true">` (never `muted`); silent b-roll is `muted`; music/voiceover is a separate `<audio>` element.

See `SKILL.md` for the full rules, a minimal composition, and troubleshooting.

## Formats

- **YouTube / landscape:** 1920×1080
- **Shorts / Reels / TikTok:** 1080×1920 — keep text and faces out of the top ~220 px and bottom ~420 px (platform UI overlays cover them)
- **Square social:** 1080×1080

## Render options

```bash
npx hyperframes render -q draft|looks|delivery   # quality presets (default: looks)
npx hyperframes render --fps 30
npx hyperframes render --crf 18
npx hyperframes render -w 4                      # parallel workers
npx hyperframes render --docker                  # deterministic production render
npx hyperframes render -o out/final.mp4
```

## Useful extras

- `hyperframes add` / `catalog` — ready-made registry blocks & components
- `hyperframes transcribe` — word-level transcript for captions
- `hyperframes tts` — local speech (Kokoro-82M) for voiceover
- `hyperframes beats` / `audio` — beat/onset detection to sync cuts to music
- `hyperframes remove-background` — transparent media from video/image
- `hyperframes publish` — upload the project to a shareable URL
- `hyperframes benchmark` — compare render configs for speed/size

## Docs

- HyperFrames site: https://hyperframes.heygen.com
- Machine-readable docs index: https://hyperframes.heygen.com/llms.txt
- Local reference: `npx hyperframes docs <topic>` — topics: `data-attributes`, `gsap`, `compositions`, `rendering`, `examples`, `troubleshooting`
