# Boiling Frog Multi-Model Pipeline

Notebook-based pipeline for cross-domain, cross-model semantic drift analysis under iterative paraphrasing.

This version extends the PADBen-style drift framework to compare **7 paraphrase models** across **7 domains** and measure how drift accumulates over 3 hops.

## What this directory contains

- `BoilingFrog_MultiModel_Pipeline.ipynb`: end-to-end multi-model pipeline.
- `outputs/`: exported CSVs and figures (if notebook has been run).

Tracked output files currently present:

- `outputs/data_quality_report.csv`
- `outputs/validation_failures.csv`

## Input format

Default data root is `./output_json` (configurable in notebook).

Expected directory structure:

```text
output_json/
  <DOMAIN>/
    <MODEL>.json
    <MODEL>.json
    ...
```

The notebook scans each domain subdirectory and each `*.json` file, then injects metadata:

- `domain` from the folder name
- `model` from the file stem

Each JSON record is validated through `MultiModelRecord` and must include:

- `idx`
- `dataset_source`
- `human_original_text(type1)`
- `llm_paraphrased_generated_text(type5)-1st`
- `llm_paraphrased_generated_text(type5)-2nd`
- `llm_paraphrased_generated_text(type5)-3rd`

Notes:

- Type2/Type3/Type4 are intentionally not used here.
- Invalid rows are written to `outputs/validation_failures.csv`.

## Pipeline stages

The notebook is organized into the following analysis flow:

1. **Configuration & setup**
   - `PipelineConfig` dataclass for paths, thresholds, and weights.
   - Seed/device setup and logging.

2. **Data ingestion + schema validation**
   - Loads all domain/model JSON files.
   - Builds a Domain × Model inventory table.

3. **Data profiling**
   - Completeness and token-length diagnostics.
   - Domain × model distribution checks.

4. **Preprocessing**
   - Cleans/filters records before metric computation.

5. **SBERT embedding computation (cached)**
   - Model: `all-mpnet-base-v2`
   - Embeddings cached in `cache/` with hash/size keys.

6. **Metric computation (3 hops + E2E)**
   - Hop A: `Type_1 -> Type_5_1st`
   - Hop B: `Type_5_1st -> Type_5_2nd`
   - Hop C: `Type_5_2nd -> Type_5_3rd`
   - E2E: `Type_1 -> Type_5_3rd`
   - Metrics: Jaccard, ROUGE-L, METEOR, SBERT cosine/euclidean, edit distance.
   - Composite SDS = weighted normalized blend:
     - SBERT: `0.6`
     - METEOR: `0.2`
     - ROUGE-L: `0.2`

7. **Validation + artifact classification**
   - Sanity/range checks and correlation checks.
   - Classifies zero-overlap cases into:
     - `extreme_boiling_frog`
     - `generation_artifact`
   - Filters generation artifacts before core analysis.

8. **Core analysis + visualizations**
   - Boiling Frog 3-hop panel.
   - Cross-model comparison.
   - Cross-domain comparison.
   - Domain × model heatmap and metric correlation heatmap.

9. **Cross-dataset hyper view**
   - Compares this multi-model dataset against PADBen output.
   - Expects `../BoilingFrog_SemanticDrift_Pipeline/outputs/drift_results.csv` (with fallback paths in notebook).

10. **Export & reproducibility**
    - Writes final CSV/JSON summaries plus figures.

## Key configuration defaults

From `PipelineConfig`:

- `data_dir`: `./output_json`
- `output_dir`: `./outputs`
- `cache_dir`: `./cache`
- `log_dir`: `./logs`
- `sbert_model`: `all-mpnet-base-v2`
- `sbert_batch_size`: `64`
- `parascore_threshold (gamma)`: `0.35`
- Severity boundaries: `low_med=0.35`, `med_high=0.45`
- Artifact thresholds: `rouge<=0.05`, `sbert<=0.10`
- `seed`: `42`

## How to run

1. Open `BoilingFrog_MultiModel_Pipeline.ipynb`.
2. Place multi-model JSON data under `output_json/` (or update `data_dir`).
3. Run Section `0A` once (dependency install), then run all cells top-to-bottom.
4. For cross-dataset section, run the semantic pipeline first (or update fallback paths).

Dependency install command used by the notebook:

```bash
pip install sentence-transformers nltk rouge-score bert-score scipy scikit-learn seaborn tqdm python-Levenshtein pydantic ftfy watermark
```

## Main outputs

Typical exports from a full run:

- `outputs/drift_results.csv`
- `outputs/pipeline_summary.json`
- `outputs/data_quality_report.csv`
- `outputs/validation_failures.csv` (when invalid rows exist)
- `outputs/extreme_boiling_frog_cases.csv`
- `outputs/generation_artifacts.csv`
- Figures such as:
  - `fig_token_distributions.png`
  - `fig_domain_model_distribution.png`
  - `fig_boiling_frog_3hop.png`
  - `fig_cross_model_comparison.png`
  - `fig_cross_domain_comparison.png`
  - `fig_domain_model_heatmap.png`
  - `fig_correlation_heatmap.png`
  - `fig_cross_dataset_comparison.png`
  - `fig_padben_per_source.png`

## Notes

- The notebook expects a fairly large corpus; embedding cache is important for reruns.
- `pipeline_summary.json` captures runtime, config snapshot, domain/model coverage, and aggregate SDS findings.
