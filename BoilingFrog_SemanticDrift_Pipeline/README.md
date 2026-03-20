# Boiling Frog Semantic Drift Pipeline

Notebook-based pipeline for measuring semantic drift in iterative LLM paraphrasing on PADBen.

The analysis asks whether each paraphrase hop can appear "safe" while cumulative drift silently exceeds a detection threshold (the "boiling frog" effect).

## What this directory contains

- `BoilingFrog_SemanticDrift_Pipeline.ipynb`: end-to-end pipeline notebook.
- `outputs/`: exported CSVs and figures (if the notebook has been run).

Tracked output files currently present:

- `outputs/data_quality_report.csv`
- `outputs/extreme_drift_cases.csv`
- `outputs/extreme_boiling_frog_cases.csv`
- `outputs/generation_artifacts.csv`
- `outputs/paper_table1_length_stats.csv`
- `outputs/paper_chain_length_stats.csv`
- `outputs/paper_all_type_length_stats.csv`

## Dataset contract (input)

Default input file is `./data.json` (configurable in the notebook).

Each record is validated via Pydantic (`PADBenRecord`) and expected to include:

- `idx`
- `dataset_source` in `{mrpc, paws, hlpc}`
- `human_original_text(type1)`
- `llm_generated_text(type2)`
- `human_paraphrased_text(type3)`
- `llm_paraphrased_original_text(type4)-prompt-based`
- `llm_paraphrased_generated_text(type5)-1st`
- `llm_paraphrased_generated_text(type5)-3rd`

Validation behavior:

- Writes `outputs/validation_failures.csv` if invalid rows exist.
- Halts if validation failure rate is above 5%.

## Pipeline stages

The notebook is organized into 8 sections:

1. **Configuration & setup**
   - Central `PipelineConfig` dataclass (paths, thresholds, model choices).
   - Seed + device selection (`auto`, `cuda`, `cpu`) and logging setup.

2. **Data ingestion & validation**
   - Loads JSON and enforces schema/quality gates.
   - Creates standardized columns (`Type_1` to `Type_5_3rd`).
   - Computes a data fingerprint hash for cache invalidation.

3. **Data profiling**
   - Completeness, duplicates, token stats, source distribution.
   - Exports paper-oriented token-length summary tables.

4. **Preprocessing**
   - Optional text fixing (`ftfy`), whitespace cleanup, empty-row filtering.

5. **SBERT embeddings (cached)**
   - Model: `all-mpnet-base-v2`.
   - Caches embeddings in `cache/` as `.npy`, keyed by fingerprint.

6. **Multi-metric drift computation**
   - Per-hop and end-to-end metrics: Jaccard, ROUGE-L, METEOR, SBERT cosine/euclidean, edit distance.
   - Builds composite SDS using weighted normalized components:
     - SBERT: `0.6`
     - METEOR: `0.2`
     - ROUGE-L: `0.2`

7. **Sanity checks + extreme case discovery**
   - Range assertions and metric-correlation checks.
   - Detects extreme semantic divergence.
   - Classifies zero-overlap records into:
     - `extreme_boiling_frog` (lexical erosion but semantic continuity)
     - `generation_artifact` (both lexical and semantic collapse)

8. **Core analysis + exports**
   - Boiling-frog figures and correlation heatmaps.
   - Final result export (`drift_results.csv`) and metadata summary (`pipeline_summary.json`).

## Key configuration defaults

From `PipelineConfig`:

- `data_path`: `./data.json`
- `output_dir`: `./outputs`
- `cache_dir`: `./cache`
- `log_dir`: `./logs`
- `sbert_model`: `all-mpnet-base-v2`
- `sbert_batch_size`: `64`
- `parascore_threshold (gamma)`: `0.35`
- Severity boundaries: `low_med=0.35`, `med_high=0.45`
- `seed`: `42`

## How to run

1. Open `BoilingFrog_SemanticDrift_Pipeline.ipynb`.
2. Ensure `data.json` is available at the configured path (or edit `PipelineConfig`).
3. Run Section `0A` once (dependency install), then run all cells top-to-bottom.

Dependency install command used by the notebook:

```bash
pip install sentence-transformers nltk rouge-score bert-score scipy scikit-learn seaborn tqdm python-Levenshtein pydantic ftfy watermark
```

## Main outputs

Typical exports created by a full run:

- `outputs/drift_results.csv`
- `outputs/pipeline_summary.json`
- `outputs/data_quality_report.csv`
- `outputs/validation_failures.csv` (only when failures exist)
- `outputs/extreme_drift_cases.csv`
- `outputs/extreme_boiling_frog_cases.csv`
- `outputs/generation_artifacts.csv`
- Figures such as:
  - `fig_token_distributions.png`
  - `fig_source_distribution.png`
  - `fig_extreme_drift_discovery.png`
  - `fig_artifact_classification.png`
  - `fig_boiling_frog_effect.png`
  - `fig_correlation_heatmap.png`

## Notes

- Embedding generation is the most expensive step; cache reuse makes reruns much faster.
- `pipeline_summary.json` stores runtime, config snapshot, dataset stats, and key aggregate metrics.
