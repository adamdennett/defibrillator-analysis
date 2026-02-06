# Implementation Plan: MCLP Optimization for AED Placement in Brighton & Hove

## Overview

Create a new Quarto document `brighton_mclp.qmd` that implements a Maximal Coverage Location Problem (MCLP) optimization to identify optimal locations for new AEDs in Brighton & Hove.

## What This Analysis Will Do

1. **Use existing Census 2021 workday population density** as demand weights (same source as current analysis)
2. **Extract candidate AED locations from OpenStreetMap** - pharmacies, schools, community centres, libraries, sports centres, GP surgeries, supermarkets, plus existing telephone boxes
3. **Calculate network-based walking distances** using the `dodgr` package with OSM street network
4. **Solve MCLP optimization** using `ompr` + GLPK to find optimal new AED locations that maximise population coverage
5. **Test multiple scenarios**: 100m, 200m, 300m coverage radii × 5, 10, 15, 20 new AEDs
6. **Compare before/after coverage** to quantify improvement from optimised placement

## New R Packages Required

| Package | Purpose |
|---------|---------|
| `ompr` | MIP model formulation |
| `ompr.roi` | ROI bridge for ompr |
| `ROI` | Optimisation infrastructure |
| `ROI.plugin.glpk` | GLPK solver (bundles Windows binary) |
| `dodgr` | Network walking distances |
| `osmdata` | OSM POI extraction |

All packages are on CRAN and work on Windows.

## Document Structure (~600 lines)

### Section 1: Introduction
- Explain MCLP approach and how it extends the priority-ranking analysis

### Section 2: Load Libraries
- All required packages including new optimisation libraries

### Section 3: Load Data (reuse patterns from brighton_analysis.qmd)
- Load AED data from Excel, filter to Brighton & Hove
- Download LSOA boundaries from ONS
- Download Census 2021 workday population density from Nomis

### Section 4: Build Demand Points
- Create population-weighted LSOA centroids as demand points

### Section 5: Extract Candidate Locations from OSM
- Query OSM for pharmacies, schools, community centres, libraries, sports centres, GP surgeries, supermarkets
- Include telephone boxes (same Overpass query as existing analysis)
- Filter out candidates within 50m of existing AEDs

### Section 6: Network Distance Calculation
- Download OSM foot network with `dodgr_streetnet()`
- Compute walking distance matrix: demand points → candidate locations
- Compute walking distance matrix: demand points → existing AEDs
- Cache all results as RDS files (computation is expensive)

### Section 7: Current Coverage Baseline
- Calculate % population within 100m, 200m, 300m of existing AEDs

### Section 8: MCLP Formulation
Mathematical formulation:
- **Decision variables**: y[j] = 1 if new AED at candidate j; z[i] = 1 if demand i is covered
- **Objective**: Maximise sum of (population_weight[i] × z[i])
- **Constraints**:
  - z[i] ≤ sum(y[j]) for all j within radius of i
  - sum(y[j]) = p (place exactly p new AEDs)

### Section 9: Run Optimisation Scenarios
- 3 radii × 4 p-values = 12 scenarios
- Each solves in <1 second with GLPK

### Section 10: Sensitivity Analysis Table
- Compare coverage rates across all scenarios
- Show improvement over baseline

### Section 11: Best Scenario Map (200m radius, 10 new AEDs)
- Interactive leaflet showing optimal new AED locations
- Existing AEDs for context
- LSOA boundaries coloured by workday density

### Section 12: Coverage Comparison Map
- Before/after visualisation with 200m buffer zones
- Blue = current coverage, Green = new additional coverage

### Section 13: Network vs Euclidean Distance Comparison
- Show why network distances matter (typically 1.3-1.5× longer than Euclidean)

### Section 14: Summary & Key Findings
- Quantify coverage improvement
- List optimal new AED locations

## Files to Create/Modify

### New files:
- `brighton_mclp.qmd` - Main analysis document

### Generated at runtime (add to .gitignore):
- `brighton_dodgr_graph.rds` - Cached street network
- `brighton_distance_matrix.rds` - Cached distance matrix
- `brighton_existing_dist.rds` - Cached existing AED distances

### Modify:
- `.gitignore` - Add `*.rds` to ignore cache files

## Expected Outputs

1. **Sensitivity analysis table** showing coverage % for each radius × number of new AEDs combination
2. **Interactive map** of optimal new AED locations
3. **Coverage comparison map** showing before/after
4. **Key metrics**: e.g., "Adding 10 optimally-placed AEDs increases 200m coverage from X% to Y%"

## Technical Notes

- All data loading patterns match existing `brighton_analysis.qmd`
- Distance computation is cached to avoid re-running on each render
- MCLP handles already-covered demand by setting weights to 0 for those points
- Document is standalone (doesn't require running other analysis first)
- Includes required disclaimer from CLAUDE.md about emergency use
