# EffectorHunt

> **Iterative RXLR Effector Motif Prediction Pipeline for *Phytophthora* and related oomycetes**

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux-lightgrey)](https://www.linux.org/)

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

### Option A — Install from tarball (recommended for most users)

```bash
# Activate your Conda environment first
conda activate effector_tools

# Install the package
pip install effector_motif_pred-1.0.0.tar.gz
```

### Option B — Install from GitHub source

```bash
git clone https://github.com/dasdeepbendu12-dev/EffectorHunt.git
cd EffectorHunt
pip install .
```

### Option C — Editable install (for developers)

```bash
git clone https://github.com/dasdeepbendu12-dev/EffectorHunt.git
cd EffectorHunt
pip install -e ".[dev]"
```

---

## Quick Start

### Step 1 — Activate your Conda environment

```bash
conda activate effector_tools
```

> ⚠️ You must do this every time you open a new terminal before running the pipeline.

---

### Step 2 — Install the package (one-time only)

```bash
pip install effector_motif_pred-1.0.0.tar.gz
```

---

### Step 3 — Run the interactive setup wizard

```bash
effector-motif-pred init
```

The wizard will ask you four sets of questions. Just press **Enter** to accept any default shown in brackets:

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

---

### Step 4 — Verify your environment

```bash
effector-motif-pred check
```

You should see a table with green ✔ on every row:

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

---

### Step 5 — Run the pipeline

```bash
effector-motif-pred run
```

The terminal will show live progress for every iteration:

```
────────────── Effector Motif Prediction Pipeline ──────────────
Seed FASTA  : /home/ubuntu/data/128_Phytopthora_RXLR.fa
Genome FASTA: /home/ubuntu/data/all_128_effectors_fix.fa
Output dir  : /home/ubuntu/effector_results
Max iters   : 20

──────────────────── Iteration 1 ───────────────────────────────
CD-HIT → clustered.fasta
Clusters after CD-HIT: 24   kept (≥10 seqs): 8
ClustalW + hmmbuild ████████████████ 8/8
hmmsearch — running 8 HMMs with 4 threads each
Unique HMM hits this round: 312
Wrote 312 sequences → iter1/final.fasta
Prev hits: 128   New hits: 312   Δ = 184 new / 0 lost

──────────────────── Iteration 2 ───────────────────────────────
...

──────────── ✔ Converged at iteration 4 ────────────────────────
Final FASTA: /home/ubuntu/effector_results/iter4/final.fasta
```

---

## Output Structure

```
effector_results/
├── iter1/
│   ├── cdhit/
│   │   ├── clustered.fasta         CD-HIT representative sequences
│   │   └── clustered.fasta.clstr   Cluster membership file
│   ├── clusters/
│   │   ├── cluster_0.fa            Sequences for cluster 0
│   │   └── ...
│   ├── aln/
│   │   ├── cluster_0.aln           ClustalW alignment for cluster 0
│   │   └── ...
│   ├── hmm/
│   │   ├── cluster_0.hmm           HMM profile for cluster 0
│   │   └── ...
│   ├── hmmsearch/
│   │   ├── cluster_0.tbl           hmmsearch hit table (parseable)
│   │   ├── cluster_0.out           hmmsearch full output
│   │   └── ...
│   ├── lists/
│   │   └── unique_hits.txt         All unique hit IDs this round
│   └── final.fasta                 ★ Hits this round (seed for iter2)
├── iter2/
│   └── ...
└── iterN/
    └── final.fasta                 ★ YOUR FINAL CONVERGED RESULT
```

> **Your final result** is always the `final.fasta` in the last iteration folder. The exact path is printed at the end of the run.

To find it quickly:
```bash
find effector_results -name "final.fasta" | sort | tail -1
```

---

## Configuration Reference

```yaml
# ── Input files ──────────────────────────────────────────────
input_rxlr_fasta:   "/path/to/seed_rxlr.fa"      # Required
input_genome_fasta: "/path/to/all_proteins.fa"    # Required

# ── Output ───────────────────────────────────────────────────
output_dir: "effector_results"

# ── CD-HIT settings ──────────────────────────────────────────
cdhit_prog: "cd-hit"
cdhit_c: 0.9              # Identity threshold (0.4–1.0)
cdhit_d: 0

# ── Cluster filtering ────────────────────────────────────────
min_cluster_size: 10

# ── Tool binaries ────────────────────────────────────────────
clustalw_bin: "clustalw"
hmmbuild_bin: "hmmbuild"
hmmsearch_bin: "hmmsearch"

# ── Convergence ──────────────────────────────────────────────
max_iterations: 20

# ── Threads ──────────────────────────────────────────────────
threads:
  cdhit: 8
  hmmsearch: 4
```

### CD-HIT identity threshold guide

| `cdhit_c` value | Sensitivity | Use when |
|-----------------|-------------|---------|
| 0.9 (default) | Strict | Closely related sequences |
| 0.7 – 0.8 | Moderate | Typical effector families |
| 0.5 – 0.6 | Sensitive | Highly diverged effectors |
| 0.4 | Most sensitive | Very distant homologs |

---

## Python API

```python
from effector_motif_pred import load_config, iterate_pipeline

cfg = load_config("config.yaml")
final_fasta = iterate_pipeline(cfg)
print(f"Converged result: {final_fasta}")
```

Run a single iteration manually:

```python
from pathlib import Path
from effector_motif_pred import load_config, run_single_iteration

cfg = load_config("config.yaml")
output = run_single_iteration(
    input_fasta=Path("my_seed.fa"),
    workdir=Path("iter1"),
    cfg=cfg,
)
```

---

## CLI Reference

```
effector-motif-pred init                     Interactive wizard → writes config.yaml
effector-motif-pred check                    Verify all tools and input files
effector-motif-pred run                      Run the full pipeline
effector-motif-pred run -c myconf.yaml       Run with a specific config file
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `command not found: effector-motif-pred` | Conda env not active | Run `conda activate effector_tools` |
| `✗ NOT FOUND` in check | Tool not installed | Run `conda install -c bioconda cd-hit hmmer clustalw` |
| `kept (≥10 seqs): 0` | No clusters large enough | Lower `min_cluster_size` or `cdhit_c` in config |
| `YAML ReaderError` | Bad characters in config | Delete `config.yaml` and re-run `effector-motif-pred init` |
| Final FASTA is empty | No HMM hits in genome | Check that genome FASTA contains the right organism's proteins |

---

## Running Tests

```bash
pip install -e ".[dev]"
pytest tests/ -v
```

---

## Project Structure

```
EffectorHunt/
├── src/effector_motif_pred/
│   ├── __init__.py          Public API
│   ├── cli.py               Command-line interface + setup wizard
│   ├── config.py            YAML config loader and validator
│   ├── pipeline.py          Single iteration logic
│   ├── iterate.py           Convergence loop
│   └── config_template.yaml Bundled config template
├── tests/
│   └── test_config.py
├── pyproject.toml
├── setup.cfg
└── README.md
```

---

## Citation

If you use EffectorHunt in your research, please cite:

```
Deepbendu Das (2024). EffectorHunt: Iterative RXLR Effector Motif Prediction Pipeline.
GitHub. https://github.com/dasdeepbendu12-dev/EffectorHunt
```

---

## Author

**Deepbendu Das**
GitHub: [@dasdeepbendu12-dev](https://github.com/dasdeepbendu12-dev)

---

## License

MIT © Deepbendu Das
