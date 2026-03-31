# 🐸 The Boiling Frog Effect: Semantic Drift in Iterative LLM Paraphrasing

> **Do individual paraphrase hops appear semantically "safe" while cumulative drift silently crosses dangerous thresholds?**

This repository contains the analysis pipelines for quantifying semantic erosion in iterative LLM paraphrasing — a phenomenon we call the **Boiling Frog Effect**. Just as a frog won't react to slowly heated water, current AI text detectors fail to flag the gradual accumulation of semantic drift across paraphrase chains, even though each individual hop appears benign.

---

## Research Context

Paraphrase attacks — where LLMs iteratively rewrite text to evade plagiarism and AI text detectors — represent a growing threat to academic integrity, copyright protection, and content authenticity. Prior work (Krishna et al., 2023; Sadasivan et al., 2023) demonstrated that detection accuracy collapses from over 90% to near random chance after paraphrasing. Yet no existing benchmark systematically evaluates *how* semantic meaning erodes across these chains.

This project builds on **PADBen** (Paraphrase Attack Detection Benchmark), a comprehensive benchmark from the GenAI Research Lab at Northeastern University designed to evaluate AI text detectors against paraphrase attacks across 4 tasks, diverse datasets, and standardized evaluation protocols. PADBen curates 5 text types per record — from original human text through LLM-generated, human-paraphrased, and multi-iteration LLM-paraphrased variants — drawn from MRPC, PAWS, and HLPC source corpora.

We extend PADBen's framework with a second corpus (**Ship of Theseus**) spanning 7 domains and 7 paraphrase models, enabling cross-domain and cross-model generalizability analysis.

---

## Key Findings

### The Boiling Frog Effect Is Real — and Universal

The effect was confirmed across **both datasets**, **all domains**, and **all paraphrase models**:

| Dataset | Cumulative SDS | E2E SDS | Threshold Multiplier |
|---------|---------------|---------|---------------------|
| **PADBen** (sentence-level) | 0.661 | 0.398 | **1.9× γ** |
| **Ship of Theseus** (paragraph-level) | 1.158 | 0.638 | **3.3× γ** |

*γ = 0.35, the ParaScore safety threshold from Shen et al. (EMNLP 2022).*

### Lexical Erosion Is the Primary Drift Channel

SBERT cosine similarity remains deceptively high (0.60–0.87) while surface-level metrics collapse, meaning detectors relying on lexical features are fundamentally vulnerable:

| Metric | PADBen (E2E) | Ship of Theseus (E2E) |
|--------|-------------|----------------------|
| SBERT Cosine Sim | 0.869 | 0.602 |
| Jaccard Similarity | 0.393 | 0.206 |
| ROUGE-L | 0.455 | 0.218 |
| METEOR | 0.479 | 0.232 |

### Drift Decelerates but Accumulates

In the 3-hop Ship of Theseus pipeline, per-hop drift decreases across iterations (Hop A: 0.603 → Hop B: 0.292 → Hop C: 0.263), but the cumulative total (1.158) far exceeds the safety threshold. This deceleration pattern is consistent across all 7 models.

### Cross-Model Rankings (Ship of Theseus, E2E SDS)

| Model | E2E SDS | Cumulative SDS |
|-------|---------|---------------|
| dipper(high) | 0.889 | 2.336 |
| dipper | 0.677 | 1.447 |
| chatgpt | 0.625 | 1.061 |
| pegasus(full) | 0.616 | 0.941 |
| dipper(low) | 0.579 | 0.907 |
| palm | 0.579 | 0.917 |
| pegasus(slight) | 0.544 | 0.697 |

### Cross-Domain Rankings (Ship of Theseus, E2E SDS)

| Domain | E2E SDS | Cumulative SDS |
|--------|---------|---------------|
| WP (Writing Prompts) | 0.694 | 1.225 |
| CMV (Change My View) | 0.677 | 1.178 |
| ELI5 | 0.659 | 1.181 |
| XSUM | 0.633 | 1.172 |
| YELP | 0.625 | 1.158 |
| SCI (Scientific) | 0.610 | 1.125 |
| TLDR | 0.574 | 1.061 |

### Extreme Boiling Frog Cases

The pipelines discovered cases where **zero original words survive 3 iterations, yet semantic meaning is still recognizable** — the strongest evidence for cumulative lexical drift:

- **PADBen:** 13 extreme Boiling Frog cases (out of 16,233 records)
- **Ship of Theseus:** 426 extreme Boiling Frog cases (out of 21,122 records)

