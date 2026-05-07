# CA Wildfire Ignition ML

Code accompanying the paper:

> **Time-resolved moving-window machine learning modeling for human-caused wildfire ignitions from 2001 to 2020 in California**
> Zhijiong Cao, Fangshu Ye, Yongkang Xue, Thomas Gillespie
> *Environmental Research Letters* (2026)
> DOI: [10.1088/1748-9326/ae692d](https://doi.org/10.1088/1748-9326/ae692d)

## Overview

This repository contains the Jupyter notebooks used to build the dataset, train the model, and produce the figures in the paper. The pipeline:

- Builds a monthly, gridded dataset over California for water years 2001–2020 by combining weather, snow, vegetation, topography, road, and power infrastructure.
- Uses the FPA-FOD (https://doi.org/10.2737/RDS-2013-0009.6) wildfire occurrence database to label monthly human-caused ignitions per grid cell.
- Trains an XGBoost classifier in a **6-year moving-window** scheme (the model for year *t* is trained on years *t*−5 … *t*) so that the predictor–response relationship can change with time.
- Evaluates the model with AUC-ROC and AUC-PRG across space and time and uses SHAP for feature attribution.
- Evaluates the model performance on capturing broad spatial distribution, seasonal cycle, and interannual variability.
- Includes a climatology experiment to assess the contribution of interannual weather variability.

## Repository structure

```
notebooks/   Jupyter notebooks — numbered in execution order
```

Notebooks are named `<stage> <step> <description>.ipynb` and are intended to be run in order.

| Stage | Notebooks | What it does |
|---|---|---|
| **00 — Grid match** | `00 Fire/Line/Road/Slope/Veg_Weather_Grid_Match.ipynb` | Reproject and match each raw layer onto the common weather grid. |
| **01 — Data clean** | `01 01 … 01 05` | Clean and harmonize the raw inputs. |
| **02 — Feature merge** | `02 01 … 02 06` | Join cleaned inputs into per-cell feature tables; fill wind-direction NAs; build static (time-invariant) features; add OpenStreetMap power infrastructures. |
| **03 — Aggregate & label** | `03 01 … 03 04` | Aggregate weather to monthly, build monthly fire labels from FPA-FOD, assemble the modeling table, and prepare a calibration split. |
| **04 — Model** | `04 01` 6-year moving-window XGBoost (paper model); `04 02` X-year window-length sensitivity / power test. |
| **05 — Evaluate** | `05 01` metrics + SHAP feature importance; `05 02` spatio-temporal performance maps. |
| **06 — Climate anomaly** | `06 Climate_Anomaly_Test.ipynb` | Monthly weather inputs during testing were replaced with their 2001–2020 multi-year mean. |

## Data

The notebooks expect raw data on a local disk; this repository does **not** redistribute the source datasets. Key sources:

- **Fire ignitions:** USFS Fire Program Analysis Fire-Occurrence Database (FPA-FOD), human-caused ignitions only.
- **Weather:** GRIDMET (https://doi.org/10.1002/joc.3413).
- **Snow:** SWE (https://doi.org/10.5067/0GGPB220EX6A).
- **Vegetation:** GLASS LAI (https://doi.org/10.1175/JCLI4054.1); CALVEG (https://data.fs.usda.gov/geodata/edw/datasets.php?xmlKeyword=calveg).
- **Topography:** DEM-derived slope (https://srtm.csi.cgiar.org).
- **Infrastructure:** Road (10.1088/1748-9326/aabd42); Power Infrastructure (https://www.openstreetmap.org/export#map=5/51.50/-0.10).

Each notebook has a configuration cell near the top (`PROJECT_ROOT = ...`) that you must edit to point at your local data paths. Intermediate outputs are written as Parquet under `Clean_Data/` and `Summary_Data/`.

## Environment

Tested with Python 3.10+. The notebooks use:

```
cartopy, dbfread, geopandas, geopy, matplotlib, numpy, pandas,
pyarrow, pyproj, scipy, seaborn, shap, shapely, scikit-learn,
tqdm, xarray, xgboost
```

Quick setup with conda (recommended for cartopy/geopandas):

```bash
conda create -n ca_ignition python=3.10 -c conda-forge \
  cartopy geopandas pyproj shapely xarray pyarrow \
  pandas numpy scipy scikit-learn matplotlib seaborn tqdm
conda activate ca_ignition
pip install xgboost shap dbfread geopy
```

## How to reproduce

1. Acquire the source datasets listed above and place them under a single project root.
2. In each notebook, edit the configuration cell (`PROJECT_ROOT`, etc.) to match your layout.
3. Run the notebooks in numerical order (00 → 06). Stages are independent at the file level but later stages consume the Parquet outputs of earlier ones.

## Citation

If you use this code, please cite:

```bibtex
@article{Cao2026CAIgnition,
  title   = {Time-resolved moving-window machine learning modeling for human-caused wildfire ignitions from 2001 to 2020 in California},
  author  = {Cao, Zhijiong and Ye, Fangshu and Xue, Yongkang and Gillespie, Thomas},
  journal = {Environmental Research Letters},
  year    = {2026},
  doi     = {10.1088/1748-9326/ae692d}
}
```

## Contact

Zhijiong Cao — Department of Geography, UCLA — czj9763@g.ucla.edu

## License

Code released for academic use accompanying the paper above. See `LICENSE` (to be added) for details.
