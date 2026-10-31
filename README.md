# Rewind

**Point your phone's camera at a place. Swipe left to see what it looked like through the years.**

Rewind is a single-file web app. It uses your location (and compass, when available) to pull geo-tagged historical photos of the spot you're standing at from [Wikimedia Commons](https://commons.wikimedia.org), lays them over your live camera view, and lets you scrub backward in time with a swipe.

> Status: working prototype. No build step, no backend, no API keys.

## How it works

1. **Locate** — `navigator.geolocation` gives lat/lon. (Or type a place name; it's geocoded via Wikipedia article coordinates.)
2. **Fetch** — one call to the Commons API: `generator=geosearch` within a radius (150 m – 3 km) for files in namespace 6, with `imageinfo` + `extmetadata` so each photo comes back with a thumbnail URL and its *date taken*.
3. **Date** — `DateTimeOriginal` is messy ("1912", "circa 1900", "2015-06-12 14:03:22"). A regex pulls the first plausible year; "circa"/"?" is flagged. Falls back to the upload date (also flagged *circa*).
4. **Timeline** — photos are sorted newest → oldest. Index `-1` is *now* (the live camera). Swipe left = older, swipe right = newer. Tap to peek at the live view underneath.
5. **Aim** (optional) — with the compass on, each photo's bearing from you is computed and only photos within ±45° of where the phone is pointing are shown. Works well for landmarks, less so for street scenes (archive photos rarely record their facing direction).

Everything lives in `index.html`: markup, styles, and ~300 lines of vanilla JS.

## Running it

The camera, GPS, and compass APIs only work on **HTTPS** (or `localhost`). Opening the file directly (`file://`) will show the UI but none of the sensors.

**On a phone (the real experience)** — host `index.html` on any HTTPS URL. GitHub Pages is the easiest:

```
Settings → Pages → Source: "Deploy from a branch" → main / (root)
```

Then open `https://<user>.github.io/rewind/` on your phone, tap **Start**, allow camera + location, and tap the compass icon (iOS needs an explicit motion-permission tap).

**On a laptop (archive only)** — any static server works:

```bash
npx serve .          # or: python3 -m http.server 8000
```

Open `http://localhost:8000`, pick a demo location (Pike Place Market, Times Square, …) or type a place, and use ← / → to move through time. No camera behind the photos, but the whole timeline flow is testable.

## Controls

| Gesture / key | Action |
|---|---|
| Swipe left / `←` | One photo back in time |
| Swipe right / `→` | One photo forward (back to *now*) |
| Tap photo | Peek at the live camera for ~1 s |
| Tap a timeline dot | Jump to that photo |
| `Home` | Back to *now* |
| 150 m / 400 m / 1 km / 3 km | Search radius |
| **Aim** | Only photos in the direction you're facing |
| ⟳ | Re-acquire GPS |

## Data & limits

- **Source:** Wikimedia Commons geo-tagged files, max 50 per query. Coverage is deep for landmarks and thin for ordinary streets.
- **Dates** are as good as the uploader's metadata. Expect some *circa*.
- **No scene recognition yet.** The app knows *where you are* and *which way you face*, not *what's in frame*. See roadmap.
- Commons API is called anonymously with CORS (`origin=*`); be a good citizen and don't hammer it.

## Roadmap

- [ ] **Scene matching** — embed the live frame (CLIP / DINOv2) and rank archive photos by visual similarity, so pointing at a specific building returns *that* building.
- [ ] **Image alignment** — register the old photo onto the live view (homography from feature matches) for a true in-place cross-fade.
- [ ] **More sources** — Flickr (geo + date, needs key), city archives, Historypin, Library of Congress.
- [ ] **Pinch-to-fade** between now and then.
- [ ] **Cache** last results per location in IndexedDB for offline replays.
- [ ] PWA manifest + add-to-home-screen.

## License

MIT for the code. Photos remain under their individual Commons licenses — the caption links each one back to its source page.
