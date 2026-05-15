# EV Charging Station Placement: A Capacitated Facility Location Approach
**Author:** Omer Toledo | **Date:** May 2026 | **Status:** Working paper

## Overview
This repository contains the full reproducible pipeline for optimizing public EV charging station placement in Mountain View, California using a Capacitated Facility Location Problem (CFLP) formulation solved with Google OR-Tools SCIP.

**Key results:**
- Provably optimal 20-station network serving all 79 Mountain View block groups
- Average assignment distance: 0.471 km (haversine); ~0.949 km (road network)
- Coverage-first greedy + simulated annealing heuristic achieves within 2.6% of optimal in 55 seconds
- Existing 33-station network serves 78/79 block groups; one station addition closes the gap
- Tenure-weighted demand (Scenario B) exceeds K=20, Q=500 network capacity
- No efficiency-equity trade-off: high-vulnerability block groups served at shorter distances
- No stable-core sites robust across all six demand scenarios; 11 moderate-stability sites appear in 3–4 of 6 scenario-optimal solutions
- Minimax-regret formulation attempted; did not converge within 300-second solver budget at this instance scale

## Repository Structure

```
ev-cflp-mountain-view/
├── ev_cflp_mountain_view.ipynb    ← Full pipeline notebook (15 cells)
├── environment.yml                ← Conda environment
├── data/
│   ├── README.md                  ← Data dictionary
│   └── processed/                 ← Processed input data CSVs
└── results/                       ← Solution and analysis output CSVs
```

## Reproducing Results

### Option 1: Google Colab (recommended)
1. Upload `ev_cflp_mountain_view.ipynb` to Google Colab
2. Click **Runtime → Run all**
3. Full pipeline completes in under 20 minutes

> **Note:** The notebook loads all processed data directly from this GitHub repository. An internet connection is required. No Census API calls or shapefile downloads are needed.

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
| Solve time | 21.4s | 300s |
| MIP gap | 0.00% | 99.46%† |
| Variables | 31,868 | 31,868 |
| Constraints | 31,869 | 31,869 |

†MIP gap inflated by unserved-demand penalty term; true travel cost gap ≤ 24.4%.

## Pipeline

| Cell | Description |
|------|-------------|
| 1 | Load demand points, candidate sites, existing stations, and HCVI from GitHub |
| 2 | Home-Charging Vulnerability Index (HCVI) — merge and define H |
| 3 | Candidate sites — feasibility tiers |
| 4 | Distance matrix — haversine, 3 km proximity filter |
| 5 | MILP solver — Scenario A (proven optimal, 21.4s) |
| 6 | MILP solver — Scenario B (tenure-weighted demand, feasible) |
| 7 | Heuristic — coverage-first greedy + simulated annealing |
| 8 | Existing network baseline + incremental expansion |
| 9 | Equity analysis — HCVI-stratified metrics + equity-constrained MILP |
| 10 | Road-network distance validation — haversine vs. OSMnx shortest paths |
| 11 | Sensitivity grid — K, Q, r, adoption rate |
| 12 | Stochastic demand analysis — 6 scenarios, regret, CVaR_0.90, site stability |
| 13 | Minimax-regret analysis — attempted; exceeds solver budget at this instance scale |
| 14 | Interactive solution maps (Folium) |
| 15 | File verification + ZIP download |

## Data Sources

| Source | Data | Access |
|--------|------|--------|
| U.S. Census Bureau | Block group boundaries (TIGER/Line 2023) | Free |
| U.S. Census Bureau | ACS 2022 (B25044, B25003, B19013, B25024) | Free API |
| OpenStreetMap | Parking lot geometries | Free (osmnx) |
| U.S. DOE AFDC | Existing EV station locations | Free |

## Citation
If you use this code or data, please cite:

Toledo, O. (2026). Electric Vehicle Charging Station Placement: A Capacitated Facility Location Approach. GitHub: https://github.com/otoledo1/ev-cflp-mountain-view
