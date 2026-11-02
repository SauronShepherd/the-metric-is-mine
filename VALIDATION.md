# Validation record

Validated with Databricks CLI 1.22.1 in a private development workspace using a serverless/Pro SQL Warehouse.

- Bundle validation: `Validation OK!`
- Bundle deployment: one job created, six bundle files uploaded.
- Job run: completed successfully
- Result: `TERMINATED / SUCCESS`
- Tasks: `lab_01_build_sales_metric_view`, `lab_02_joins_and_fanout`, and `lab_03_windows_and_materialization` all succeeded.

## Expected demonstrations

| Check | Observed result | Interpretation |
|---|---:|---|
| Orders vs. distinct order keys | `7,500,000 = 7,500,000` | The order source is one row per order. |
| Duplicate customer keys | `0` | The `rely.at_most_one_match` assumption holds for the sample dimension. |
| Direct AOV vs. Metric View AOV | `151091.892028479486 = 151091.892028479486` | The composed numerator/denominator measure agrees with the SQL control. |
| Correct header total | `1,133,439,215,246.25` | The order-header economic concept. |
| Naive header total after line-item join | `5,667,029,083,716.96` | Fanout inflates the repeated header value, as explained in Lab 2. |
| One-to-many header controls by status | `F/O/P` match direct SQL exactly | Header revenue and order counts stay at header grain while line revenue remains a separate fact measure. |
| Rolling-window sample | `0, 3137, 6196, 9264, 12383` | The first dates show the seven-day customer window changing by date. |
| Materialized monthly Metric View | `latest_refresh_status: Succeeded` | The managed materialization refreshed successfully. |

The materialized view reported `metric_view.materialization.enabled=true` and `mode=relaxed`; `EXPLAIN EXTENDED` also showed a Photon Parquet scan of the generated `revenue_by_month` materialization relation. These checks are intentionally direct SQL controls rather than screenshots, so they can be repeated after a source or runtime change.
