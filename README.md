# EffectorHunt

> **Iterative RXLR Effector Motif Prediction Pipeline for *Phytophthora* and related oomycetes**

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux-lightgrey)](https://www.linux.org/)

---

## Author

**Deepbendu Das**
GitHub: [@dasdeepbendu12-dev](https://github.com/dasdeepbendu12-dev)

---

## What is EffectorHunt?

*Phytophthora* RXLR effector proteins evolve at extraordinary speed to evade plant immune systems — making them nearly invisible to standard sequence-similarity searches like BLAST. **EffectorHunt** solves this by implementing a self-reinforcing discovery loop:

```
Seed FASTA
    │
    ▼
 CD-HIT ──► Cluster sequences by identity
    │
    ▼
ClustalW ──► Align each cluster
    │
    ▼
hmmbuild ──► Build a statistical HMM fingerprint per cluster
    │
    ▼
hmmsearch ──► Scan the genome database for matches
    │
    ▼
New hits FASTA
    │
    └──► Same as before? ──► STOP (converged)
         Different?      ──► Feed back as new seed (next iteration)
```

Each round, newly discovered distant relatives are added to the seed pool, making the HMM fingerprints progressively broader and more sensitive. The pipeline repeats until convergence — when a full genome search returns exactly the same set of proteins as the previous round.

---

## Repository Contents

This repository contains three files you need to get started:

| File | Type | Purpose |
|------|------|---------|
| `effector_motif_pred-1.0.0.tar.gz` | Package | The installable Python package |
| `128_Phytopthora_RXLR.fa` | FASTA | **Test/sample file** — seed RXLR effector sequences |
| `all_128_effectors_fix.fa` | FASTA | **Test/sample file** — genome protein database |

> ⚠️ **Important:** `128_Phytopthora_RXLR.fa` and `all_128_effectors_fix.fa` are provided **only as sample/test datasets** to verify that the package is installed and running correctly on your system. They are not intended for real biological analysis. For actual effector prediction, replace them with your own organism's FASTA files.

---

## Requirements

### System
- Linux (Ubuntu 20.04+ recommended)
- Python ≥ 3.9
- Conda (for installing bioinformatics tools)

### Bioinformatics tools

Install all required tools in one Conda command:

```bash
conda create -n effector_tools -c bioconda -c conda-forge cd-hit hmmer clustalw -y
conda activate effector_tools
```

| Tool | Version | Purpose |
|------|---------|---------|
| [CD-HIT](http://cd-hit.org/) | ≥ 4.8 | Sequence clustering |
| [ClustalW](http://www.clustal.org/) | ≥ 2.1 | Multiple sequence alignment |
| [HMMER](http://hmmer.org/) | ≥ 3.3 | HMM building and searching |

### Python dependencies
These are installed **automatically** by pip — you do not need to install them manually:

| Package | Purpose |
|---------|---------|
| `biopython` | FASTA parsing and writing |
| `pyyaml` | Config file loading |
| `rich` | Coloured terminal output and progress |

---

## Installation

### Step 1 — Install bioinformatics tools (one-time)

```bash
conda create -n effector_tools -c bioconda -c conda-forge cd-hit hmmer clustalw -y
conda activate effector_tools
```

### Step 2 — Install the package (one-time)

```bash
pip install effector_motif_pred-1.0.0.tar.gz
```

---

## Quick Start

> ⚠️ Always run `conda activate effector_tools` first every time you open a new terminal.

### Step 1 — Run the interactive setup wizard

```bash
effector-motif-pred init
```

The wizard will ask four sets of questions. Press **Enter** to accept any default shown in brackets:

```
── Step 1 / 4 — Input files ────────────────
  Full path to seed RXLR FASTA: /home/ubuntu/data/128_Phytopthora_RXLR.fa
  ✔ File found
  Full path to genome/database FASTA: /home/ubuntu/data/all_128_effectors_fix.fa
  ✔ File found

── Step 2 / 4 — Output ─────────────────────
  Output folder name [effector_results]:

── Step 3 / 4 — Clustering settings ────────
  Minimum cluster size [10]:
  CD-HIT identity threshold [0.9]:
  Maximum iterations [20]:

── Step 4 / 4 — CPU threads ─────────────────
  Threads for CD-HIT [8]:
  Threads for hmmsearch [4]:

✔  config.yaml written successfully!
```

### Step 2 — Verify your environment

```bash
effector-motif-pred check
```

All rows should show green ✔:

```
        Environment Check
┌───────────────┬────────────┬────────────────────────┐
│ Item          │ Status     │ Detail                 │
├───────────────┼────────────┼────────────────────────┤
│ CD-HIT        │ ✔ found    │ /usr/bin/cd-hit        │
│ ClustalW      │ ✔ found    │ /usr/bin/clustalw      │
│ hmmbuild      │ ✔ found    │ /usr/bin/hmmbuild      │
│ hmmsearch     │ ✔ found    │ /usr/bin/hmmsearch     │
│ RXLR FASTA    │ ✔ exists   │ /path/to/rxlr.fa       │
│ Genome FASTA  │ ✔ exists   │ /path/to/genome.fa     │
└───────────────┴────────────┴────────────────────────┘
All checks passed.
```

### Step 3 — Run the pipeline

```bash
effector-motif-pred run
```

---

## Output Structure

```
effector_results/
├── iter1/
│   ├── cdhit/              CD-HIT clustering output
│   ├── clusters/           Per-cluster FASTA files
│   ├── aln/                ClustalW alignments
│   ├── hmm/                HMM profiles
│   ├── hmmsearch/          Search results (.tbl + .out)
│   ├── lists/              unique_hits.txt
│   └── final.fasta         ★ Hits this round (seed for iter2)
├── iter2/
│   └── ...
└── iterN/
    └── final.fasta         ★ YOUR FINAL CONVERGED RESULT
```

To find your final result quickly:
```bash
find effector_results -name "final.fasta" | sort | tail -1
```

---

## Configuration Reference

```yaml
# Input files
input_rxlr_fasta:   "/path/to/seed_rxlr.fa"
input_genome_fasta: "/path/to/all_proteins.fa"

# Output
output_dir: "effector_results"

# CD-HIT
cdhit_prog: "cd-hit"
cdhit_c: 0.9              # Identity threshold (0.4–1.0)
cdhit_d: 0

# Cluster filtering
min_cluster_size: 10

# Tool binaries
clustalw_bin: "clustalw"
hmmbuild_bin: "hmmbuild"
hmmsearch_bin: "hmmsearch"

# Convergence
max_iterations: 20

# Threads
threads:
  cdhit: 8
  hmmsearch: 4
```

### CD-HIT identity threshold guide

| `cdhit_c` | Sensitivity | Use when |
|-----------|-------------|---------|
| 0.9 | Strict | Closely related sequences |
| 0.7 – 0.8 | Moderate | Typical effector families |
| 0.5 – 0.6 | Sensitive | Highly diverged effectors |
| 0.4 | Most sensitive | Very distant homologs |

---

## CLI Reference

```
effector-motif-pred init                     Interactive wizard → writes config.yaml
effector-motif-pred check                    Verify all tools and input files
effector-motif-pred run                      Run the full pipeline
effector-motif-pred run -c myconf.yaml       Run with a specific config file
```

---

## Python API

```python
from effector_motif_pred import load_config, iterate_pipeline

cfg = load_config("config.yaml")
final_fasta = iterate_pipeline(cfg)
print(f"Converged result: {final_fasta}")
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `command not found: effector-motif-pred` | Conda env not active | Run `conda activate effector_tools` |
| `✗ NOT FOUND` in check | Tool not installed | `conda install -c bioconda cd-hit hmmer clustalw` |
| `kept (≥10 seqs): 0` | Clusters too small | Lower `min_cluster_size` or `cdhit_c` in config |
| `YAML ReaderError` | Bad characters in config | Delete `config.yaml`, re-run `effector-motif-pred init` |
| Final FASTA is empty | No HMM hits in genome | Check genome FASTA contains the correct organism's proteins |

---

## Running Tests

```bash
pip install -e ".[dev]"
pytest tests/ -v
```

---

## Citation

If you use EffectorHunt in your research, please cite:

```
Deepbendu Das (2024). EffectorHunt: Iterative RXLR Effector Motif Prediction Pipeline.
GitHub. https://github.com/dasdeepbendu12-dev/EffectorHunt
```

---

## License

MIT © Deepbendu Das
