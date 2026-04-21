# EffectorHunt

> **Iterative RXLR Effector Motif Prediction Pipeline for *Phytophthora* and related oomycetes**

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Linux-lightgrey)](https://www.linux.org/)

**Author:** Deepbendu Das
**GitHub:** [@dasdeepbendu12-dev](https://github.com/dasdeepbendu12-dev)

---

## What does EffectorHunt do?

*Phytophthora* RXLR effector proteins evolve rapidly to evade plant immunity and are often missed by standard searches like BLAST. EffectorHunt finds them by building statistical fingerprints (Hidden Markov Models) from known effector sequences and repeatedly scanning a genome database — adding newly discovered proteins to the search pool each round until no new effectors are found (convergence).

```
Seed FASTA ──► CD-HIT ──► ClustalW ──► hmmbuild ──► hmmsearch ──► New hits
                                                                       │
                          ◄──────── feed back as new seed if changed ◄─┘
                                      stop if nothing changed (converged)
```

---

## What is in this repository?

| File | Purpose |
|------|---------|
| `effector_motif_pred-1.0.0.tar.gz` | The installable Python package |
| `128_Phytopthora_RXLR.fa` | **Sample test file** — seed RXLR effector sequences |
| `all_128_effectors_fix.fa` | **Sample test file** — genome protein database |
| `README.md` | This guide |

> ⚠️ The two `.fa` files are **sample datasets only** — provided so you can test that the package is working correctly on your machine. For real biological analysis, replace them with your own FASTA files.

---

## Step 0 — Get all the files onto your machine

If you have Git installed, run this single command in your terminal. It downloads everything in the repository into a new folder called `EffectorHunt`:

```bash
git clone https://github.com/dasdeepbendu12-dev/EffectorHunt.git
cd EffectorHunt
```

If you do not have Git, you can also download everything manually:
- Go to `https://github.com/dasdeepbendu12-dev/EffectorHunt`
- Click the green **Code** button → **Download ZIP**
- Unzip the folder on your computer

---

## Step 1 — Install Conda (if you don't have it)

Conda is a tool that manages bioinformatics software. If you have never used it before:

1. Go to [https://docs.conda.io/en/latest/miniconda.html](https://docs.conda.io/en/latest/miniconda.html)
2. Download the **Linux 64-bit** installer
3. Run it in your terminal:
   ```bash
   bash Miniconda3-latest-Linux-x86_64.sh
   ```
4. Follow the on-screen prompts and say **yes** to everything
5. Close and reopen your terminal when it finishes

---

## Step 2 — Install the bioinformatics tools

This creates a dedicated environment called `effector_tools` and installs CD-HIT, ClustalW, and HMMER inside it. You only need to do this **once**:

```bash
conda create -n effector_tools -c bioconda -c conda-forge cd-hit hmmer clustalw -y
```

This may take a few minutes. When it finishes, activate the environment:

```bash
conda activate effector_tools
```

> ⚠️ You must run `conda activate effector_tools` **every time** you open a new terminal before using EffectorHunt. You will see `(effector_tools)` appear at the start of your terminal prompt when it is active.

---

## Step 3 — Install EffectorHunt

Make sure you are inside the `EffectorHunt` folder (the one you cloned or unzipped), then run:

```bash
pip install effector_motif_pred-1.0.0.tar.gz
```

You will see pip downloading and installing dependencies automatically. When you see `Successfully installed effector-motif-pred-1.0.0` the installation is complete.

---

## Step 4 — Run the setup wizard

EffectorHunt comes with an interactive wizard that asks you simple questions and writes your configuration file automatically. You never need to manually edit any files:

```bash
effector-motif-pred init
```

The wizard will guide you through four steps:

```
── Step 1 / 4 — Input files ──────────────────────────────
  e.g. /home/ubuntu/EffectorHunt/128_Phytopthora_RXLR.fa
  Full path to seed RXLR FASTA: /home/ubuntu/EffectorHunt/128_Phytopthora_RXLR.fa
  ✔ File found

  e.g. /home/ubuntu/EffectorHunt/all_128_effectors_fix.fa
  Full path to genome/database FASTA: /home/ubuntu/EffectorHunt/all_128_effectors_fix.fa
  ✔ File found

── Step 2 / 4 — Output ───────────────────────────────────
  Output folder name [effector_results]:        ← press Enter to accept

── Step 3 / 4 — Clustering settings ─────────────────────
  Minimum cluster size [10]:                    ← press Enter to accept
  CD-HIT identity threshold [0.9]:              ← press Enter to accept
  Maximum iterations [20]:                      ← press Enter to accept

── Step 4 / 4 — CPU threads ──────────────────────────────
  Threads for CD-HIT [8]:                       ← press Enter to accept
  Threads for hmmsearch [4]:                    ← press Enter to accept

✔  config.yaml written successfully!
```

**Tip:** For the file paths, type the full path to where the `.fa` files are on your machine. If you used `git clone`, they will be inside the `EffectorHunt` folder. To find the exact path, run:
```bash
find ~ -name "128_Phytopthora_RXLR.fa" 2>/dev/null
```

---

## Step 5 — Check your environment

Before running, verify that all tools are found and your files exist:

```bash
effector-motif-pred check
```

Expected output — every row should show a green ✔:

```
        Environment Check
┌───────────────┬────────────┬───────────────────────────┐
│ Item          │ Status     │ Detail                    │
├───────────────┼────────────┼───────────────────────────┤
│ CD-HIT        │ ✔ found    │ /usr/bin/cd-hit           │
│ ClustalW      │ ✔ found    │ /usr/bin/clustalw         │
│ hmmbuild      │ ✔ found    │ /usr/bin/hmmbuild         │
│ hmmsearch     │ ✔ found    │ /usr/bin/hmmsearch        │
│ RXLR FASTA    │ ✔ exists   │ /path/to/rxlr.fa          │
│ Genome FASTA  │ ✔ exists   │ /path/to/genome.fa        │
└───────────────┴────────────┴───────────────────────────┘
All checks passed.  Run with: effector-motif-pred run
```

If any row shows a red ✗, do not proceed — fix the issue first. See the Troubleshooting section below.

---

## Step 6 — Run the pipeline

```bash
effector-motif-pred run
```

The terminal will print live progress for every iteration. When the pipeline finishes, it will print the exact path of your final result:

```
✔ Converged at iteration 4
Final FASTA: /home/ubuntu/effector_results/iter4/final.fasta
```

---

## Where are my results?

All results are saved in the output folder you named in the wizard (default: `effector_results`), organized by iteration:

```
effector_results/
├── iter1/
│   ├── cdhit/          CD-HIT clustering output
│   ├── clusters/       Per-cluster FASTA files
│   ├── aln/            ClustalW alignments
│   ├── hmm/            HMM profiles
│   ├── hmmsearch/      Raw search results
│   ├── lists/          List of all unique hit IDs
│   └── final.fasta     Hits from this round
├── iter2/
│   └── ...
└── iterN/
    └── final.fasta     ★ YOUR FINAL ANSWER — the converged effector set
```

The **most important file** is the `final.fasta` inside the last iteration folder. To find it quickly:

```bash
find effector_results -name "final.fasta" | sort | tail -1
```

---

## Troubleshooting

| Problem | What it means | How to fix |
|---------|--------------|------------|
| `command not found: effector-motif-pred` | Conda environment is not active | Run `conda activate effector_tools` |
| `✗ NOT FOUND` for a tool | Tool is not installed | Run `conda install -c bioconda cd-hit hmmer clustalw` |
| `kept (≥10 seqs): 0` | No clusters were large enough | Lower `min_cluster_size` to 5 or `cdhit_c` to 0.7 by re-running `init` |
| `YAML ReaderError` | Config file has bad characters | Delete `config.yaml`, run `effector-motif-pred init` again |
| Final FASTA is empty | No HMM matches found | Make sure the genome FASTA contains proteins from the correct organism |

---

## CD-HIT identity threshold guide

The `cdhit_c` setting controls how similar sequences must be to be grouped together. Lower values find more distant relatives:

| Value | Sensitivity | When to use |
|-------|-------------|-------------|
| 0.9 (default) | Strict | Closely related sequences |
| 0.7 – 0.8 | Moderate | Typical effector families |
| 0.5 – 0.6 | Sensitive | Highly diverged effectors |
| 0.4 | Most sensitive | Very distant homologs |

---

## Citation

If you use EffectorHunt in your research, please cite:

```
Deepbendu Das (2025). EffectorHunt: Iterative RXLR Effector Motif Prediction Pipeline.
GitHub. https://github.com/computational-genomics-lab/EffectorHunt.git
```

---

## License

Copyright (c) 2026 STLAB at Indian Institute of Chemical Biology

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
