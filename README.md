# Ohio School Districts Interactive Map

An interactive map showing all 612 Ohio school districts built for the [Vouchers Hurt Ohio](https://vouchershurtohio.com/) coalition — a coalition of Ohio school districts opposing the EdChoice voucher program.

## Live Demo

🗺️ **[https://awillard44.github.io/VHO-district-map/](https://awillard44.github.io/VHO-district-map/)**

## Features

- ✅ All 612 Ohio school districts rendered with GeoJSON boundaries
- ✅ Coalition member highlighting — **blue (#04A6E1)** for members, **orange (#E13F04)** for non-members
- ✅ Searchable district dropdown with two-way map sync (search zooms map, clicking map updates dropdown)
- ✅ Hover tooltips and click info panel showing district name, county, and coalition status
- ✅ Animated stats counters (total districts and coalition count, counted dynamically from GeoJSON on load)
- ✅ No basemap tile layer — clean GeoJSON-only rendering on plain background
- ✅ Map bounded to Ohio with min zoom 7
- ✅ Responsive design for mobile

## Quick Start

### 1. Download the GeoJSON data

**Option A: From the API Explorer (Recommended)**
1. Go to: https://ohio-framework-datasets-geohio.hub.arcgis.com/datasets/13c69be53f854a3cb5806a9422da2bf5/api
2. Click "Open in API Explorer"
3. Under "Output Options", set format to GeoJSON
4. Click "Get Response" and save the result as `ohio-school-districts.geojson`

**Option B: Direct API URL**
```
https://maps.ohio.gov/arcgis/rest/services/Hosted/Ohio_School_Districts_2025/FeatureServer/0/query?where=1%3D1&outFields=*&outSR=4326&f=geojson
```
Open this URL in your browser and save the page as `ohio-school-districts.geojson`.

### 2. Add the file to this project

Place `ohio-school-districts.geojson` in the same folder as `index.html`.

### 3. Run locally

Browsers block loading local JSON files directly, so you need a local web server:

```bash
# Python 3
python -m http.server 8000

# Node.js
npx serve .

# PHP
php -S localhost:8000
```

Then open http://localhost:8000 in your browser.

## Data Sources

### District Boundaries (GeoJSON)

| Resource | URL |
|----------|-----|
| Data Portal | https://ohio-framework-datasets-geohio.hub.arcgis.com/datasets/13c69be53f854a3cb5806a9422da2bf5/api |
| API (GeoJSON) | `https://maps.ohio.gov/arcgis/rest/services/Hosted/Ohio_School_Districts_2025/FeatureServer/0/query?where=1%3D1&outFields=*&outSR=4326&f=geojson` |

### Available GeoJSON Fields

| Field | Description |
|-------|-------------|
| `irn` | Ohio Information Retrieval Number — unique 6-digit district ID |
| `name` | District name |
| `taxid` | Tax ID |
| `countycode` | Ohio county code (zero-padded string, e.g. `"04"`) |
| `SHAPE__Length` | Perimeter (not displayed) |
| `SHAPE__Area` | Area (not displayed) |
| `globalid` | Internal ID (not displayed) |

### Additional Data Sources

To extend the map with enrollment, funding, or performance data, join on the `irn` field — it's the unique identifier used across all Ohio education datasets.

| Data Type | Source |
|-----------|--------|
| Enrollment Data | https://education.ohio.gov/Topics/Data/Frequently-Requested-Data/Enrollment-Data |
| Report Cards | https://reportcard.education.ohio.gov/download |
| All District Data | https://oeds.ode.state.oh.us/dataextract |

## Coalition Member List

The map highlights all 286 Vouchers Hurt Ohio coalition members in blue. Members are identified by **IRN (Information Retrieval Number)** — Ohio's unique 6-digit district ID. IRN-based matching was chosen over name-string matching to eliminate ambiguity bugs (e.g. three different "Eastern Local School Districts" exist in Brown, Meigs, and Pike counties — only Brown and Meigs are VHO members).

To update the member list:

1. Get the current list from: https://vouchershurtohio.com/districts/
2. Look up each district's IRN via the [Ohio Education Directory](https://oeds.ode.state.oh.us/)
3. Edit `index.html` and find the `coalitionMembers` Set (around line 561)
4. Add or remove 6-digit IRN strings

```javascript
const coalitionMembers = new Set([
    '043489', // Akron City
    '044230', // Toledo City
    '043786', // Columbus City
    // ... 283 more verified IRN codes
]);
```

> **Note:** IRN codes are zero-padded strings (`'043489'`, not `43489`). The `getDistrictIRN()` helper handles both lowercase `irn` and uppercase `IRN` field names from the GeoJSON.

## WordPress Integration

The map is being integrated into [vouchershurtohio.com](https://vouchershurtohio.com/), which runs WordPress with the Genesis framework and Altitude Pro child theme.

**Approach:** A custom WordPress page template (`page-[slug].php`) placed in the child theme directory. This loads the map's HTML/CSS/JS natively within the site's normal navigation and footer — no iframe required.

**Alternative options considered:**

| Option | Best For |
|--------|----------|
| Custom page template *(selected)* | Dedicated map page, full layout control |
| Shortcode plugin `[vho_coalition_map]` | Map appearing on multiple pages |
| Gutenberg custom block | Most native WP experience |
| WPCode/Code Snippets plugin | No direct server file access |

The iframe embed approach was tested and rejected — it produced a redundant nested header and had mobile responsiveness issues.

## Deployment

### GitHub Pages (Current)

The map is live at **https://awillard44.github.io/VHO-district-map/**. To deploy your own fork:

1. Push this repo to GitHub
2. Go to Settings → Pages
3. Set source to the `main` branch, `/ (root)` folder
4. Your map will be live at `https://yourusername.github.io/VHO-district-map/`

> **Note:** If the GeoJSON exceeds GitHub's 25MB file limit, use [mapshaper.org](https://mapshaper.org) to simplify/compress it first.

### WordPress Custom Page Template

Once the page slug is confirmed by VHO stakeholders, a `page-[slug].php` template will be added to the Altitude Pro child theme. The map's assets (HTML/CSS/JS) will be enqueued via `wp_enqueue_scripts` and the GeoJSON hosted on the WordPress server or a CDN.

## File Structure

```
VHO-district-map/
├── index.html                      # Main map application (HTML + CSS + JS)
├── ohio-school-districts.geojson   # District boundaries (~28MB)
├── VHO_logo.png                    # VHO logo displayed in the header
└── README.md                       # This file
```

## Technical Details

| Item | Detail |
|------|--------|
| Mapping Library | [Leaflet.js](https://leafletjs.com/) v1.9.4 |
| Basemap | None — GeoJSON renders on plain background |
| Fonts | Bebas Neue (headers), Headland One (body) via Google Fonts |
| District matching | IRN-based (`coalitionMembers` Set of 286 IRN strings) |
| Build process | None — pure HTML/CSS/JS, single file |
| Hosting | GitHub Pages |

### Known Syntax Pitfall

Optional chaining (`?.`) and nullish coalescing (`??`) operators must have **no spaces**. GitHub's web editor has previously auto-corrected these to `? .` and `? ?`, causing an `Unexpected token '.'` error that silently breaks the entire script. Always edit `index.html` locally or verify these operators after any web-based edits.

## Future Enhancements

- [ ] Include enrollment data on hover
- [ ] Add voucher impact data ($ lost per district)
- [ ] Accessibility improvements (keyboard navigation, screen reader support)
- [ ] Alternative table view for accessibility

## License

- Map code: MIT License
- District boundary data: Public domain (Ohio OGRIP)
- Leaflet.js: BSD-2-Clause License

## Credits

- District boundaries from [Ohio Geographically Referenced Information Program (OGRIP)](https://ogrip.oit.ohio.gov/)
- Built for [Vouchers Hurt Ohio](https://vouchershurtohio.com/)
- Developed by [Alex Willard](https://github.com/awillard44) / [Precision New Media](https://precisionnewmedia.com/)
