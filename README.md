# Train Trap

Status page for the train trap: a live river-conditions frame plus a gallery
of recent train art, with the trap's own captions. Built to sit in an iframe
on skamaniadispatch.com (Ghost), hosted on GitHub Pages.

No build step. One HTML file. **No Dropbox credentials anywhere** — the trap
already publishes everything on public shared folder links, and this site
reads only those.

## How the plumbing works

The trap's Dropbox app folder exposes three shared folders:

- `river/latest.jpg` — overwritten about every 5 minutes. The page hot-links
  it directly and re-pulls every `REFRESH_SECONDS` (default 120) with a
  cache-buster. Truly live, no middleman.
- `latest/01.jpg 02.jpg 03.jpg + latest.json` — the current top-3 art shots
  at fixed names, with ranks, captions, and capture times in the JSON.
- `daily/YYYY-MM-DD/` — per-day archive keyed by event id.

Dropbox serves no CORS headers, so the page's JavaScript cannot read
`latest.json` directly (images are fine, JSON is not). The GitHub Action at
`.github/workflows/dropbox-sync.yml` bridges that: every 30 minutes it reads
`latest.json` over the public link, downloads any art event it hasn't seen
into `gallery/`, records it in `gallery/index.json`, and regenerates
`manifest.json` for the page. It commits only when new art appears. Over
time the site accumulates its own archive of every piece the trap ranks,
without ever needing to list the Dropbox folders.

`gallery/` was seeded on 2026-08-02 with all nine shots then in Dropbox
(2026-05-29, 2026-05-30, 2026-07-18); the two May days predate captions, so
they carry `archive · <date>` labels.

Only photos are ever published. The train recordings stay in Dropbox and
nothing in this repo can touch them.

## Freshness

- River frame: live, at most `REFRESH_SECONDS` + Dropbox's ~60s cache stale.
- New art: appears within one Action cycle (≤ ~30 min, GitHub cron is
  best-effort). Lower the cron to `*/15` if that ever feels slow.

## Themes

Default is the amber CRT look. Adding `?theme=light` switches to a white
theme matched to skamaniadispatch.com (Inter, black text, `#6bafd1` accent).
One deploy serves both.

## Ghost embed

On the Ghost page, add an **HTML card**. For skamaniadispatch.com:

```html
<iframe src="https://kate-rose.github.io/train-trap/?theme=light"
        style="width:100%; height:950px; border:0; background:#ffffff"
        loading="lazy" title="Train Trap"></iframe>
```

For a dark page, drop the `?theme=light` and use `background:#0c0906`.

## If a Dropbox link ever breaks

The shared-link URLs (in `index.html` CONFIG and the workflow env block) are
tied to the shared links themselves. If a link is revoked and re-shared, or
a file is deleted and re-created rather than overwritten, grab the fresh
per-file URLs from the folder's public web view and swap them in those two
places.

## Local preview

```bash
python3 -m http.server 5187 -d ~/train-trap
```

Then open <http://localhost:5187/> (add `?theme=light` for the white look).
