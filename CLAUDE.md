# DharmaBox

A single-page kiosk web app that plays a 508-lecture Buddhist series for Raymond's mother on a Lenovo Tab One Android tablet. The entire app is one self-contained HTML file with no build step, no backend, and no dependencies beyond the YouTube IFrame API.

## Live URL

**https://lorecrafting.github.io/dharma-box/**

GitHub repo: https://github.com/lorecrafting/dharma-box

Deployed via GitHub Pages from the `main` branch root. Push to `main` → live in ~60 seconds.

## Files

```
index.html      — The entire app (playlist data embedded, ~56 KB)
playlist.json   — Source of truth for all 508 video IDs/titles/durations
```

No build step. Edit `index.html`, push, done.

## The Playlist

**淨空老法師 二零一四淨土大經科註** — Venerable Master Jing Kong, 2014 Pure Land Scripture Annotations.

- 508 lectures, ~107 min average, ~906 hours total
- Original YouTube playlist: `https://www.youtube.com/watch?v=Fs0qCZuGSJg&list=PLj3-6Dw0D3Q7ox3n7zsYPsJdh5aZ9IA5s`
- All 508 video IDs are embedded directly in `index.html` as a JS array — no API calls needed at runtime

If the playlist ever needs to be re-scraped, `yt-dlp` is installed at `/opt/homebrew/bin/yt-dlp` and `playlist.json` is the regeneration source.

## Architecture

Pure HTML/CSS/JS, no frameworks. YouTube IFrame API streams the video. Key design decisions:

- **Single menu button** in the bottom bar — all navigation is behind one intentional tap to prevent accidental prev/next by the mother
- **Drawer** slides up on menu tap, contains: large Prev/Next buttons + full scrollable lecture list (TOC)
- **Auto-advance** fires 1.5 s before the video ends (via `setInterval` polling `getCurrentTime`) so YouTube's "more videos" end-screen never appears
- **Position saving** — `currentIdx` is stored in `localStorage` under key `dharmabox_idx`; resumes on next visit
- **Error handling** — `onError` auto-advances to next lecture after 2 s

## YouTube Player Config

```js
playerVars: {
  autoplay: 1,
  controls: 1,       // YouTube's native controls for pause/seek within a lecture
  iv_load_policy: 3, // no annotations
  modestbranding: 1,
  rel: 0,            // limits end-screen suggestions to same channel
  playsinline: 1,
}
```

Known limitation: the Share and More Videos (⋮) buttons inside the YouTube player cannot be removed — they live in a cross-origin iframe. The only way to fully eliminate them is `controls: 0` plus building custom play/pause and seek controls.

## Kiosk Deployment (Lenovo Tab One)

1. Open **https://lorecrafting.github.io/dharma-box/** in **Fully Kiosk Browser** (free tier, Play Store)
2. Set that URL as the Start URL in Fully Kiosk Browser settings
3. Enable Kiosk Mode + Start on Boot
4. OR use Android built-in **Screen Pinning** (Settings → Security → Screen Pinning) as a no-app alternative

## Context / Constraints

- **Offline**: WiFi signal disappears entirely at 11 pm. The simple streaming approach means the app stops working at 11 pm. A Raspberry Pi running a local caching server was discussed as a future upgrade but not built.
- **Storage**: Lenovo Tab One has 64 GB / 19 GB used. Full video download is not feasible (906 hours ≈ 120+ GB even at lowest quality). Audio-only at 64 kbps would be ~25 GB but was not pursued.
- **No API key needed**: playlist data is baked into the HTML, no YouTube Data API calls at runtime.

## UI Summary

```
┌──────────────────────────────┐
│   [YouTube player]           │  full height, controls=1
│                              │
├──────────────────────────────┤
│   [  ≡  目錄  ]              │  single subtle full-width button
└──────────────────────────────┘

Tap 目錄 → drawer slides up:
  [◀ 上一講]   [下一講 ▶]
  ── 目錄 · 共 508 講 ──
  1   淨空老法師...
  2   淨空老法師...  ← active (gold highlight, auto-scrolled)
  ...508 items...
```

Tapping a TOC item or Prev/Next navigates and closes the drawer. Tapping the dark backdrop or × closes without navigating.
