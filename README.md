# Tecity News Dashboard

A single-file, self-contained HTML dashboard that collates each trading day's articles from four newspaper subscriptions — **WSJ**, **The Straits Times**, **The Business Times** and **SCMP** — into one briefing, with a rolling window of the 7 most recent trading days.

Open `Tecity_Dashboard.html` in any browser. No server, no build step, no network required.

---

## What's in this folder

| Path | What it is |
| --- | --- |
| `Tecity_Dashboard.html` | **The dashboard.** Bookmark this. Rebuilt on every run; contains the last 7 trading days of data embedded as JSON. |
| `Tecity_News_Dashboard_Master_Prompt.md` | **The operating spec (v3.1).** The authoritative, step-by-step workflow Claude follows on each run. Read this before changing anything. |
| `_build/Dashboard_Template.html` | The design source of truth — all CSS, inlined React + ReactDOM, and the pre-compiled UI component, with a single `/*__ALL_DAYS__*/` placeholder where the data is injected. |
| `Archive/YYYY-MM-DD.json` | Full payloads for days that have rolled out of the 7-day window. Nothing is deleted, only archived. |
| `Archive/_backup/Tecity_Dashboard.prev.html` | The previous dashboard, copied here before each overwrite. Restore point if a write goes wrong. |
| `Publish (GitHub Pages)/index.html` | Deploy copy of the dashboard for the optional public mirror. |
| `<D MMM YYYY>/` | Daily source folders (e.g. `29 Jun 2026`), each containing four subfolders `<DD MMM> WSJ`, `... ST`, `... BT`, `... SCMP` full of article PDFs. These are the raw input. |

---

## How to run it

In a Claude Cowork session with this folder attached as a **writable** workspace folder, say:

> **run Tecity News Dashboard**

That runs the whole workflow end to end without further prompting. In short, it:

1. Resolves today's date in Singapore time (UTC+8) and finds today's dated source folder.
2. Enumerates the PDFs in the four source subfolders.
3. Reads each PDF's text (locally, or via the Microsoft 365 connector if the file is a OneDrive cloud-only placeholder).
4. Synthesises the day's payload — executive digest, 6 briefing cards, 6 key metrics, and a 3–4 sentence summary per article tagged from a fixed 10-tag taxonomy.
5. Merges it into the rolling window, backfilling any missing trading days whose source folders exist, and archiving days that fall out.
6. Rebuilds `Tecity_Dashboard.html` from the template, backs up the old file, and copies the result to the publish folder.
7. Reports back in chat: the signal of the day, three items worth flagging, and housekeeping.

The full detail — payload schema, tag taxonomy, card themes, OneDrive URL pattern, quality bar, failure handling — lives in `Tecity_News_Dashboard_Master_Prompt.md`. That file, not this README, is the spec.

### Before you run

- Folder attached to Cowork with **read/write** access.
- Folder set to **"Always keep on this device"** in OneDrive, so the PDFs are hydrated rather than cloud-only placeholders.
- **Microsoft 365 connector** connected — used to resolve source links and to read any unhydrated PDF.
- A dated folder for **today** exists. If it doesn't, the run stops rather than showing stale data.

---

## Design and build rules

Two rules matter more than the rest:

**Self-contained.** The output must have zero external script dependencies — no CDN `<script src>`, no in-browser Babel. React and ReactDOM are inlined and the UI component is transpiled to plain `React.createElement` JavaScript ahead of time. Relying on `unpkg.com` previously produced blank pages when the network blocked it. Google Fonts links may stay; they degrade to system fonts offline.

**Template is the single source of design truth.** Day-to-day runs change only the injected data. Edit the look by editing `_build/Dashboard_Template.html`, never the generated `Tecity_Dashboard.html`.

The visual language is premium-minimal: Inter, neutral palette, light-first with a dark toggle, sticky header with a date picker, full-width executive digest hero, 6-up metric grid, an Overview tab (featured lead story + briefing grid + coverage chart) and a Feed tab (search + source/tag filter pills). Source colours: WSJ `#D85A30`, ST `#1D9E75`, BT `#378ADD`, SCMP `#D4537E`. Target file size under 1 MB.

Every build is verified: file ends with `</html>`, embedded JSON parses, size under 1 MB, zero CDN references, and the page populates `#root` with no network. A failed verification restores the backup rather than leaving a broken file to sync.

---

## Publishing the public mirror

The dashboard is a static snapshot, so the mirror needs re-publishing after each run — though because of the date picker, one upload keeps the whole rolling week current. Source links point at OneDrive web URLs and require a tenant sign-in.

Setup is one-time: install Git, clone the public repo to `%USERPROFILE%\tecity-dashboard`, sign in once interactively. After that, publishing is a double-click on `Publish to GitHub.bat` or a weekday Windows Task Scheduler job running `publish.ps1`.

> **Note:** `Publish (GitHub Pages)/` currently contains only `index.html`. The `publish.ps1`, `Publish to GitHub.bat` and `AUTO_PUBLISH_SETUP.md` files described in the spec are not present in this folder — they need to be (re)created before automated publishing will work.

---

## Current state

- Rolling window: **2026-06-19 → 2026-06-29** (7 trading days).
- `Tecity_Dashboard.html` — 645 KB, self-contained, zero CDN references, tail intact.
- No dated source folders are present in the workspace root right now, so a fresh run will stop at the precondition check until today's folder is available locally.

---

## Troubleshooting

| Symptom | What to do |
| --- | --- |
| Run stops immediately | Today's dated folder is missing, or the folder isn't attached read/write. Deliberate — it won't serve stale data. |
| Articles summarised from filenames only | Those PDFs were cloud-only placeholders and the connector read also failed. Set the folder to "Always keep on this device" and re-run. |
| Blank page when opened | Should not happen — the build is offline-capable. If it does, check for CDN `<script src>` tags, which mean the template was rebuilt incorrectly. |
| Dashboard looks broken or truncated | Restore `Archive/_backup/Tecity_Dashboard.prev.html` over `Tecity_Dashboard.html`, then re-run. |
| Bookmarked link shows old data | OneDrive sync lag. Give it a minute; check the sync icon. |
| Live site shows old data | The mirror wasn't re-published. Run the publish step. |