---

## Repository Structure

```
semantic-drift/
├── README.md                              ← You are here
├── requirements.txt                       ← Python dependencies
├── .gitignore
│
├── BoilingFrog_SemanticDrift_Pipeline/     ← Pipeline 1: PADBen corpus
│   ├── BoilingFrog_SemanticDrift_Pipeline.ipynb
│   ├── README.md
│   ├── data.json                          ← PADBen input (16,233 records)
│   └── outputs/                           ← Generated results & figures
│       ├── drift_results.csv
│       ├── data_quality_report.csv
│       ├── extreme_boiling_frog_cases.csv
│       ├── extreme_drift_cases.csv
│       ├── generation_artifacts.csv
│       ├── paper_table1_length_stats.csv
│       ├── paper_chain_length_stats.csv
│       ├── paper_all_type_length_stats.csv
│       ├── fig_boiling_frog_effect.png
│       ├── fig_correlation_heatmap.png
│       ├── fig_artifact_classification.png
│       ├── fig_extreme_drift_discovery.png
│       ├── fig_token_distributions.png
│       └── fig_source_distribution.png
│
├── BoilingFrog_MultiModel_Pipeline/       ← Pipeline 2: Ship of Theseus corpus
│   ├── BoilingFrog_MultiModel_Pipeline.ipynb
│   ├── README.md
│   ├── output_json/                       ← Multi-model input data
│   │   ├── CMV/                           ← 7 model JSON files per domain
│   │   ├── ELI5/
│   │   ├── SCI/
│   │   ├── TLDR/
│   │   ├── WP/
│   │   ├── XSUM/
│   │   └── YELP/
│   └── outputs/                           ← Generated results & figures
│       ├── drift_results.csv
│       ├── data_quality_report.csv
│       ├── extreme_boiling_frog_cases.csv
│       ├── generation_artifacts.csv
│       ├── validation_failures.csv
│       ├── fig_boiling_frog_3hop.png
│       ├── fig_cross_model_comparison.png
│       ├── fig_cross_domain_comparison.png
│       ├── fig_domain_model_heatmap.png
│       ├── fig_cross_dataset_comparison.png
│       ├── fig_padben_per_source.png
│       ├── fig_correlation_heatmap.png
│       ├── fig_token_distributions.png
│       └── fig_domain_model_distribution.png
```

---

## Pipelines Overview

### Pipeline 1: PADBen Semantic Drift (Sentence-Level)

Analyzes the PADBen benchmark dataset (16,233 sentence-level records from MRPC, PAWS, and HLPC) to measure drift across a 2-hop paraphrase chain (`Type 2 → Type 5-1st → Type 5-3rd`), plus an end-to-end measurement and an anchor comparison against the human original (Type 1).

**Text types analyzed:** Type 1 (Human Original), Type 2 (LLM-Generated), Type 3 (Human-Paraphrased), Type 4 (LLM-Paraphrased Human), Type 5-1st & Type 5-3rd (Iterative LLM Paraphrases).

### Pipeline 2: Ship of Theseus Multi-Model (Paragraph-Level)

Extends the analysis to 21,122 paragraph-level records across 7 domains (CMV, ELI5, SCI, TLDR, WP, XSUM, YELP) and 7 paraphrase models (ChatGPT, DIPPER, DIPPER-high, DIPPER-low, PaLM, Pegasus-full, Pegasus-slight). Measures drift across a full 3-hop chain (`Type 1 → Type 5-1st → Type 5-2nd → Type 5-3rd`) plus end-to-end.

Includes a **cross-dataset hyper view** that compares findings between PADBen and Ship of Theseus side-by-side.

### Shared Architecture

Both pipelines follow an identical 8-stage architecture:

```
[Extract] → [Validate] → [Profile] → [Transform] → [Compute] → [Verify] → [Analyze] → [Export]
   JSON      Pydantic     Quality     Cleaning     Metrics     Sanity      Boiling     Results
   Load      Schema       Report      & Filter     (cached)    Checks      Frog        & Logs
```

---

## Metrics & Methodology

### Composite Semantic Drift Score (SDS)

The SDS is a weighted, normalized composite that balances deep semantic similarity with surface-level metrics:

| Component | Weight | What It Captures |
|-----------|--------|-----------------|
| SBERT Cosine Distance (`all-mpnet-base-v2`) | 0.6 | Deep semantic drift (embedding-level) |
| 1 − METEOR | 0.2 | N-gram alignment + synonymy erosion |
| 1 − ROUGE-L | 0.2 | Longest common subsequence loss |

