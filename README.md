# EV Charging Station Placement: A Capacitated Facility Location Approach
**Author:** Omer Toledo | **Date:** May 2026 | **Status:** Working paper

## Overview
This repository contains the full reproducible pipeline for optimizing public EV charging station placement in Mountain View, California using a Capacitated Facility Location Problem (CFLP) formulation solved with Google OR-Tools SCIP.

**Key results:**
- Provably optimal 20-station network serving all 79 Mountain View block groups
- Average assignment distance: 0.471 km (haversine); ~0.949 km (road network)
- Coverage-first greedy + simulated annealing heuristic achieves within 1.1% of optimal in 55 seconds
- Existing 33-station network serves 78/79 block groups; one station addition closes the gap
- Tenure-weighted demand (Scenario B) exceeds K=20, Q=500 network capacity; all stations overloaded under queueing model
- No efficiency-equity trade-off: high-vulnerability block groups served at shorter distances
- No stable-core sites robust across all six demand scenarios; 11 moderate-stability sites appear in 3–4 of 6 scenario-optimal solutions
- Queueing analysis (M/M/m Erlang C, m=6 ports): mean utilization 39.7%, mean expected wait 1.45 min under baseline demand
- Minimax-regret formulation attempted; did not converge within 300-second solver budget at this instance scale

## Repository Structure

```
ev-cflp-mountain-view/
├── ev_cflp_mountain_view.ipynb    ← Full pipeline notebook (16 cells)
├── environment.yml                ← Conda environment
├── data/
│   ├── README.md                  ← Data dictionary
│   └── processed/                 ← Processed input data CSVs
└── results/                       ← Solution and analysis output CSVs
```

## Reproducing Results

### Option 1: Local (VS Code or Jupyter) — recommended
```bash
git clone https://github.com/otoledo1/ev-cflp-mountain-view.git
cd ev-cflp-mountain-view
conda env create -f environment.yml
conda activate ev-cflp
python -m ipykernel install --user --name ev-cflp --display-name "EV CFLP"
jupyter notebook ev_cflp_mountain_view.ipynb
```
Open in VS Code with the Jupyter extension and select the **EV CFLP** kernel. Full pipeline runs in under 20 minutes.

### Option 2: Google Colab
1. Upload `ev_cflp_mountain_view.ipynb` to Google Colab
2. Click **Runtime → Run all**
3. Full pipeline completes in under 20 minutes

> **Note:** The notebook loads all processed data directly from this GitHub repository. No Census API calls or shapefile downloads are needed. Comment out the `from google.colab import files` lines in Cells 15 and 16 when running locally.

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
| Solve time (Apple Silicon) | 4.54s | 300s |
| Solve time (Google Colab) | ~21s | 300s |
| MIP gap | 0.00% | 99.48%† |
| Variables | 31,868 | 31,868 |
| Constraints | 31,869 | 31,869 |

†MIP gap inflated by unserved-demand penalty term; true travel cost gap ≤ 31.0%.

## Queueing Model Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| m | 6 | Charging ports per station (baseline) |
| μ | 1.0 | Service rate (sessions/port/hour; 60-min avg Level 2) |
| Public charging share | 25% | Fraction of assigned EVs using public charging regularly |
| Weekly visits | 2 | Average visits per EV per week |
| Peak factor | 3× | Peak hour relative to average hourly demand |

## Pipeline

| Cell | Description |
|------|-------------|
| 1 | Load demand points, candidate sites, existing stations, and HCVI from GitHub |
| 2 | Home-Charging Vulnerability Index (HCVI) — merge and define H |
| 3 | Candidate sites — feasibility tiers |
| 4 | Distance matrix — haversine, 3 km proximity filter |
| 5 | MILP solver — Scenario A (proven optimal) |
| 6 | MILP solver — Scenario B (tenure-weighted demand, feasible) |
| 7 | Heuristic — coverage-first greedy + simulated annealing |
| 8 | Existing network baseline + incremental expansion |
| 9 | Equity analysis — HCVI-stratified metrics + equity-constrained MILP |
| 10 | Road-network distance validation — haversine vs. OSMnx shortest paths |
| 11 | Sensitivity grid — K, Q, r, adoption rate |
| 12 | Stochastic demand analysis — 6 scenarios, regret, CVaR_0.90, site stability |
| 13 | Queueing / wait-time model — M/M/m Erlang C, port sensitivity, Scenario B stress test |
| 14 | Reliability analysis — single-station outage scenarios, redundancy metrics |
| 15 | Minimax-regret analysis — attempted; exceeds solver budget at this instance scale |
| 16 | Interactive solution maps (Folium) |
| 17 | File verification + ZIP download |

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

