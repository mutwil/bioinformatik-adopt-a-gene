# Module 4 "Adopt-a-Gene" — Quarto Live (webR) interactive page

This folder is a self-contained **Quarto Live** module. R runs **in the student's browser** (webR/WebAssembly) — no install for students, and it hosts for free as a static site (GitHub Pages / Quarto Pub / Netlify). All heavy steps (kallisto, DESeq2, BioGRID) are **pre-computed** into `data/`.

```
quarto_live/
├── module4-adopt-a-gene.qmd     # the interactive lesson
├── data/                        # pre-computed, browser-sized (loaded by webR)
│   ├── expression_matrix.csv    # Klepikova atlas E-MTAB-7978, 3585 genes × 56 organs (REAL)
│   ├── gene_info.csv            # gene_id, symbol, description (REAL)
│   ├── ppi_edges.csv            # BioGRID physical interactions among matrix genes (REAL)
│   ├── gene_sets.csv            # GO term → genes, for in-browser ORA (REAL)
│   ├── qc_table.csv             # kallisto QC — EXAMPLE values (replace with a real run)
│   └── de_results.csv           # DESeq2-style result — EXAMPLE values (replace with a real run)
└── README.md
```

## One-time setup

You need [Quarto](https://quarto.org) installed. In **this folder**:

```bash
quarto add r-wasm/quarto-live      # installs the live (webR) extension into ./_extensions
```

## Preview locally

```bash
quarto preview module4-adopt-a-gene.qmd
```

The first load downloads webR (~tens of MB) into the browser cache; afterwards it's fast. Code cells share one R session — students run them top to bottom.

## Publish (free)

```bash
quarto publish gh-pages module4-adopt-a-gene.qmd     # → https://<user>.github.io/<repo>/
# or:  quarto publish quarto-pub
```

Link it from Absalon. Students just click — no R, no Python, no Colab.

## How the data flows

- `resources: [data]` in the `.qmd` YAML copies `data/` into webR's virtual filesystem at startup, so `read.csv("data/expression_matrix.csv")` works on the published site.
- The lesson uses **base R only** (cor, phyper, plot, p.adjust) → no package installs → maximum reliability in webR.
- State is shared across the ordinary `{webr}` cells, so the adopted gene threads through every stage.

## What's real vs. example

- **Real & precomputed by you:** expression matrix (E-MTAB-7978), PPI (BioGRID), GO sets (GO GAF). The co-expression → network → enrichment science is genuine — e.g. CESA7's neighbours come back as CESA8/IRX/TBL/CTL (the real secondary-cell-wall module).
- **Example only:** `qc_table.csv` and `de_results.csv`. Regenerate them from a real run with the server notebook (`Module4_AdoptAGene_Dossier_ANSWERS.ipynb`, Stage 5 needs a replicated 2-condition dataset) and drop the CSVs in `data/`.

## Regenerating the data

From the course root: `python3 precompute_quarto_data.py` (downloads the public sources to /tmp, writes the CSVs here).

## Turning a step into a formally-graded exercise (optional)

The lesson uses editable cells + `✅`/hint checks (reliable, keeps shared state). To use Quarto Live's graded boxes instead, convert a step to an `exercise` + `check` cell:

```{webr}
#| exercise: tpm
tpm <- sweep(rpk, 2, ______, "/") * 1e6
```
```{webr}
#| exercise: tpm
#| check: true
if (all(abs(colSums(.result) - 1e6) < 1)) list(correct = TRUE, message = "Nailed it!") else
  list(correct = FALSE, message = "Each column should sum to 1e6.")
```
Note: exercise cells are sandboxed (own environment), so give each one a `#| setup: true` cell that re-loads the data — that's why the threaded pipeline here uses shared ordinary cells instead.
