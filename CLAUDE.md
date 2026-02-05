# CLAUDE.md

## Project Overview

UK Automated External Defibrillator (AED) location data repository. Contains geospatial point data for ~116,000 defibrillators sourced from The Circuit (defibfinder.uk). Used for spatial analysis and public health research.

## Project Structure

```
defibrillator/
└── defibrillator_data.xlsx   # Main data file (~17 MB)
    ├── Sheet: "ToR"                  # Terms of Reference / licensing
    ├── Sheet: "Data guide"           # Data dictionary / column definitions
    └── Sheet: "data_extract_2026-02-02"  # AED records (116,027 rows × 17 columns)
```

## Data Columns

- `unique_identifier` — Unique AED ID
- `address_line1`, `address_line2`, `address_line3` — Address fields
- `address_city`, `address_county`, `address_post_code` — Location details
- `ladnm` — Local Authority District name
- `lsoa21` / `lsoa21nm` — Lower Super Output Area code / name
- `defibrillators_availability` — 24/7 or varied access
- `defibrillators_access_type` — Public or restricted
- `lat` / `long` — Geographic coordinates (WGS84)
- `location_name` — Descriptive location name
- `country` — Country within the UK
- `county` — County

## Language and Tools

- **Primary language: R**
- Use `readxl::read_excel()` to load data (specify `sheet = "data_extract_2026-02-02"` for the data)
- Use `sf` package for spatial operations (convert lat/long to `sf` point geometry with CRS 4326)
- Use `tmap` or `ggplot2 + geom_sf()` for mapping
- Use Quarto (`.qmd`) for reproducible analysis documents
- Coordinate reference system: WGS84 (EPSG:4326) for raw data; transform to EPSG:27700 (British National Grid) for UK-specific spatial analysis

## Data Usage Restrictions

- **Non-commercial use only**
- Cannot be used to create public-facing apps or maps
- Public websites using this data must link to https://www.defibfinder.uk/
- Required disclaimer for any public use: "Is this an emergency? This website is used to locate defibrillators, but it is not intended for use in an emergency. If you require urgent medical assistance, call 999 now."

## Common R Patterns

```r
# Load data
library(readxl)
defib <- read_excel("defibrillator_data.xlsx", sheet = "data_extract_2026-02-02")

# Convert to spatial object
library(sf)
defib_sf <- st_as_sf(defib, coords = c("long", "lat"), crs = 4326)

# Transform to British National Grid
defib_bng <- st_transform(defib_sf, 27700)
```

## Git

- Branch: `master` (main branch is `main`)
- `.gitignore` should include: `.Rproj.user`, `.Rhistory`, `.RData`, `.Ruserdata`
- The `.xlsx` file is large (~17 MB); consider whether it should be tracked in git or managed via `.gitignore` / Git LFS
