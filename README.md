# SMLM aquaporin cluster analysis workflow

This repository contains the Jupyter notebook used for the SMLM analysis in my MSc thesis, (Cryo-Super-Resolution: Establishing a Universal Structural Preservation Method for Nanoscale Imaging, Karolinska Institutet, 2026). It compares the nanoscale organisation of aquaporin across three sample preparation conditions: fixation with 4% PFA, target-specific fixation approaches aka Best Practice (BP), and cryofixation.

The notebook documents the analysis sequence used in the thesis. It is not a general-purpose SMLM software package.

## What the notebook does

The notebook is organised into the following stages:

1. **Data loading and filtering**
   - loads localization tables exported from **Zeiss ELYRA** or **ThunderSTORM** via `locan`
   - pools localization metrics across all files and conditions
   - suggests percentile-based default thresholds (precision, PSF width, intensity, frame)
   - applies the same user-confirmed thresholds to all three conditions

2. **Manual visual QC**
   - displays log-scaled localization heatmaps of the filtered data
   - lets the user keep or reject each dataset

3. **ROI selection and Ripley's H analysis**
   - tiles each dataset into 3 × 3 µm square ROIs and ranks them by localization density
   - the user accepts or rejects candidate ROIs (up to 5 per dataset)
   - computes Ripley's H(r) per ROI (r = 0–500 nm)
   - plots mean ± SEM and median ± IQR curves per condition

4. **DBSCAN clustering**
   - applies DBSCAN to the selected ROIs with user-defined `eps` and `min_samples`
   - calculates ROI-level metrics: number of clusters, fraction of localizations in clusters, mean cluster diameter, localization density
   - saves a summary table

5. **Cluster visualisation**
   - interactive ROI picker (`ipywidgets`)
   - cluster overlays on reconstructed density images, with convex hulls

6. **Spatial statistics and per-cluster metrics**
   - pair correlation function g(r)
   - k-nearest-neighbour distances between cluster centroids (k = 3)
   - per-cluster size, convex hull area, radius, intra-cluster density and localizations per cluster

7. **Condition comparison and figures**
   - violin, box and swarm plots for PFA vs BP vs Cryo
   - pairwise two-sided Mann–Whitney U tests with significance brackets
   - a final figure recreation cell where all styling (colours, font sizes, line widths, DPI) can be changed and all figures regenerated from the computed results

## Expected inputs

Localization tables from:

- **Zeiss ELYRA**
- **ThunderSTORM**

After loading through `locan`, the analysis expects these columns:

- `position_x`
- `position_y`
- `uncertainty`
- `intensity`
- `frame`
- `psf_sigma` **or** `psf_half_width`

Coordinates are assumed to be in nm. If your files differ, adapt the loader in Step 1.

## Interactive steps

The notebook is interactive because it reproduces the analysis decisions made in the thesis. During execution you will be asked to:

- select files for each condition (`PFA`, `BP`, `Cryo`) via a file dialog
- specify whether the files are `ELYRA` or `THUNDERSTORM`
- confirm or edit the suggested filtering thresholds
- approve or reject filtered datasets during QC
- accept or reject candidate ROIs
- enter DBSCAN parameters (`eps` in nm and `min_samples`)

The file dialog uses `tkinter`, so the notebook needs to run locally (not on Colab or a remote server without a display).

## Parameters used in the thesis

| Parameter | Value |
|---|---|
| ROI size | 3000 nm × 3000 nm |
| ROIs per dataset | up to 5 |
| Min. localizations per ROI | 20 |
| Ripley's H max radius | 500 nm |
| DBSCAN `eps` | [VALUE] nm |
| DBSCAN `min_samples` | [VALUE] |
| k (nearest neighbours) | 3 |
| Max. points per ROI (random subsampling) | 5000 (Ripley/DBSCAN), 3000 (g(r)) |

Note: ROIs above the max. point count are randomly subsampled. Ripley's H and g(r) subsampling is not seeded, so results can vary slightly between runs.

## Installation

### Option 1: pip

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Option 2: conda

```bash
conda create -n smlm-aqp python=3.11
conda activate smlm-aqp
pip install -r requirements.txt
```

Then start Jupyter:

```bash
jupyter lab
```

## Output files

Outputs are written to a dated folder on the Desktop (`~/Desktop/YYYYMMDD_glyco_analysis/`). These include:

- ROI summary CSV
- Ripley's H metadata CSV
- DBSCAN summary and statistics CSVs
- k-NN centroid distance CSV
- PNG figures (600 dpi)
- `cluster_overlays/` and `cluster_visualisations/` subfolders

## Acknowledgements

The workflow is adapted from [glyco-PAINT-analysis](https://github.com/bruno-stojcic/glyco-PAINT-analysis) by Bruno Stojcic. [ADD SUPERVISOR / LAB]

## Citation

If you use this code, please cite:

> [YOUR NAME] ([YEAR]). *[THESIS TITLE]*. MSc thesis, [UNIVERSITY].

## License

MIT
