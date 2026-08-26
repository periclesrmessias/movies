# Movie Dashboard

A single-file HTML dashboard for couples (or any duo) to track movies watched together: ratings, streaming platforms, genres, charts, and a "spin the wheel" picker for what to watch next. No backend, no build step — data lives in a Google Sheet and the page just reads it.

![status](https://img.shields.io/badge/stack-HTML%20%2B%20JS-blue) ![data](https://img.shields.io/badge/data-Google%20Sheets%20CSV-green)

## What it does

- **KPI cards**: total movies, watched vs pending counts, individual and combined average ratings, and a highlight card for the last movie watched.
- **Filters**: by status (watched/pending), genre, streaming service, and free-text search.
- **Sortable table**: every movie with both people's ratings, the combined average, IMDb rating, and watch date. Click any column header to sort.
- **Charts** (via Chart.js): movie count by genre, average rating by genre per person, a scatter plot comparing both people's ratings side by side, and a rolling-average trend of ratings over time.
- **Random picker**: draws a random movie from the currently filtered pending list — useful for "we can't decide what to watch" nights.
- **Offline-safe fallback**: if the spreadsheet can't be reached, the page falls back to a locally embedded snapshot of the data so it never shows a blank screen.

## How it works

The entire app is one `index.html` file: HTML, CSS, and JavaScript inline, no build tools, no dependencies to install. It only pulls two things over the network:

1. **Chart.js**, loaded from a CDN (with a second CDN as fallback if the first fails).
2. **Your data**, as a CSV, fetched directly from a published Google Sheet.

On load, the page fetches the CSV, parses it with a small custom parser (handles quoted fields, commas and line breaks inside cells), transforms each row into a movie object, and renders everything from that in-memory array. Every filter, sort, and chart interaction re-runs against that same array — there's no server round-trip after the initial load.

## Setting up your own copy

### 1. Create the spreadsheet

Create a Google Sheet with one row per movie and these columns (header names are up to you, just keep the mapping consistent when you edit the parsing code):

| Column | Meaning | Example |
|---|---|---|
| Movie title | Display name | `Whiplash` |
| Watched (person A) | Boolean | `TRUE` / `FALSE` |
| Watched (person B) | Boolean | `TRUE` / `FALSE` |
| Date watched | `DD/MM/YYYY` or empty | `12/06/2026` |
| Streaming service | Free text | `Netflix`, `HBO`, `Cinema`... |
| Rating (person A) | Number 0–5, or blank | `4.3` |
| Rating (person B) | Number 0–5, or blank | `4.6` |
| Genre | Free text | `Drama` |
| Trailer link (optional) | URL | `https://youtube.com/...` |
| Original title (optional) | For a tooltip | `Whiplash` |
| Add any other column you want| - | - |

A movie counts as "watched" once both people have marked it. The combined rating is simply the average of the two individual ratings (or just one, if only one person rated it).

### 2. Publish the sheet as CSV

In Google Sheets: **File → Share → Publish to web**, choose the specific tab, select **CSV** as the format, and publish. Copy the generated URL.

### 3. Point the page at your sheet

Open `index.html` and update this line near the top of the `<script>` block:

```js
const SHEET_CSV_URL = "https://docs.google.com/spreadsheets/d/e/.../pub?gid=...&single=true&output=csv";
```

### 4. Adjust the CSV-to-object mapping

The function that turns each CSV row into a movie object expects specific column positions or names. Update it to match your sheet's actual column order or headers, and adjust the `FALLBACK_DATA` array (a hardcoded snapshot near the bottom of the script) so the page still shows something meaningful if the fetch fails.

### 5. Customize the look

All colors and identity are CSS variables at the top of the `<style>` block:

```css
:root{
  --gabrielle: #e0568c;  /* person A's color, used in charts and ratings */
  --pericles: #4aa8e6;   /* person B's color */
  --accent: #e6b04a;     /* combined/couple accent color */
  ...
}
```

Rename the CSS classes (`.gabrielle`, `.pericles`) and the labels in the HTML/JS to match your own names, and swap the colors to taste.

### 6. Host it

Since it's a static file with no backend, any static host works: GitHub Pages, Netlify, Vercel, or just opening the file locally. GitHub Pages is the simplest free option:

1. Push `index.html` to a repository.
2. Enable GitHub Pages for that repo (Settings → Pages → deploy from branch).
3. Share the resulting URL.

## Updating your data

No redeploy needed. Just edit the Google Sheet — the page fetches fresh data (`cache: 'no-store'`) every time it loads.

## Tech notes

- **No frameworks.** Vanilla JS, DOM manipulation, and Chart.js for the four charts.
- **CSV parsing is hand-rolled** to correctly handle quoted fields containing commas or newlines, since Google's published CSV export can include those.
- **Resilience by design**: a missing Chart.js load, a failed CSV fetch, or a rendering error in any single widget won't break the rest of the page — each render function is wrapped and failures are logged to the console with a visible banner shown to the user when relevant.
- **No backend, no auth, no database.** This trades live collaborative editing (you edit the sheet, not the page) for zero maintenance and zero cost.

## Possible extensions

- Add a "watchlist priority" score or tags.
- Support more than two raters.
- Add a poster fetch from an external movie API (OMDb, TMDb) using the title as the query.
- Persist the random-draw history so the same pending movie isn't suggested twice in a row.
