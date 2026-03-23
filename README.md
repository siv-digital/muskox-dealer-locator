# MUSKOX Dealer Locator

## Summary

Interactive dealer locator widget for muskoxmn.com that displays all MUSKOX dealer and distributor locations on a searchable map. Users can search by city, state, or zip code and filter by distance radius (25/50/100 miles). Each dealer pin shows name, address, website, and directions link.

**Live URL:** https://www.muskoxmn.com/find-dealer

## Architecture

```
Google Sheet (source of truth)
    ↓ Published CSV URL
GitHub Pages (siv-digital.github.io/muskox-dealer-locator/)
    ↓ iframe embed
Wix (muskoxmn.com/find-dealer)
```

- **Frontend:** Single self-contained `index.html` — no build step, no dependencies to install
- **Map:** Leaflet.js 1.9.4 with OpenStreetMap tiles (free, no API key)
- **Data:** Google Sheets published as CSV, fetched on every page load
- **Geocoding:** OpenStreetMap Nominatim (for search queries and dealers missing lat/lng in the sheet)
- **Hosting:** GitHub Pages — auto-deploys on push to `main`
- **Wix integration:** iframe pointing to GitHub Pages URL — no manual Wix update needed after code deploys

## Data Source

Dealer data lives in a Google Sheet with these columns:

| Column | Purpose | Required |
|--------|---------|----------|
| Name | Dealer/business name | Yes |
| Tags | Dealer classification (e.g. "Inventory Dealer Location") | No |
| Address | Street address | Yes |
| City | City | Yes |
| State | State/Province | Yes |
| Postcode | Zip/Postal code | Yes |
| Country | Country | Yes |
| Website | Dealer website URL | No |
| Latitude | Decimal latitude (e.g. 45.1195176) | Recommended |
| Longitude | Decimal longitude (e.g. -95.0651955) | Recommended |

The published CSV URL is configured in `CONFIG.SHEET_CSV_URL` inside `index.html`.

## Coordinate Resolution (Priority Order)

1. **Sheet columns** — `Latitude` and `Longitude` values from the Google Sheet (primary)
2. **Inline DEALER_COORDS** — Pre-geocoded fallback cache embedded in the HTML, keyed by CSV row index (legacy, kept for backward compatibility)
3. **Nominatim auto-geocode** — If both above are missing, the widget geocodes from the full address at page load time (least reliable)

## Adding a New Dealer

1. Add a row to the Google Sheet with all required fields
2. Get the lat/lng: right-click the address in Google Maps → copy coordinates
3. Paste latitude and longitude into the sheet columns
4. The dealer appears on the map automatically on next page load (no code deploy needed)

If you skip the lat/lng, Nominatim will attempt to geocode from the address. This works for most addresses but can fail for ambiguous place names (e.g. "Henderson" resolving to Nevada instead of Colorado).

## Configuration

All configuration lives in the `CONFIG` object at the top of `index.html`:

| Setting | Purpose |
|---------|---------|
| `SHEET_CSV_URL` | Published Google Sheet CSV URL |
| `COLUMNS` | Maps sheet column headers to data fields |
| `DEFAULT_CENTER` | Initial map center `[lat, lng]` — default US center |
| `DEFAULT_ZOOM` | Initial zoom level — default 4 |
| `MARKER_COLOR` | Pin color — `#ffff00` (MUSKOX yellow) |

## Deployment

Push to `main` → GitHub Pages auto-deploys → Wix iframe loads updated version.

```bash
cd clients/muskox/technical/dealer-locator
# make changes to index.html
git add index.html
git commit -m "Description of change"
git push origin main
```

No Wix changes needed unless the GitHub Pages URL changes.

## Project History

### v1 — Initial Build (Feb 2026)

- Built self-contained HTML widget with Leaflet.js and OpenStreetMap
- Data sourced from Google Sheets CSV
- Pre-geocoded all dealer coordinates using Nominatim and stored them in an inline `DEALER_COORDS` JavaScript object, keyed by CSV row index
- Deployed to GitHub Pages, embedded on Wix via iframe
- ~100 dealer locations across US and Canada

### v2 — Coordinate Fix + Sheet Migration (Mar 23, 2026)

**Problem discovered:** Three dealer pins were in the wrong location despite correct city/state data in the sheet. Root cause: Nominatim returned incorrect geocode results during the initial bulk geocoding.

**Bad geocodes fixed:**
- **Farm Rite Equipment (Willmar, MN)** — was pinned near Arcadia, WI (~200 miles off). Caused by "Wilmar" misspelling in the sheet confusing the geocoder.
- **Hardline Equipment (Henderson, CO)** — was pinned in Henderson, NV (~750 miles off). Nominatim resolved "Henderson" to the more well-known Nevada city.
- **Dave's Repair (Hills, MN)** — was pinned near Rochester, MN (~193 miles off). Correct state but wrong city within Minnesota.

**Structural fix:** The inline `DEALER_COORDS` approach was fragile — coordinates keyed by row index could silently break if rows were added/removed/reordered, and there was no way for MUSKOX to manage coordinates without a code deploy.

**Changes made:**
1. Added `Latitude` and `Longitude` columns to the Google Sheet
2. Audited all 88 dealer coordinates via reverse geocoding — confirmed all others correct at state level
3. Updated `CONFIG.COLUMNS` to read lat/lng from sheet columns (`'Latitude'`, `'Longitude'`)
4. Updated `SHEET_CSV_URL` to the new published sheet with coordinate columns
5. Added auto-geocode fallback via Nominatim for new dealers added without lat/lng
6. Kept inline `DEALER_COORDS` as secondary fallback for backward compatibility
7. Fixed "Wilmar" → "Willmar" spelling in the data

**Result:** MUSKOX can now manage dealer locations entirely from the Google Sheet — add a row with lat/lng and it appears on the map. No code deploys needed for data changes.

## Repository

- **GitHub:** https://github.com/siv-digital/muskox-dealer-locator
- **Branch:** `main`
- **GitHub Pages:** https://siv-digital.github.io/muskox-dealer-locator/
