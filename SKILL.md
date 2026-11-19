---
name: hyperframes
description: Build and render videos with HyperFrames (HeyGen's open-source HTML/CSS/GSAP-to-video framework) — motion graphics, explainers, Shorts, ads, and captioned videos written as HTML compositions and rendered to MP4/WebM with the hyperframes CLI. Use for any request to create, edit, animate, or render a video with HyperFrames.
---

# HyperFrames — Video from HTML

HyperFrames turns an HTML page into a video. You write a **composition** (HTML + CSS + GSAP animations on a seekable timeline), and the CLI renders it frame-by-frame in headless Chrome to MP4/WebM. If you can build a web page, you can build the video.

- Framework: HyperFrames by HeyGen (open source) — https://hyperframes.heygen.com
- Docs index (machine-readable, do not guess URLs): https://hyperframes.heygen.com/llms.txt
- Local reference in terminal: `npx hyperframes docs <topic>` — topics: `data-attributes`, `gsap`, `compositions`, `rendering`, `examples`, `troubleshooting`

## Install

```bash
npm install -g hyperframes      # or use npx hyperframes everywhere
hyperframes doctor              # verify Chrome + FFmpeg are reachable
```

Requirements: Node.js, FFmpeg on PATH, Chrome/Chromium (the CLI manages its own headless shell).

## The contract — non-negotiable rules

Breaking any of these produces a broken or blank render:

1. **Root element** carries `data-composition-id`, `data-width`, `data-height`, `data-start="0"`, `data-duration="<total seconds>"`.
2. **Every timed element** has `data-start` and `data-duration` (seconds). `data-start` is what makes an element timed at all.
3. **Timed visual elements get `class="clip"`** — the shared `.clip` CSS gives a scene its full-frame box; lint warns without it.
4. **One paused GSAP timeline per composition**, registered by composition id:
   ```js
   window.__timelines = window.__timelines || {};
   window.__timelines["main"] = gsap.timeline({ paused: true });
   ```
   Scene timelines added into the root must NOT be paused themselves.
5. **Deterministic only** — no `Date.now()`, no `Math.random()`, no network fetches at render time. Same input must render the same frames.
6. **Vendor libraries locally.** Load GSAP from a local `vendor/` copy, never a CDN — renders must work offline and identically every time.
7. **Audio lives on the media element.** Footage with sound: `<video data-has-audio="true">` (never `muted`). Silent b-roll: `muted`. Music/voiceover/replacement audio: separate `<audio>` elements.

## Minimal composition

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=1080, height=1920" />
    <script src="vendor/gsap.min.js"></script>
    <style>
      html, body { margin: 0; width: 1080px; height: 1920px; overflow: hidden; background: #0b0e14; }
      .clip { position: absolute; inset: 0; display: flex; align-items: center; justify-content: center; }
      #title { color: #fff; font-size: 110px; font-weight: 800; text-align: center; }
    </style>
  </head>
  <body>
    <div id="root" data-composition-id="main" data-start="0" data-duration="6"
         data-width="1080" data-height="1920">
      <h1 id="title" class="clip" data-start="0" data-duration="6">Hello</h1>
    </div>
    <script>
      const tl = gsap.timeline({ paused: true });
      tl.fromTo("#title", { opacity: 0, y: 60 }, { opacity: 1, y: 0, duration: 0.5, ease: "power4.out" }, 0.2);
      tl.to("#title", { opacity: 0, duration: 0.4 }, 5.4);
      window.__timelines = window.__timelines || {};
      window.__timelines["main"] = tl;
    </script>
  </body>
</html>
```

## Data attributes (the timing language)

| Attribute | Meaning |
|---|---|
| `data-start` | Start time in seconds |
| `data-duration` | Duration in seconds |
| `data-track-index` | Studio display lane only — the render ignores it; use CSS z-index for paint order |
| `data-media-start` | Media trim/offset point (seconds) |
| `data-volume` | Gain: 1 = 0 dB, 0 = silence, up to 3.98 (+12 dB) |
| `data-fade-in` / `data-fade-out` | Clip-edge audio fades in seconds |
| `data-has-audio="true"` | Keep the media file's own sound on this clip |
| `data-composition-src` | Embed another composition file (sub-composition) |
| `data-composition-variables` | On `<html>`: JSON array declaring variables `{id, type, label, default}` |
| `data-variable-values` | On a sub-comp host: JSON object of per-instance overrides |

GSAP rules: timelines always `{ paused: true }`; the 3rd argument to `to/from/fromTo` is the **absolute time position**; animate only seek-safe properties (opacity, x, y, scale, rotation, width, height, visibility). Do NOT animate `autoAlpha` on `.clip` elements — it fights the runtime's visibility control.

## Sub-compositions & variables

Split long videos into scene files under `compositions/` and mount them from `index.html`:

```html
<div data-composition-id="scene1" data-composition-src="compositions/scene1.html"
     data-start="0" data-duration="8"></div>
```

Declare variables once, reuse the same scene with different text/colors per mount via `data-variable-values`, read them with `window.__hyperframes.getVariables()`, and override at render time with `hyperframes render --variables '{"title":"Pro"}'`.

## The work loop — always in this order

```bash
npx hyperframes init my-video        # scaffold a project
# ... write/edit index.html and compositions/ ...
npm run check                        # lint + runtime + layout + motion + contrast — fix ALL errors
npx hyperframes snapshot --at 1,3,5  # PNG frames at those seconds — LOOK at them before rendering
npx hyperframes render               # MP4 out (add -q draft while iterating)
```

A passing check does not show how it looks. Snapshot the moments you edited and read the frames. Render draft quality while iterating; render delivery quality once.

Preview for a human: `npx hyperframes preview --background` (agent-safe persistent Studio), verify with `preview --status`, stop with `preview --stop`. Never leave a foreground `npm run dev` as the handoff.

## Render options that matter

```bash
npx hyperframes render -q draft|looks|delivery   # quality presets (default: looks)
npx hyperframes render --fps 30                  # or root data-fps
npx hyperframes render --crf 18                  # encoder quality override (vs --video-bitrate)
npx hyperframes render -w 4                      # parallel workers (4 is the usual sweet spot)
npx hyperframes render --docker                  # deterministic production render (exact Chrome/fonts)
npx hyperframes render -o out/final.mp4
```

## Formats

- **YouTube / landscape:** 1920×1080
- **Shorts / Reels / TikTok:** 1080×1920 — keep text and faces out of the **top ~220 px** and **bottom ~420 px** (platform UI overlays cover them)
- **Square social:** 1080×1080

## Useful extras in the CLI

- `hyperframes add` / `catalog` — ready-made registry blocks & components; search before hand-building a named visual
- `hyperframes transcribe` — word-level transcript for captions
- `hyperframes tts` — local speech (Kokoro-82M) for voiceover
- `hyperframes beats` / `audio` — beat/onset detection to sync cuts to music
- `hyperframes remove-background` — transparent media from video/image
- `hyperframes publish` — upload the project to a shareable URL
- `hyperframes benchmark` — compare render configs for speed/size

## Troubleshooting

- **Blank/black render** → timeline not registered on `window.__timelines`, or timeline left unpaused children; check with `hyperframes validate`
- **Element missing mid-video** → missing `data-duration`, or a paused child timeline that never advances
- **Fonts/layout shift between preview and render** → CDN assets; vendor everything locally, prefer `--docker` for final
- **Stutter** → animating non-seek-safe properties (filters/layout thrash); stick to transform + opacity

## This user's environment (Muse's VM)

- CLI v0.9.19 installed at `~/workspace/.npm-global`; **always call the shim `~/workspace/bin/hyperframes`** — it sets the browser path (Remotion's Chrome Headless Shell), TMPDIR, and frames cache to workspace paths that survive this VM's limits
- GSAP etc. must be vendored into the project (`vendor/`), never CDN — same rule as above, enforced here by shimmed network behaviour
- Verified working: test project `~/workspace/hyperframes-test` rendered a 10 s 1920×1080 MP4 end-to-end
