# Exercise 12 EV-Style Harmonized-HS6 Expansion Adversarial Review

Created: 2026-06-02

Review status: local adversarial review, not an independent fresh-agent pass. The available callable tools in this turn did not provide a fresh reviewer agent. The review uses the project `adversarial-econometrics-review`, `empirical-reporting-replication`, `econ-literature-review`, and `econ-ai-data-source-planner` guidance.

## Executive Verdict

Verdict: Mostly trustworthy for the current descriptive HS6 appendix, not yet trustworthy as a full Lukaszuk-Torun / Harvard Growth Lab weighted harmonization.

- The rerun completes on the full `rd2_countries` Exercise 12 source: 60 countries and 161,654,903 filtered product-partner rows.
- Product-dependent filters are respected: HS6 `999999` and `partnerCode == 0` are both zero after filtering.
- The accounting checks pass: no duplicate product, partner, or combined channel keys; no product or combined accounting residual violations.
- The result is materially different from HS4 and therefore useful: horizon-5 persistent new harmonized-HS6 families account for 43.6% of pooled positive expansion, versus 3.0% at HS4.
- The main limitation is measurement: 21.2% of filtered product-trade value is linked but ambiguous under the current WCO-family map. Lukaszuk-Torun / HGL-style weights are needed to convert that part probabilistically instead of leaving oversized components revision-specific.

## Highest-Risk Findings

1. Severity: Medium to High. The run cites Lukaszuk-Torun and inspects the Harvard Growth Lab generator, but does not use generated LT/HGL conversion weights.
   - What happened: `scripts/run_exercise_12_ev_hs6_harmonized_expansion.py` uses the repository's WCO adjacent-revision harmonized-family lookup. The HGL repository was inspected at commit `503c5d89287fdba5f9af8cb7482ba344b8da7cea`, but no precomputed optimized weights were available in the public clone.
   - Why it matters: The paper can say the rerun is motivated by Lukaszuk-Torun and HGL, but not that it has solved many-to-many HS revision ambiguity with their weighted algorithm.
   - How to verify: See `run_manifest_exercise_12_ev_hs6_harmonized_expansion.json`, field `lt_hgl_weighted_conversion_status`, and `hgl_local_inspection.precomputed_optimized_weights_present`.
   - Fix or next diagnostic: Generate conversion weights using the HGL pipeline with raw Comtrade data, R, MATLAB, and an API key; then rerun a truly LT/HGL-weighted HS6 decomposition.

2. Severity: Medium. Ambiguous HS revision components are large enough to affect product-entry interpretation.
   - What happened: Validation reports clean non-ambiguous value share 78.8%, ambiguous value share 21.2%, unmatched value share 0.0%.
   - Why it matters: A 43.6% new-HS6-family positive-expansion share may partly reflect HS revision splits/merges in ambiguous components. Some countries have very high ambiguity, including Ireland and Hong Kong during the run log.
   - How to verify: See `ev_hs6_harmonized_validation.json` and per-country checkpoint `source_check` files under `data/processed/samples/rd2_countries/exercise_12_ev_hs6_harmonized_checkpoints/`.
   - Fix or next diagnostic: Add country-level ambiguity exposure to the published table and run a drop-ambiguous sensitivity before leaning on country examples.

3. Severity: Low to Medium. Partner-spread shares use two denominators.
   - What happened: As in the HS4 runner, partner-spread rows retain product-level positive-expansion shares and also report within-continuing-product shares.
   - Why it matters: Raw partner-spread shares need not sum to one. A validation check that uses the wrong denominator will flag a false failure.
   - How to verify: In `ev_hs6_harmonized_pooled_summary.csv`, `partner_spread_continuing_hs6_harmonized` rows should be checked with `pooled_within_positive_expansion_share`.
   - Fix or next diagnostic: Any website or memo text should label the partner-spread denominator explicitly.

4. Severity: Low. 2025 source rows are not used because the deflator file has no usable 2025 value.
   - What happened: Validation reports `source_years_without_deflator: [2025]` and `deflator_missing_years_used: []`.
   - Why it matters: Windows requiring 2025 real values are omitted. This matches the HS4 EV-style run and avoids silently using nominal 2025 values.
   - How to verify: See `ev_hs6_harmonized_validation.json`.
   - Fix or next diagnostic: Update the World Bank deflator file when 2025 is available and rerun.

## Data Lineage And Sample Audit

