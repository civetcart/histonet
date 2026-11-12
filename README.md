# HISTONET

**Histone-based Temporal Network Analysis**

Code and results for the thesis _HISTONET: A Machine Learning Framework for Classifying Hub Gene Dynamics in Drosophila melanogaster using Temporal Histone Mark Profiles_.

We ask one question: can histone marks predict whether a gene stays a 3D chromatin hub across early development? We build a Hi-C contact network for each of four embryonic stages, call hub genes in each stage, and label every hub gene as **Stable** (hub in 3–4 stages) or **Transient** (hub in 1–2 stages). Then we train Random Forest and XGBoost models to predict that label from histone ChIP-seq signal alone.

## Pipeline

| Step | Notebook                       | What it does                                                                                                                                                                                             | Output                     |
| ---- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| 1    | `01_gene-coord.ipynb`          | Collapses GTF transcripts into one row per gene. Keeps main chromosomes only and drops genes that map to more than one locus.                                                                            | `gene_coordinates.csv`     |
| 2    | `02_histone_feat.ipynb`        | Reads 32 WIG tracks (8 marks × 4 stages). Takes mean signal in a ±1 kb window around each TSS. No coverage is stored as NaN, not 0.                                                                      | `histone_features.csv`     |
| 3    | `03_compute_eigenvector.ipynb` | Builds one weighted graph per chromosome per stage from long-range (≥ 500 kb), observed/expected Hi-C contacts. Computes eigenvector centrality on the largest connected component and maps it to genes. | `eigenvector_features.csv` |
| 4    | `04_build_labels.ipynb`        | Calls a gene a hub if its centrality is in the top 20% of its chromosome for that stage. Counts hub stages and assigns Stable / Transient. Never-hub genes are dropped.                                  | `modeling_table.csv`       |
| 5    | `05_train_model.ipynb`         | Trains Random Forest and XGBoost on the 32 histone features. Reports hold-out and 5-fold CV metrics, a hub-cutoff sensitivity check, confusion matrices and SHAP importance.                             | `results/`                 |

Run the notebooks in order. Each one reads the previous step's file from `data/processed/`.

### Key settings

| Setting                  | Value                                                          |
| ------------------------ | -------------------------------------------------------------- |
| Genome build             | dm3                                                            |
| Stages                   | C1-8, C9-13, C14E, C14L                                        |
| Chromosomes              | chr2L, chr2R, chr3L, chr3R, chr4, chrX                         |
| Hi-C bin size            | 10 kb                                                          |
| Minimum contact distance | 500 kb                                                         |
| Histone marks            | H3, H3K4me1, H3K4me3, H3K9ac, H3K18ac, H3K27ac, H4K5ac, H4K8ac |
| TSS window               | ±1 kb                                                          |
| Hub cutoff               | top 20% centrality, per chromosome, per stage                  |
| Labels                   | Stable = hub in 3–4 stages, Transient = hub in 1–2 stages      |
| Train / test split       | 80 / 20, stratified, seed 42                                   |
| Missing values           | median imputation, fit on training data only                   |
| Trees                    | 400 (both models)                                              |

Centrality is used **only** to make the labels. It is never a model feature.

## Results

Modeling table: 4,509 genes (2,865 Stable, 1,644 Transient).

**Hold-out test set (902 genes)** — `results/results.csv`

| Model         | ROC-AUC | Balanced accuracy | Macro F1 |
| ------------- | ------- | ----------------- | -------- |
| Random Forest | 0.708   | 0.605             | 0.603    |
| XGBoost       | 0.680   | 0.602             | 0.603    |

**5-fold stratified CV** — `results/results_cv.csv`

| Model         | ROC-AUC       | Balanced accuracy | Macro F1      |
| ------------- | ------------- | ----------------- | ------------- |
| Random Forest | 0.702 ± 0.018 | 0.597 ± 0.010     | 0.592 ± 0.012 |
| XGBoost       | 0.691 ± 0.031 | 0.618 ± 0.025     | 0.621 ± 0.028 |

**Hub cutoff sensitivity (Random Forest, 5-fold CV)** — `results/hub_cutoff_sensitivity.csv`

| Cutoff | Genes | % Stable | ROC-AUC | Balanced accuracy |
| ------ | ----- | -------- | ------- | ----------------- |
| 0.80   | 4,509 | 63.5     | 0.702   | 0.597             |
| 0.85   | 3,592 | 57.8     | 0.662   | 0.599             |
| 0.90   | 2,545 | 51.0     | 0.671   | 0.618             |
| 0.95   | 1,385 | 44.6     | 0.712   | 0.635             |

**Top features (SHAP).** Both models rank late-stage active marks highest. H4K8ac at C14E is first for both, followed by H3K4me3 (C14E, C14L), H3K9ac (C14L) and H3K18ac / H3K27ac (C14E). Full rankings are in `results/figures/shap/`.

## Repository structure

```
histonet/
├── data/
│   ├── raw/                 # not tracked (too large), see data/README.md
│   │   ├── gene_annotation/
│   │   ├── hic_contacts/
│   │   └── histone_wig/
│   ├── processed/           # tracked, outputs of notebooks 01-04
│   └── README.md            # where to download the raw data
├── notebooks/               # 01-05, run in order
├── results/
│   ├── figures/
│   │   ├── *_confusion_matrix.png
│   │   └── shap/            # SHAP bar plots + importance tables
│   ├── results.csv          # hold-out metrics
│   ├── results_cv.csv       # 5-fold CV metrics
│   └── hub_cutoff_sensitivity.csv
├── requirements.txt
└── README.md
```

## Setup

We used Python 3.10.

```bash
git clone https://github.com/<your-username>/histonet.git
cd histonet
python -m venv .venv
# Windows: .venv\Scripts\activate   |   macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

**Only want to rerun the models?** The processed tables are already in `data/processed/`, so you can open `05_train_model.ipynb` and run it directly.

**Want to run the full pipeline?** Download the raw files listed in [`data/README.md`](data/README.md), place them in `data/raw/`, then run notebooks 01 to 05.

## Data sources

- **Hi-C:** Ogiyama Y, Schuettengruber B, Papadopoulos GL, Chang J-M, Cavalli G. (2018). Polycomb-dependent chromatin looping contributes to gene silencing during _Drosophila_ development. _Molecular Cell_ 71(1):73–88. GEO: [GSE103625](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE103625)
- **Histone ChIP-seq:** Li X-Y, Harrison MM, Villalta JE, Kaplan T, Eisen MB. (2014). Establishment of regions of genomic activity during the _Drosophila_ maternal to zygotic transition. _eLife_ 3:e03737. GEO: [GSE58935](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE58935)
- **Gene annotation:** UCSC dm3 refGene GTF.
