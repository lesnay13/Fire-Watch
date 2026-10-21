# Fire-Watch

**Fire-Watch uses decades of Washington wildfire history, weather, and fire station locations to suggest where firefighting crews will matter most this season.**

Built for the Seattle Kiro Buildathon, *"Where does Washington burn?"*

---

## The problem

Every spring, a WA DNR Wildfire Division resource coordinator (our persona, "Dana") has to split a fixed number of crews and engines across DNR's six regions. Fire-Watch answers one question:

> *"I have a fixed number of crews. Where and when do they matter most this season?"*

## What it does

- **Where fire demand has been:** fire counts and acres burned by county and region, 1970 to 2007.
- **When it peaks:** fire activity by week of the year, and whether the season is getting longer.
- **How weather shifts it:** spring rainfall compared with summer fire activity, by county.
- **Where coverage is thin:** distance from each historical fire to the nearest fire station.
- **What to do about it:** a suggested share of crews for each DNR region, with a plain-language "why" for each one.
- **Did it hold?** The same checks re-run on 2008 to present, so the advice isn't built on outdated patterns.

All insight is **deterministic** (counts, math, and comparisons). There is no runtime AI in the analysis.

## Data sources

| Data | Source | Used for |
|---|---|---|
| DNR Fire Statistics 1970-2007 (38,116 fires) | WA DNR public wildfire data (REST, layer 3) | Main historical demand data |
| DNR Fire Statistics 2008-present | Local CSV in `Data/` | "Did it hold?" check and county-to-region lookup |
| Daily weather (rain, temperature, humidity, wind) | [Open-Meteo Historical Archive](https://open-meteo.com/) | Weather link, one series per county |
| Fire stations | HIFLD Open, with OpenStreetMap (Overpass) as fallback | Coverage gaps |

## Run it locally

You need Python 3. The site itself is static (HTML, JS, CSS) with Leaflet and Chart.js loaded from a CDN, so there is no build step.

```bash
# 1. Fetch the raw data
python scripts/fetch_fires.py
python scripts/fetch_weather.py
python scripts/fetch_stations.py

# 2. Normalize, join, and compute the analysis files
python scripts/build_data.py
python scripts/analyze.py

# 3. Serve the website
cd web
python -m http.server 8000
```

Then open <http://localhost:8000>.

> Script names follow our build plan. If a name in `scripts/` differs, run the scripts in the same order: fetch, build, analyze.

## Project structure

```
Fire-Watch/
├── Data/        # Source data files (2008+ CSV and other local data)
├── scripts/     # Fetch, normalize, and analysis scripts
├── web/         # Static website (index.html, app.js, styles.css, data/)
├── Data.MD      # Notes on the datasets
└── PLAN.md      # Full build plan
```

## How the allocation works

For each DNR region:

1. Start with its share of historical acres burned (1970-2007).
2. Scale it by how much more fire follows dry springs than normal ones, using this year's spring rain.
3. Add a small boost (10-20%) for regions in the top quartile for coverage gaps.
4. Normalize so all regions add up to 100%.

Every number in the app links to the view that explains it.

## Limitations

- **DNR-protected lands only.** The data does not cover all Washington wildfires, so Fire-Watch never claims statewide totals.
- **Stations are today's stations.** Coverage is measured against historic demand on purpose, but the stations are mostly municipal and district stations, not DNR crew bases.
- **Missing fields.** The 1970-2007 data has no region, escape reason, or control and out times. Region comes from a county lookup.
- **Weather links vary by area.** Where the correlation is weak, the app shows it honestly instead of forcing a story.
- **No individual blame.** The app shows aggregates by county and region only, with no fire names.

## What's next

- Live DNR fire danger and IFPL levels
- A 16-day forecast
- ICS-209 personnel data to convert acres into crew counts
- Real DNR crew base locations
- Review with local emergency management coordinators

## Useful DNR links

- [DNR Fire Dashboard](https://experience.arcgis.com/experience/6cdda73cf6154949a1fae76ccb2900a0)
- [Burn Risk Map](https://fortress.wa.gov/dnr/protection/firedanger/)
- [IFPL levels](https://fortress.wa.gov/dnr/protection/ifpl/)
- [NWCC briefings](http://gacc.nifc.gov/nwcc/)
