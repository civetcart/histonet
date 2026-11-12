# Data

`data/raw/` is not tracked in git because the files are too large (each Hi-C contact file is about 390 MB, and most WIG files are 25–125 MB). `data/processed/` is tracked, so notebook 05 runs without the raw data.

## Raw files

Put the files in these folders with these exact names. The notebooks look for them by name.

### `raw/gene_annotation/`

| File | Source |
|---|---|
| `dm3.refGene.gtf` | UCSC Genome Browser, dm3 refGene annotation |

### `raw/hic_contacts/`

Hi-C from Ogiyama et al. (2018), GEO [GSE103625](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE103625). 10 kb bins, one pair of files per stage.

| Stage | Files |
|---|---|
| C1-8 | `C1-8_10000.bins`, `C1-8_10000.n_contact` |
| C9-13 | `C9-13_10000.bins`, `C9-13_10000.n_contact` |
| C14E | `C14E_10000.bins`, `C14E_10000.n_contact` |
| C14L | `C14L_10000.bins`, `C14L_10000.n_contact` |

`.bins` must have the columns `cbin`, `chr`, `from.coord`, `to.coord`.
`.n_contact` must have the columns `cbin1`, `cbin2`, `observed_count`.

### `raw/histone_wig/`

Histone ChIP-seq from Li et al. (2014), GEO [GSE58935](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE58935). Files are named `Dmel-<mark>-<cycle>.wig`.

Cycle-to-stage mapping used in the code:

| WIG cycle | Stage |
|---|---|
| c8 | C1-8 |
| c12 | C9-13 |
| c14a | C14E |
| c14c | C14L |

Marks used: H3, H3K4me1, H3K4me3, H3K9ac, H3K18ac, H3K27ac, H4K5ac, H4K8ac (8 marks × 4 cycles = 32 files).

H3K27me3 and H3K36me3 WIGs are also in the folder but are not used, because they only exist for 3 of the 4 cycles.

## Processed files

| File | Made by | Rows | Contents |
|---|---|---|---|
| `gene_coordinates.csv` | 01 | 16,846 | gene name, chromosome, strand, transcript start/end |
| `histone_features.csv` | 02 | 16,846 | 32 histone features (`<mark>_<stage>`), NaN = no coverage |
| `eigenvector_features.csv` | 03 | 16,846 | eigenvector centrality per stage (`centrality_<stage>`) |
| `modeling_table.csv` | 04 | 4,509 | target, hub flags per stage, 32 histone features |

In `modeling_table.csv`, `target` is 1 = Stable (hub in 3–4 stages) and 0 = Transient (hub in 1–2 stages).
