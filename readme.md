# Empty-Passage Optimization 🚚✨

**Optymalizacja pustych przebiegów dla tras transportowych**

A toolkit of Jupyter notebooks and scripts for **detecting**,
**analyzing**, and **minimizing** empty (deadhead) kilometers between
transport routes.\
Ideal for logistics teams, analysts, and optimization engineers working
with route chaining, cost reduction, and fleet efficiency.

> Reduce unnecessary driving. Improve fleet utilization. Evaluate solver
> performance.\
> A practical, data-driven workflow for real transport datasets.

### 📄 Documentation

For a comprehensive overview, please consult [the documentation](./documentation/).

------------------------------------------------------------------------

## 🎯 What Problem Does This Solve?

In transport operations, trucks often end one route in City A and start
the next in City B --- creating **puste przebiegi** (deadhead km).\
When repeated across dozens of tractors and thousands of trips, these
empty kilometers:

- waste fuel, time, and driver hours
- raise operational costs
- reduce overall efficiency

**This project builds an optimization pipeline that:**

- detects realistic candidate empty connections between full routes
- evaluates feasibility based on time windows and distances
- generates solutions using multiple solvers (CBC, GLPK, CPLEX)
- minimizes total empty kilometers across the fleet

A complete workflow from raw data → distance matrices → sampling →
optimization → evaluation.

------------------------------------------------------------------------

## 🔧 Key Capabilities

- 🧹 **Data Cleaning & Normalization**\
    Parse timestamps, unify formats, fix corrupted rows, prepare clean
    input.

- 🗺️ **City Distance & Travel Time Matrices**\
    Build matrices using precomputed data or external routing APIs.

- 🔍 **Detection of Candidate Empty Trips**\
    Automatically pair feasible route-to-route combinations based on
    timing and geography.

- 🧠 **MILP Optimization**\
    Minimize empty km with support for several solvers:

    - CBC (default, open-source)
    - GLPK
    - IBM CPLEX (optional, fastest if available)

- 📊 **Evaluation & Solver Benchmarking**\
    Compare objective values, runtimes, feasibility, and number of empty
    trips selected.

- 🧪 **Extensible Notebooks**\
    Each step is isolated in Jupyter notebooks for clarity and fast
    experimentation.

------------------------------------------------------------------------

## 📦 Project Structure

    .
    ├── 1.1 clear-data.ipynb                      # Data cleaning & normalization
    ├── 3.1 sampling_de.ipynb                    # Utility sampling routines
    ├── 3.2 sampling_de-cities_distances.ipynb   # City distance/time matrix generation
    ├── 3.3 sampling_de-empty-runs-matrix.ipynb  # Extract candidate empty-run edges
    ├── 4. sampling_solver_optimalization.ipynb  # Optimization with multiple solvers
    └── data/                                    # Raw & processed datasets

------------------------------------------------------------------------

## 📁 Important Data Files

- `data/cities_latitude_longitude.csv` --- coordinates used for
    distance calculations
- `data/output/sample_de/sample_de_cities_distance_matrix.xlsx` --- km
    matrix
- `data/output/sample_de/sample_de_cities_time_matrix.xlsx` --- hours
    matrix
- `data/output/sample_de/sample_dane_puste_przebiegi.xlsx` --- example
    full/empty route data

------------------------------------------------------------------------

## 🧰 Environment Setup

**Recommended Python:** 3.10--3.11\
**Core packages:** `pandas`, `numpy`, `geopy`, `pulp`, `openpyxl`,
`requests`, `tqdm`\
**Optional:**

- GLPK executable (`glpsol`)
- IBM CPLEX + Python API (`cplex`, `docplex`)

### Quick Setup (PowerShell)

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -U pip
pip install pandas numpy openpyxl requests geopy tqdm pulp
# Optional:
# pip install docplex cplex
```

------------------------------------------------------------------------

## 🚀 Workflow Overview

### 0️⃣ Data Collection & Preprocessing Constraints (DE-focused sample)

1. Cleaning of city names  
2. Filling missing data (e.g., geographic coordinates)  
3. Standardizing formats (dates, timestamps)  
4. Selecting a **1‑week sample** limited to operations within or involving **Germany (DE)**

#### Geographical & Time Filtering Rules

- Load/unload must fall within: **2025‑09‑08 → 2025‑09‑14**  
- Routes must involve Germany (`DE`)  
- For routes partially in Germany, **only the German segment is kept**

Example trimming:

| idx | load | unload | status |
|-----|------|--------|--------|
| 0 | PL | CZ | ❌ removed |
| 1 | CZ | DE | ✅ start DE block |
| 2 | DE | NL | ✅ connected |
| 3 | NL | BE | ⚪ neutral (between DE segments) |
| 4 | BE | DE | ✅ connected |
| 5 | DE | FR | ✅ connected |
| 6 | FR | IT | ❌ removed |

### 1️⃣ Clean Input Data

Run `1.1 clear-data.ipynb`

- parse timestamps
- normalize records
- remove invalid entries

### 2️⃣ Build Distance & Time Matrices

Run `3.2 sampling_de-cities_distances.ipynb`\
You can:

- use included matrices, or
- generate new ones via OpenRouteService API (supports multiple rotating
keys)

### 3️⃣ Detect Candidate Empty Passages

Run `3.3 sampling_de-empty-runs-matrix.ipynb`\
Outputs a matrix of route-pairs that could be connected by empty trips.

### 4️⃣ Optimization

Run `4. sampling_solver_optimalization.ipynb`\
The solver will:

- construct feasible empty edges
- validate time windows
- minimize total empty km with CBC/GLPK/CPLEX
- produce final assignments

------------------------------------------------------------------------

## 📊 Evaluation & Benchmarking

**Primary metric:**

- Total empty km after optimization

**Secondary metrics:**

- solver runtime
- solver status (Optimal / Feasible / Infeasible)
- number of selected empty trips

**Suggested workflow:**

1. Compute original baseline (no optimization)
2. Run CBC for full solution
3. Compare GLPK and CPLEX performance
4. Export results for reporting

------------------------------------------------------------------------

## 📝 Practical Notes & Tips

- API distance values are converted to **km**; durations to
    **hours**.
- ORS API keys rotate automatically to avoid quotas.
- Big-M constraints depend on unified timestamps --- ensure no
    timezone inconsistencies.
- Filters in the optimization notebook (max empty km / max idle time)
    strongly affect runtime and solution quality.

------------------------------------------------------------------------

## 🐞 Troubleshooting

- **Zeros in distance matrix?**\
    Check if the coordinates file contains valid lat/lon pairs.

- **GLPK/CPLEX not found?**\
    Use CBC or update correct binary path in notebook.

- **Optimization too slow?**\
    Limit candidate empty edges via distance/time thresholds.

------------------------------------------------------------------------

## 🌱 Possible Extensions

- Automated solver benchmarks exported to CSV
- Persistent caching for API matrix results
- Adding cost-based objective (fuel, driver hours, penalties)
- Adding multi-day, multi-tractor schedule linking

------------------------------------------------------------------------

## 🤝 Contributing

Contributions, fixes, and ideas are welcome!

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

------------------------------------------------------------------------

## 📝 License

Open-source. See the `LICENSE` file for details.
