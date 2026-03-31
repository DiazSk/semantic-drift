# Paper Errata — Required Changes in LaTeX Source

These changes must be applied to the LaTeX source of `COLM_2026__Semantic_Paraphrasing.pdf`
before submission. They were identified by a systematic verification of every claim in the
paper against the pipeline outputs in `BoilingFrog_SemanticDrift_Pipeline/` and
`BoilingFrog_MultiModel_Pipeline/`.

---

## CRITICAL

### C1. Table 1 — Ship of Theseus Numbers Are Wrong

The entire Ship of Theseus column in Table 1 must be recomputed. Run the new cell
`2A.1: PAPER TABLE 1` added to `BoilingFrog_MultiModel_Pipeline.ipynb` (cell 13)
and replace the Ship of Theseus row with the output values. The current numbers
(Mean Paraphrase = 268.45, Min Source = 45.00, Max Source = 892.00, etc.) do not
match the pipeline data (actual Mean Paraphrase = ~186.1, Min Source = 6, Max Source = 5764).

Only Source Mean (273.8) is correct. All other cells are wrong.

Also update the Table 1 caption description: if "Source and paraphrase refer to the
first and final texts in each chain" remains the definition, the numbers must reflect
Type_1 vs Type_5_3rd statistics after artifact filtering.

### C2. Section 5.2 — Statistical Tests Must Be Updated

Run the new cells `7A.1: STATISTICAL SIGNIFICANCE TESTS` added to both notebooks
and update the paper's Section 5.2 with the actual computed values. The current
paper claims:

- PADBEN: t = 20.17, p < 10^-89, Cohen's d = 0.16
- Ship of Theseus Hop A > B: t = 213.15, p < 10^-300, d = 1.49
- Ship of Theseus Hop B > C: t = 31.95, p < 10^-218, d = 0.22

These must be replaced with the actual values from the new pipeline outputs in
`outputs/statistical_tests_deceleration.csv` in each pipeline directory.

### C3. Section 5.2 — Figure Cross-Reference Error

**Current text (page 5):**
> "in Ship of Theseus the deceleration is more pronounced: 0.603 → 0.292 → 0.263 (Figure 2)."

**Fix:** Change `(Figure 2)` to `(Figure 3)`.

Figure 2 is captioned "PADBEN: Hop-by-hop vs. cumulative SDS..." while Figure 3
shows the Ship of Theseus data. The Ship of Theseus deceleration reference should
point to Figure 3.

### C4. Section 5.3 — Correlation Values Must Be Verified

Run the updated correlation heatmap cells (7G in MultiModel, 7D in SemanticDrift)
which now export `spearman_correlation_matrix.csv`. Verify that the specific rho
values cited in Section 5.3 match the exported matrix:

- Jaccard <-> METEOR rho = 0.902
- ROUGE-L <-> METEOR rho = 0.807
- SBERT rho range = 0.535–0.646
- SDS <-> SBERT rho = -0.902
- SDS <-> METEOR rho = -0.867

If any value differs, update the paper text accordingly.

---

## MODERATE

### M1. "PADBen" vs "PADBEN" Capitalization

The original dataset citation (Zha et al., 2025) uses "PADBen". The paper body
uses "PADBEN" throughout. Choose one form and apply it consistently:

- If using "PADBEN" (as an acronym): update the bibliography entry to also say "PADBEN"
- If using "PADBen" (matching original): find-and-replace "PADBEN" → "PADBen" throughout

### M2. Re-Run Pipelines for Consistent Outputs

The `extreme_boiling_frog_cases.csv` in MultiModel has 430 data rows, but the
notebook stdout and paper both report 426. This 4-row discrepancy indicates the
CSV is from a different run. Both pipelines should be re-run end-to-end to
produce a single consistent set of outputs, and all paper numbers should be
verified against that run.

### M3. PADBEN Hop B Spans Two Paraphrasing Iterations

PADBEN Hop B is defined as T5-1st → T5-3rd, which spans two actual paraphrasing
iterations (1st → 2nd → 3rd). Hop A (T2 → T5-1st) spans one iteration. This
asymmetry should be acknowledged in Section 4.2 or 4.3. Add a brief note such as:

> "Note that PADBEN Hop B spans two paraphrasing iterations (1st to 3rd), as the
> intermediate 2nd iteration is not available in this dataset."

### M4. Table 1 Caption — Approximate Record Count

Table 1 caption says "≈21,000 records" for Ship of Theseus. The precise post-filter
count is 20,595. Change to "20,595 records" for consistency with the rest of the paper.

### M5. Appendix A — Standard Deviation Table Is Empty

The appendix references "Full hop-by-hop and cumulative standard deviation values"
but contains no table. Run the new cells `7I: APPENDIX A` (MultiModel) and
`7E: APPENDIX A` (SemanticDrift) to generate `appendix_a_stddev_table2.csv` in
each pipeline, then populate the appendix with those values.