Each component is min-max normalized before weighting. Higher SDS = more drift.

### Additional Metrics Computed

- **Jaccard Similarity** — word-level overlap (bag of words)
- **SBERT Euclidean Distance** — magnitude of embedding displacement
- **Levenshtein Edit Distance** — character-level transformation cost
- **BERTScore** — contextual token-level alignment (pipeline 1 only)

### Thresholds & Classification

- **ParaScore γ = 0.35** (Shen et al., EMNLP 2022) — the safety boundary for paraphrase detection
- **Severity bands:** Low (< 0.35), Medium (0.35–0.45), High (> 0.45)
- **Artifact classification (dual-criteria):**
  - *Extreme Boiling Frog:* ROUGE-L ≈ 0 AND SBERT cosine > 0.1 (zero words survive, meaning preserved)
  - *Generation Artifact:* ROUGE-L ≈ 0 AND SBERT cosine ≤ 0.1 (hallucination — both form and meaning lost)

---

## Getting Started

### Prerequisites

- Python 3.10+
- CUDA-capable GPU recommended (embedding computation is the bottleneck)
- ~4 GB disk space for cached embeddings

### Installation

```bash
git clone https://github.com/<your-username>/semantic-drift.git
cd semantic-drift
pip install -r requirements.txt
```

**Full dependency list** (used by the notebooks):

```bash
pip install sentence-transformers nltk rouge-score bert-score scipy scikit-learn seaborn tqdm python-Levenshtein pydantic ftfy watermark
```

### Running Pipeline 1 (PADBen)

```bash
cd BoilingFrog_SemanticDrift_Pipeline
# Ensure data.json is present
jupyter notebook BoilingFrog_SemanticDrift_Pipeline.ipynb
```

Run Section 0A once (dependency install), then run all cells top-to-bottom. Runtime: ~3 minutes on GPU.

### Running Pipeline 2 (Ship of Theseus)

```bash
cd BoilingFrog_MultiModel_Pipeline
# Ensure output_json/ directory contains domain/model JSON files
jupyter notebook BoilingFrog_MultiModel_Pipeline.ipynb
```

Run Section 0A once, then run all cells top-to-bottom. Runtime: ~26 minutes on GPU.

For the cross-dataset comparison section, Pipeline 1 must be run first (it reads `drift_results.csv` from the PADBen pipeline outputs).

---

## Configuration

Both pipelines centralize all parameters in a `PipelineConfig` dataclass:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `sbert_model` | `all-mpnet-base-v2` | Sentence-BERT model for embeddings |
| `sbert_batch_size` | 64 | Embedding batch size |
| `weight_sbert` | 0.6 | SBERT weight in composite SDS |
| `weight_meteor` | 0.2 | METEOR weight in composite SDS |
| `weight_rouge` | 0.2 | ROUGE-L weight in composite SDS |
| `parascore_threshold` | 0.35 | ParaScore γ safety threshold |
| `seed` | 42 | Random seed for reproducibility |
| `device` | `auto` | Compute device (`auto`, `cuda`, `cpu`) |

---

## Quality Assurance

Both pipelines enforce strict quality gates throughout:

- **Pydantic schema validation** on every input record (halts if >5% fail)
- **Data profiling** — completeness, duplicates, token-length distributions, encoding checks
- **23 automated sanity checks** per pipeline run (monotonicity, cross-metric correlations, known-pair tests, range assertions)
- **Dual-criteria artifact classification** to separate true Boiling Frog cases from generation hallucinations
- **Embedding cache** with hash-based invalidation for reproducible reruns
- **Pipeline summary JSON** capturing full runtime, config snapshot, and aggregate statistics

---

## Key References

- **PADBen:** Paraphrase Attack Detection Benchmark — GenAI Research Lab, Khoury College, Northeastern University
- **ParaScore:** Shen et al., "ParaScore: Automatic Evaluation for Paraphrase Generation," EMNLP 2022
- **DIPPER:** Krishna et al., "Paraphrasing Evades Detectors of AI-Generated Text, but Retrieval Is an Effective Defense," NeurIPS 2023
- **RAID:** Dugan et al., "RAID: A Shared Benchmark for Robust Evaluation of Machine-Generated Text Detectors," ACL 2024
- **Sadasivan et al.,** "Can AI-Generated Text be Reliably Detected?" 2023

---

## License

This project is part of ongoing research, Please contact the authors for usage and citation information.
