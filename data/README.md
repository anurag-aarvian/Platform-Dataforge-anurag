# Synthetic FMCG distribution data, India, 2005–2024

A fully synthetic, labelled demand dataset generated with **Aarvian DataForge** (engine 0.2.0). It simulates 20 Indian FMCG distribution companies over 20 years, then records how each company's own systems would have captured that world. The true cause of every number is stored next to it.

No real person's or customer's record is in this data. Public context data (World Bank macro indicators) calibrates the economy; see [Attribution](#attribution).

## Folder map

```
data/
├── README.md                                  this file: design and contents
├── scenario_fmcg_distribution_india_20y.yaml  the spec that produced everything
├── fmcg_distribution_india_20y-ebbe4bef19c8/  full native dataset (truth, observed, logs, events, notes, market, report)
└── exports/                                   the same data in every format DataForge can export
    ├── README.md                              format-by-format guide
    ├── forecasting/                           ready-to-load ML layouts
    ├── enterprise_systems/                    emulated ERP / accounting extracts (3 sample companies each)
    └── company_files/                         each company's own files, one zip per company
```

## 1. Scope

| Item | Value |
|---|---|
| Sector template | `fmcg_distribution` (the only template that ships with DataForge) |
| Period | 2005-01-01 to 2024-12-31, daily grain, 7,305 days |
| Country | India only (currency INR) |
| Companies | 20: 1 large, 6 mid-sized, 13 small |
| Series | 2,323 product × location series |
| Values | 16,969,515 daily values; 310.5 M units demanded; 18.6 M units lost to stock-outs |
| Seed / engine | seed 20261108, engine 0.2.0, spec hash `599dbbb622b5` |
| Content hash | `8e910071d0968162…` (see `manifest.json`) |

Reproducibility: a dataset is a pure function of (spec, seed, engine version). Re-running `scenario_fmcg_distribution_india_20y.yaml` with the same engine version should reproduce the dataset. I have not re-run it to confirm.

## 2. How the data was made (design)

```
spec → compile → worlds (one per company) → simulate truth → observe → notes → validate
```

1. **World first.** Each company is its own world with its own seed: products (English and Hindi names, price, elasticity, pack size, shelf life, lifecycle), locations (region, channel format, urbanisation, climate zone) and suppliers (lead time, reliability, minimum order). Prices are in INR.
2. **Truth layer.** True demand is simulated once, from a statistical base texture multiplied by shared causal drivers and business behaviours. Every stage is stored as an additive contribution, so `y = base + trend + texture + seasonality + calendar + weather + macro + lifecycle + price_effect + promo + cannibalisation + event_shock + noise` holds for every value.
3. **Stock.** A periodic-review (s, S) inventory simulation with supplier lead times, partial fills and disruptions turns demand into recorded sales. `sales = y − lost_sales`, so censoring (stock-outs) comes from the simulated world rather than from a mask.
4. **Observation layer.** Each company's data maturity decides how it records sales. Artefacts such as month-end batching, late postings, manual corrections, duplicate invoices, migration gaps, partial months and SKU renames are applied and every one is logged in `corruption_log.parquet`. Applying the log to the postings recovers recorded sales exactly.
5. **Context notes.** Planner notes in English, Hindi and Hinglish are written from the event log and linked to their event and true effect. A noise policy makes some vague, some wrong in magnitude and some about events that never happened. All text is template-based; no AI phrasing was applied to this run.
6. **Validation.** Every run produces `validation.json` (see section 7).

### Causal drivers
- **Calendar:** each country's official holidays (the `holidays` library, v0.108), classified into kinds (festival, closure and so on), plus payday and financial-year effects.
- **Weather:** climate by region with seasons and heatwave events.
- **Macro:** `calibrated` to the bundled World Bank reference (GDP growth, inflation, lending rate, exchange rate; CC BY 4.0). All 2005–2024 values are inside the published range, so every market row is labelled `calibrated`.
- **Shocks:** demonetisation-like (anchor 2016-11-08), lockdown-like (anchor 2021-04-15) and a commodity spike.
- **Business behaviours:** price elasticity (normal, mean −1.4, sd 0.5), promotions with a post-promo dip (2–12 a year, lift 1.2–3.0), cannibalisation within substitute groups, launches and end of life, intermittent demand (35% share), recalls, supply disruptions and competitor entry.

### Companies
| Size | Count | | System | Count |
|---|---|---|---|---|
| large | 1 | | Tally Prime | 14 |
| mid | 6 | | Excel workbooks | 4 |
| sme | 13 | | SAP S/4 | 2 |

Observation profiles: 14 typical mid-market, 4 spreadsheet SME, 2 clean enterprise. Regions: NCR, UP-West, Punjab, Rajasthan, Maharashtra, Karnataka, Tamil Nadu, West Bengal.

## 3. What is in the data

Series mix (2,323 series):

| Dimension | Breakdown |
|---|---|
| Category | staples 480, snacks 338, beverages 321, home care 315, personal care 270, dairy 222, ice cream 211, sweets 166 |
| Channel format | general trade 1,180, modern trade 512, cash and carry 335, depot 296 |
| Lifecycle | mature 1,859, launch 312, end of life 152 |
| Demand class (ADI / CV²) | erratic 707, lumpy 608, intermittent 563, smooth 445 |
| Region | Tamil Nadu 555, Maharashtra 415, UP-West 404, Rajasthan 330, Karnataka 240, Punjab 163, NCR 147, West Bengal 69 |

Overall, 37.6% of one sampled company's daily demand values are zero, and promotions are on for 18.2% of its series-days (checked on C00001 only). The median series averages about 3.9 units a day.

### Events (`events.parquet`, 60,556 rows)
Each event has a type, the component it acts through, start and end, scope (products, regions, categories) and its isolated true effect in units and percent.

| Type | Count | Median true effect |
|---|---|---|
| promotion | 46,279 | +39% |
| price change | 11,421 | −6% |
| festival | 1,600 | +9% |
| supply disruption | 497 | −14% |
| heatwave | 240 | +15% |
| viral spike | 203 | +68% |
| product recall | 109 | −66% |
| launch | 68 | −50% |
| macro shock | 60 | −11% |
| end of life | 36 | −75% |
| competitor entry | 23 | −10% |
| macro regime | 20 | −2% |

### Observed postings and corruptions
10,305,281 postings; 684,754 logged corruptions: month-end batching 432,962, late posting 167,018, manual correction 40,784, duplicate invoice 26,414, migration gap 12,326, partial month 2,899, censoring 2,323, SKU rename 28.

### Notes (`notes.jsonl`, 13,907 rows)
Languages: Hinglish 4,649, English 4,631, Hindi 4,627. Voices: chat 5,183, email 4,305, minutes 4,268, news 151. Accuracy labels: exact 8,197, vague 3,177, wrong magnitude 1,503, phantom (event never happened) 1,030.

### Market data (`market.parquet`, 7,305 rows)
Daily India table: GDP index (100 → 356.5), CPI index (100 → 392.7), interest rate %, exchange rate per USD (44.0 → 79.0), equity index, commodity index, regime (6,939 expansion days, 366 recession days) and `origin` (all `calibrated`). Equity and commodity indices are synthetic by design.

## 4. Native dataset layout: `fmcg_distribution_india_20y-ebbe4bef19c8/`

| Path | Contents |
|---|---|
| `manifest.json` | scenario, seed, engine and generator versions, hashes, stats, company list, labels |
| `spec.yaml` | the exact spec used |
| `validation.json` | release gate, checks, coverage maps, suggested spec patches |
| `series.parquet` | one row per series: category, region, format, lifecycle, base level, stock-out days, totals, demand class |
| `features.parquet` | 12 time-series features per series (ADI, CV², ACF1, seasonality, trend strength, entropy, lumpiness, spikiness and others) |
| `events.parquet`, `notes.jsonl`, `market.parquet` | all companies combined |
| `ATTRIBUTION.md` | public-data attribution |
| `worlds/C000NN/truth.parquet` | daily truth with all components (see 4.1) |
| `worlds/C000NN/observed/year=YYYY/` | observed postings, one folder per year |
| `worlds/C000NN/corruption_log.parquet` | every observation artefact with true and observed quantity |
| `worlds/C000NN/{products,locations,suppliers,series,events}.parquet`, `notes.jsonl` | per-company master data |

Companies C00004, C00010 and C00018 have `truth.parts/part-NNN.parquet` instead of one `truth.parquet`, because GitHub rejects files over 100 MB. The parts have identical rows in the original order. Concatenate them to use DataForge's own reader.

### 4.1 `truth.parquet` columns
| Column | Type | Meaning |
|---|---|---|
| `unique_id` | category | `C00001/P0001/L01` = company / product / location |
| `ds` | datetime | day |
| `y` | int32 | true demand |
| `sales` | int32 | recorded sales (demand capped by stock) |
| `lost_sales` | int32 | `y − sales` |
| `stock_eod` | int32 | end-of-day stock |
| `mu` | float32 | expected demand before noise |
| `price`, `regular_price` | float32 | selling price and list price (INR) |
| `on_promo` | bool | promotion running |
| `base … noise` (13 columns) | float32 | additive components that sum to `y` |

Checked on C00001: components sum to `y` within 2.3e-4 (float32 storage error), `sales + lost_sales = y`, `sales ≤ y` and stock is never negative.

## 5. Exports

See [`exports/README.md`](exports/README.md) for each format, its grain, columns and which companies it covers.

## 6. Known limitations
- **Uncalibrated priors.** Template numbers are not calibrated against a licensed real reference. Use this data to build and test pipelines, not to quote accuracy.
- **One country, one sector,** daily grain only. No hourly layer.
- **Coverage targets are hand-set placeholders,** so the 71.4% coverage figure is indicative.
- **Enterprise-system layouts are emulations** modelled on public vendor documentation, not official exports, and cover 3 sample companies (C00001, C00008, C00009).
- **Observed-layer ML exports needed a reader patch.** DataForge's `observed_years` failed with `KeyError: 'doc_no'` when asked for columns without `doc_no`. I patched it locally so the two observed-layer exports could be produced. Generated data is unaffected, and the patch is not in this repository.
- **Not evaluated:** fidelity to real data, discriminative realism, train-synthetic-test-real usefulness and AI note phrasing.

## 7. Validation summary
Release gate: **pass**, score 100 over the checks that were evaluated.

| Dimension | Check | Result |
|---|---|---|
| Causal correctness | Components reproduce every value | pass |
| | Recorded sales never exceed stock | pass (0 violations) |
| | Promotions lift only inside their window | pass (0) |
| | Corruption log recovers the truth | pass (0 mismatches) |
| | Notes state only the magnitudes their label allows | pass (0 failures) |
| | Model-phrased notes keep every number | not evaluated (phrasing off) |
| Diversity | Near-duplicate series rate | pass (0.0) |
| Coverage | Template feature-space coverage | pass (0.7143, threshold 0.6) |
| Statistical fidelity, discriminative realism, usefulness (TSTR) | | not evaluated |
| Memorisation | No copying of real data | pass (asserted by design: no real series read) |

## Attribution
Calibrated with the World Bank World Development Indicators (CC BY 4.0) for India 2005–2024. Holiday calendars come from the `holidays` library (MIT). Full details are in `ATTRIBUTION.md` in the native dataset and in every export zip.
