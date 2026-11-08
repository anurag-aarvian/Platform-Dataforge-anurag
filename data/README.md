# FMCG distribution, India, 2005-2024 (synthetic)

Generated with Aarvian DataForge engine 0.2.0, seed 20261108, from `scenario_fmcg_distribution_india_20y.yaml`.
20 companies, 2,323 series, daily grain, about 17 million values. Release gate: pass.

- `fmcg_distribution_india_20y-ebbe4bef19c8/`: full dataset (truth, observed records, corruption logs, events, notes, market data, validation report, manifest)
- `fmcg_20y_weekly_nixtla.zip`: weekly truth layer in Nixtla format (`unique_id, ds, y, sales, price, on_promo`)

Fully synthetic. Fidelity, discriminative realism and TSTR checks are not evaluated. Template numbers are uncalibrated priors.

Note: Git LFS is not used. For GitHub's 100 MB file limit, `truth.parquet` of companies C00004, C00010 and C00018
is split into `truth.parts/part-NNN.parquet` (row-for-row identical, in original order, zstd compressed).
Read with `pyarrow.dataset.dataset("worlds/C00010/truth.parts")` or `pandas.concat` over the parts. DataForge's own
reader expects a single `truth.parquet`; concatenate the parts into one file to use it on those worlds.
