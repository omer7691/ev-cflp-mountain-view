# EV Charging Station Placement: A Capacitated Facility Location Approach

**Author:** Omer Toledo
**Date:** May 2026
**Status:** Working paper

## Overview

This repository contains the full reproducible pipeline for optimizing public
EV charging station placement in Mountain View, California using a Capacitated
Facility Location Problem (CFLP) formulation solved with Google OR-Tools SCIP.

**Key results:**
- Provably optimal 20-station network serving all 79 Mountain View block groups
- Average assignment distance: 0.471 km (haversine); ~0.949 km (road network)
- Coverage-first greedy + simulated annealing heuristic achieves within 1.6% of optimal in 55 seconds
- Existing 33-station network serves 78/79 block groups; one station addition closes the gap
- Tenure-weighted demand (Scenario B) exceeds K=20, Q=500 network capacity
- No efficiency-equity trade-off: high-vulnerability block groups served at shorter distances
- Four stable-core sites robust across six demand scenarios (J27, J444, J610, J642)

## Repository Structure

```
ev-cflp-mountain-view/
├── ev_cflp_mountain_view.ipynb    ← Full pipeline notebook
├── environment.yml                ← Conda environment
├── data/
│   ├── README.md                  ← Data dictionary
│   └── processed/                 ← Processed input data CSVs
└── results/                       ← Solution and analysis output CSVs
```

## Reproducing Results

### Option 1: Google Colab (recommended)

1. Upload `ev_cflp_mountain_view.ipynb` to [Google Colab](https://colab.research.google.com)
2. Click **Runtime → Run all**
3. Full pipeline completes in under 20 minutes

> **Note:** The notebook downloads Census shapefiles (~50MB) and the Mountain
> View road network automatically. An internet connection is required for
> OSM and Census API calls.

### Option 2: Local environment

```bash
git clone https://github.com/otoledo1/ev-cflp-mountain-view.git
cd ev-cflp-mountain-view
conda env create -f environment.yml
conda activate ev-cflp
jupyter notebook ev_cflp_mountain_view.ipynb
```

## Model Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| K | 20 | Stations to build |
| Q | 500 | Capacity per station (EV units) |
| r | 3.0 km | Proximity radius |
| Adoption rate | 10% | EV stock fraction (Scenario A) |
| Fixed cost | $50,000 | Per-station installation cost (uniform) |

## Solver

| Metric | Scenario A | Scenario B |
|--------|------------|------------|
| Solver | OR-Tools SCIP v9.x | OR-Tools SCIP v9.x |
| Status | Optimal | Feasible (time limit) |
| Solve time | 10.4s | 300s |
| MIP gap | 0.00% | 99.46%† |
| Variables | 31,868 | 31,868 |
| Constraints | 31,869 | 31,869 |

†MIP gap inflated by unserved-demand penalty term; true travel cost gap ≤ 24.4%.

## Data Sources

| Source | Data | Access |
|--------|------|--------|
| U.S. Census Bureau | Block group boundaries (TIGER/Line 2023) | Free |
| U.S. Census Bureau | ACS 2022 (B25044, B25003, B19013, B25024) | Free API |
| OpenStreetMap | Parking lot geometries | Free (osmnx) |
| U.S. DOE AFDC | Existing EV station locations | Free |

## Citation

If you use this code or data, please cite:

Toledo, O. (2026). Electric Vehicle Charging Station Placement:
A Capacitated Facility Location Approach. 
GitHub: https://github.com/otoledo1/ev-cflp-mountain-view


