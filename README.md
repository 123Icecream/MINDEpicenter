# Structural Epicenters of Autism Spectrum Disorder: A Multi-Cohort Normative Modeling Study

This repository contains the analysis code for identifying structural epicenters of autism spectrum disorder (ASD) using normative modeling and MIND (Morphometry INformed by functional Dynamics) connectivity across two independent cohorts: **ABIDE-II** and **CABIC**.

## Overview

The main pipeline consists of three stages:

```
features/   →  Cortical morphometric feature extraction (VolSubGroup)
epicenter/  →  Normative modeling, subtyping, and epicenter identification
figures/    →  Figure generation (Fig1–Fig5)
```

> **Note on preprocessing**: participant filtering and quality control (MIND network completeness, ICV and Euler-number QC) were performed separately; the resulting subject information is provided in `data/` (see *Input data* below) with an `aparc` column pointing to each subject's `MIND_network_aparc.csv`.

1. **Feature extraction** — Parse FreeSurfer `*.aparc.stats` files (Desikan-Killiany 68 atlas) to obtain eight cortical morphometric features (e.g., GrayVol, SurfArea, ThickAvg).
2. **Epicenter analysis** — Build Gaussian Process Regression (GPR) normative models on TDC controls, compute Z-score deviations, cluster ASD participants into two biotypes (Subtype L: volume reductions; Subtype H: volume increases), and identify individual-level epicenters via minimum goodness-of-fit (GOF) between Z-score deviation patterns and MIND connectivity fingerprints. Core epicenters are defined via a rank-based integration across cohorts.
3. **Figures** — Generate the main and supplementary figures reported in the manuscript.

## Cohorts

| Cohort | Description | Site |
|--------|-------------|------|
| **ABIDE-II** | Autism Brain Imaging Data Exchange II | Multi-site, international |
| **CABIC** | Chinese Autism Brain Imaging Consortium | Multi-site, China |

## Installation

```bash
conda create -n mind_epicenter python=3.9
conda activate mind_epicenter
pip install -r requirements.txt
```

## Usage

Each stage is implemented as a self-contained Jupyter notebook. The expected data layout is described below.

### Data Layout

```
<DATA_ROOT>/
├── ABIDE-II/                    # ABIDE-II BIDS root (or site folders)
│   ├── ABIDEII_Composite_Phenotypic.csv
│   └── <SITE>/
│       └── derivatives/
│           ├── smri_prep/<sub>/MIND_network_*.csv
│           └── freesurfer/<sub>/stats/{lh,rh}.aparc.stats
└── CABIC/
    ├── CABIC_Subjects_info.xls
    └── <SITE>/
        └── T1Img/
            ├── smri_prep/<sub>/MIND_network_*.csv
            └── recon/<sub>/stats/{lh,rh}.aparc.stats
```

### Pipeline Order

Run the notebooks in the following order:

1. `features/VolSubGroup.ipynb`
2. `epicenter/Epicenter.ipynb` (ABIDE-II)
3. `epicenter/EpicenterCABIC.ipynb` (CABIC)
4. `epicenter/UnifyEpicenter.ipynb` (cross-cohort integration)
5. `figures/Fig1.ipynb` … `figures/Fig5.ipynb`

### Input data (`data/`)

| File | Description |
|------|-------------|
| `ABIDE2_AGE.xlsx` | ABIDE-II subject info (demographics, site, ICV, `aparc` MIND-network path) |
| `CABIC_AGE.xlsx` | CABIC subject info (demographics, site, ICV, `aparc` MIND-network path) |
| `ABIDE2_AGE_Sub.xlsx` | ABIDE-II subject info with K-means subtype labels (used for epicenter seeding) |
| `CABIC_AGE_Sub.xlsx` | CABIC subject info with K-means subtype labels (used for epicenter seeding) |
| `ABIDE2_all_aparc_features.csv` | ABIDE-II cortical morphometric features (parsed from FreeSurfer `aparc.stats`) |
| `CABIC_all_aparc_features.csv` | CABIC cortical morphometric features (parsed from FreeSurfer `aparc.stats`) |
| `NormativeChart_rhcACC_Trajectory.csv` | Precomputed GPR normative chart trajectory for rh-cACC (Fig1) |
| `allen_gene_expression_dk68.csv` | DK68 region-wise Allen Human Brain Atlas gene expression (cache; Fig5 cell 3 regenerates it from the external Allen data) |
| `SFARI-Gene_genes_05-01-2026release_06-07-2026export.csv` | SFARI Gene reference list (ASD risk genes) used in the gene-enrichment analyses |
| `MIND/ABIDE2/` | Per-subject `MIND_network_aparc.csv` (68×68) matrices for all 739 ABIDE-II participants |
| `MIND/CABIC/` | Per-subject `MIND_network_aparc.csv` (68×68) matrices for all 2226 CABIC participants |

> The `aparc` column in the two `*_AGE.xlsx` files points to these per-subject MIND matrices via the relative paths `data/MIND/<COHORT>/<SUBID>.csv`.
>
> **Convention**: `data/` holds all inputs; `output/` holds all generated intermediates (Z-scores, TDC-average MIND, GOF scores, epicenter frequency maps, subtype labels, Table S1).

## License

See the `LICENSE` file.
