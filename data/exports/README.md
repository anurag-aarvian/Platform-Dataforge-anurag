# Exports

The same synthetic FMCG data as the native dataset, written by DataForge's 14 export writers. See [`../README.md`](../README.md) for how the data was made and what it contains.

Every zip carries `ATTRIBUTION.md` (World Bank data, CC BY 4.0). Zips are used because DataForge's single-file writers switch to a zip whenever public data is involved.

## `forecasting/`: ML layouts

Weekly grain starts on Monday (the first week begins 2004-12-27 and contains 2005-01-01). `truth` is the simulated demand. `observed` is what each company recorded, so it carries stock-outs and the observation artefacts.

| Folder | Layer, grain | Companies | Contents |
|---|---|---|---|
| `nixtla_truth_week/` | truth, week | all 20 | CSV `unique_id, ds, y, sales, price, on_promo` for StatsForecast, MLForecast and NeuralForecast |
| `nixtla_observed_week/` | observed, week | all 20 | CSV `unique_id, ds, y`; ids are the company's own `sku@location` codes |
| `darts_truth_week/` | truth, week | all 20 | CSV `unique_id, time, y, sales, price, on_promo, company_id, category, region, format`; load with `TimeSeries.from_group_dataframe(group_cols="unique_id")` |
| `gluonts_truth_week/` | truth, week | all 20 | JSON Lines, one series per line: `start`, `target`, `item_id`, static categories, price and promotion as dynamic features |
| `m5_truth_day/` | truth, **day** | all 20 | M5 competition layout: `sales_train.csv` (wide), `calendar.csv`, `sell_prices.csv` |
| `parquet_decomposition_truth_week/` | truth, week | all 20 | Parquet panel with every component, stock and lost sales. Stored as `parts/part-001.parquet`, `part-002.parquet` (GitHub's 100 MB limit); concatenate in order. Row-count checked against the original. |
| `parquet_decomposition_observed_week/` | observed, week | all 20 | Parquet panel of observed demand |

## `enterprise_systems/`: emulated system extracts

Observed layer, daily, **only companies C00001 (Tally profile, typical mid-market), C00008 (clean enterprise, SAP) and C00009 (spreadsheet SME)** — one per observation profile, to keep the repository small. One folder per company inside each zip, one file per year. These are emulations modelled on public vendor documentation, for testing ingestion pipelines. They are not official exports.

| Folder | System and layout | Files per company |
|---|---|---|
| `sap_s4/` | SAP S/4HANA billing: `VBRK_YYYY.csv` (headers), `VBRP_YYYY.csv` (items), `MARA_material.csv`, `T001W_plant.csv`; semicolon-separated | per year + 2 master files |
| `oracle_fusion/` | Oracle Fusion AutoInvoice interface: `RA_INTERFACE_LINES_ALL_YYYY.csv` | per year |
| `dynamics365/` | Dynamics 365 invoice lines: `SalesInvoiceLines_YYYY.csv` | per year |
| `netsuite/` | NetSuite saved search: `TransactionSearch_Invoices_YYYY.csv` | per year |
| `odoo/` | Odoo `account.move.line`: `account.move.line_YYYY.csv` | per year |
| `shopify/` | Shopify orders export: `orders_export_YYYY.csv` | per year |
| `tally_xml/` | Tally Prime sales vouchers: `Tally_Sales_Vouchers_YYYY.xml` (XML import envelope) | per year |

## `company_files/`: each company's own files

Observed layer, daily, **all 20 companies**, one zip per company (the single DataForge export was 312 MB and was repacked per company for GitHub). Each company's format follows how it keeps its data:

- Tally companies: `Tally_Sales_Register_FY_YYYY-YY.xlsx` (financial-year workbooks, Indian number grouping, Hindi item names)
- SAP companies (C00008, C00010): `SAP_VBRP_extract_YYYY.csv`
- Spreadsheet companies (for example C00009): `bills_export_YYYY.csv`

Larger companies are about 70 MB per zip (C00004, C00018); others are much smaller.

## Provenance

All files were exported from the same dataset (content hash `8e910071d0968162…`) with `DatasetReader.export_to` from DataForge engine 0.2.0. The two observed-layer ML exports (`nixtla_observed_week`, `parquet_decomposition_observed_week`) required a local one-line fix to the reader (`observed_years` did not read the `doc_no` column it sorts by). The fix is not in this repository and does not change generated data.
