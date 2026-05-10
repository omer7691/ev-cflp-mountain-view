# Data Dictionary

All processed data files are derived from publicly available sources.
Raw data is not included due to file size; the notebook downloads and
reproduces everything from scratch automatically.

---

## data/processed/

### Mountain_View_demand_points.csv
Census block groups within Mountain View's incorporated boundary with
ACS-derived vehicle ownership and demand estimates.

| Column | Type | Description |
|--------|------|-------------|
| block_group | string | Census GEOID (12-digit) |
| lat | float | Block group centroid latitude (WGS84) |
| lon | float | Block group centroid longitude (WGS84) |
| population | int | Total population (ACS 2022, B01003) |
| vehicles | float | Estimated total vehicles (ACS 2022, B25044) |
| renter_share | float | Fraction of occupied units that are renter-occupied (ACS 2022, B25003) |
| demand_A | float | EV demand under Scenario A — uniform 10% adoption rate |
| demand_B | float | EV demand under Scenario B — tenure-weighted (renters 30%, owners 5%) |
| demand | float | Active demand used in optimization (= demand_A) |

### Mountain_View_candidate_sites.csv
OSM parking lots within Mountain View used as candidate installation sites.

| Column | Type | Description |
|--------|------|-------------|
| site_id | string | Site identifier (J0–J791) |
| lat | float | Parking lot centroid latitude (WGS84) |
| lon | float | Parking lot centroid longitude (WGS84) |
| c_j | int | Fixed installation cost — uniform $50,000 |

### Mountain_View_existing_stations.csv
Publicly accessible EV charging locations in Mountain View as of May 2026.
Source: U.S. DOE Alternative Fuels Station Locator database.

| Column | Type | Description |
|--------|------|-------------|
| station_name | string | Station name |
| lat | float | Station latitude (WGS84) |
| lon | float | Station longitude (WGS84) |

### Mountain_View_HCVI.csv
Home-Charging Vulnerability Index scores for each block group.

| Column | Type | Description |
|--------|------|-------------|
| block_group | string | Census GEOID |
| lat | float | Centroid latitude |
| lon | float | Centroid longitude |
| HCVI_norm | float | Normalized composite vulnerability score [0, 1] |
| c1_renter | float | Renter share component [0, 1] |
| c2_income | float | Income vulnerability component [0, 1] — inverted normalized median income |
| c3_multifamily | float | Multifamily housing share component [0, 1] |
| c4_zero_vehicle | float | Zero-vehicle household share component [0, 1] |
| high_vulnerability | bool | True if HCVI_norm ≥ 0.767 (top quartile) |

### Mountain_View_site_tiers.csv
Feasibility tier classification for all 792 candidate sites.

| Column | Type | Description |
|--------|------|-------------|
| site_id | string | Site identifier (J0–J791) |
| lat | float | Site centroid latitude |
| lon | float | Site centroid longitude |
| tier | int | Feasibility tier (1=municipal, 2=retail, 3=surface/unknown, 4=underground, 5=private) |

---

## results/

### Mountain_View_CFLP_solution.csv
MILP-optimal 20-station solution under Scenario A (uniform 10% adoption).
Proven optimal in 10.4 seconds. Serves all 79 block groups.

| Column | Type | Description |
|--------|------|-------------|
| site_id | string | Selected station site identifier |
| lat | float | Station latitude |
| lon | float | Station longitude |
| n_assigned | int | Number of block groups assigned |
| stn_demand | float | Total EV demand served (units) |
| avg_dist_km | float | Average assignment distance (km) |
| load_pct | float | Station load as % of capacity (Q=500) |

### Mountain_View_CFLP_solution_B.csv
MILP solution under Scenario B (tenure-weighted demand).
Feasible solution found within 300s time limit. 9 of 79 block groups unserved.
Same columns as Mountain_View_CFLP_solution.csv.

### Mountain_View_heuristic_solution.csv
Coverage-first greedy + simulated annealing heuristic solution.
Serves all 79 block groups. 1.6% worse than optimal. Solved in 55 seconds.
Same columns as Mountain_View_CFLP_solution.csv.

### Mountain_View_scenario_analysis.csv
Full results of the six-scenario stochastic demand analysis including
best-case costs, Scenario A evaluation costs, and regret values.

### Mountain_View_equity_results.csv
Equity-constrained MILP results for varying delta values (0.0, 0.1, 0.2).
All constraints non-binding — no efficiency-equity trade-off exists.

### Mountain_View_road_vs_haversine.csv
Road-network vs haversine distance comparison for all 79 assigned pairs.

| Column | Type | Description |
|--------|------|-------------|
| block_group | string | Census GEOID |
| site_id | string | Assigned station identifier |
| haversine_km | float | Great-circle distance (km) |
| road_km | float | OSMnx road-network shortest path (km) |
| ratio | float | road_km / haversine_km |
| diff_km | float | road_km − haversine_km |

### sensitivity_K.csv / sensitivity_Q.csv / sensitivity_r.csv / sensitivity_adoption.csv
Sensitivity grid results varying station budget K, capacity Q,
proximity radius r, and EV adoption rate respectively.

---

## Data Sources

| Source | Data | Table/File | Access |
|--------|------|------------|--------|
| U.S. Census Bureau | Block group boundaries | TIGER/Line 2023 | Free |
| U.S. Census Bureau | Vehicle ownership | ACS 2022, B25044 | Free API |
| U.S. Census Bureau | Tenure by occupancy | ACS 2022, B25003 | Free API |
| U.S. Census Bureau | Median household income | ACS 2022, B19013 | Free API |
| U.S. Census Bureau | Housing units by type | ACS 2022, B25024 | Free API |
| OpenStreetMap | Parking lot geometries | amenity=parking | Free (osmnx) |
| U.S. DOE AFDC | Existing EV station locations | May 2026 snapshot | Free |
| OpenStreetMap | City boundary | Nominatim geocoder | Free |

**Mountain View city boundary:** GEOID 0649670 (U.S. Census TIGER/Line Places 2023)