- Source aggregate: `data/processed/samples/rd2_countries/exercise_12_export_aggregates.parquet`.
- Product identity: `hs6_harmonized_family` from repository WCO adjacent-revision harmonized families.
- Output script: `scripts/run_exercise_12_ev_hs6_harmonized_expansion.py`.
- Processed output: `data/processed/samples/rd2_countries/exercise_12_ev_hs6_harmonized_expansion_decomposition.parquet`.
- Main output directory: `results/samples/rd2_countries/exercise_12_ev_hs6_harmonized_expansion_tables/`.
- Country count: 60 of 60 expected `rd2_countries`.
- Source rows after filters: 161,654,903.
- HS6 `999999` rows after filtering: 0.
- `partnerCode == 0` rows after filtering: 0.
- Country-window rows: 10,816.
- Partner-spread country-window rows: 10,796.
- Combined country-window rows: 35,152.
- Latest 5-year country rows: 60.

## Merge/Join Audit

- Deflator join is many trade rows to one year. Missing 2025 deflator rows are reported and not used.
- HS6 harmonization join is many trade rows to one `(classification_code, cmd_code)` lookup row.
- No many-to-many merge is used in the final channel construction.
- Duplicate output keys are zero for product, partner-spread, and combined panels.
- The HGL repository is not merged into the trade data; it is recorded as an inspected method reference.

## Variable Construction Audit

- `trade_value_2024_usd` equals nominal trade value times the U.S. GDP-deflator factor to 2024.
- Product ID is current repository `hs6_harmonized_family`, not raw revision-specific HS6 and not HS4.
- Base window is `t,t+1`; future window is `t+h,t+h+1`.
- Base-active and future-active require both annual values to be at least $50,000 in constant 2024 USD.
- Positive expansion is `max(future_two_year_avg - base_two_year_avg, 0)`.
- Net contribution is `future_two_year_avg - base_two_year_avg`.
- Product channels are mutually exclusive: new, continuing, dying, below-threshold residual.
- Combined product-first and partner-first panels are alternative partitions and must not be added together.

## Specification Audit

There is no regression. This is descriptive accounting:

`delta_c,t,h,p = future_avg_c,t,h,p - base_avg_c,t,p`

The main share is:

`sum positive_expansion over channel / sum positive_expansion over all channels`

For horizon 5, the main harmonized-HS6 product pooled positive-expansion shares are:

- New harmonized-HS6 families: 43.6%.
- Continuing harmonized-HS6 families: 49.2%.
- Dying harmonized-HS6 families: 0.2%.
- Below-threshold residual: 7.0%.

For horizon 5, the product-first combined pooled positive-expansion shares are:

- Net-new harmonized-HS6 families: 36.2%.
- Continuing HS6 families to new product-specific partners: 7.9%.
- Continuing HS6 families to existing product-specific partners: 47.4%.
- Other residual/lost/dying channels: remaining share.

## Inference And Identification Audit

No causal inference, standard errors, clustering, or fixed effects are estimated. The credible claim is descriptive: under strict EV-style windows and the current WCO-family HS6 map, fine-grained product entry matters much more than it does at HS4. The identification risk is measurement, not omitted-variable bias.

## Replication Checklist

Run:

`/opt/anaconda3/bin/python -m py_compile scripts/run_exercise_12_ev_hs6_harmonized_expansion.py`

`/opt/anaconda3/bin/python -m unittest tests.test_exercise_12_ev_hs4_expansion tests.test_exercise_12_extensive_margin`

`OMP_NUM_THREADS=2 OPENBLAS_NUM_THREADS=2 /usr/bin/nice -n 8 /opt/anaconda3/bin/python scripts/run_exercise_12_ev_hs6_harmonized_expansion.py --country-sample rd2_countries --horizons 5 10 --fresh`

Then check:

- `ev_hs6_harmonized_validation.json` has `status: ok`.
- Product and combined pooled positive-expansion shares sum to one by horizon.
- Partner-spread rows sum to one only under `pooled_within_positive_expansion_share`.
- `latest_5y_country.csv` has 60 unique reporters.
- Manifest records the Lukaszuk-Torun citation and HGL generator inspection.

## Minimal Patch Plan

No blocking patch is needed for an appendix use. Before website publication, add the HS6 table as a clearly labeled appendix and include:

1. HS4 versus HS6 comparison table.
2. The 78.8% clean / 21.2% ambiguous harmonization diagnostic.
3. A country-level ambiguity warning column.
4. A statement that full LT/HGL weighted conversion remains future work unless generated weights are added.

## Questions For The Researcher

1. Should the paper present HS4 as the conservative headline and harmonized HS6 as the fine-grained appendix, or move HS6 into the main text?
2. Should we run a drop-ambiguous sensitivity before using Ireland, Hong Kong, Singapore, Switzerland, or Sweden as examples?
3. Do you have access to the raw Comtrade/HGL weight-generation prerequisites needed for a full LT/HGL weighted rerun?
