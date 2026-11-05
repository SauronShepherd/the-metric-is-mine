# Validation record

Validated on 2026-11-05 with Databricks CLI 1.22.1 in a private development workspace using a serverless/Pro SQL Warehouse. The first post-refactor run caught a real parser bug in the parameterized relationship target; the target was changed to a local identifier after setting the requested catalog and schema explicitly. The corrected runs (`757932730375855`, `975807630188710`) and the latest flat-layout run (`831883261582261`) all completed successfully, so the `CREATE OR REPLACE`, parameter passing, materialization, date check, relationship, and root-level notebook paths are repeatable in this workspace.

- Bundle validation: `Validation OK!`
- Bundle deployment: one job updated; the latest flat-layout edit uploaded the four root-level notebook files and removed the old nested copies.
- Repository layout: the four notebooks and `metric-views-labs.yml` are now at the repository root; bundle paths and article links point to those flat paths.
- Job run: `831883261582261` completed successfully
- Result: `TERMINATED / SUCCESS`
- Tasks: `lab_01_build_sales_metric_view`, `lab_02_joins_and_fanout`, `lab_03_windows_and_materialization`, and `lab_04_recent_features` all succeeded.
- Notebook parameters: every task received `catalog=workspace` and `schema=metric_views_lab`; the checked-in notebooks contain no workspace-specific object names.
- Lab 3 preflight: the notebook now queries `MIN(o_orderdate)`, `MAX(o_orderdate)`, and source row count before applying its window filter.

## Expected demonstrations

| Check | Observed result | Interpretation |
|---|---:|---|
| Orders vs. distinct order keys | `7,500,000 = 7,500,000` | The order source is one row per order. |
| Duplicate customer keys | `0` | The `rely.at_most_one_match` assumption holds for the sample dimension. |
| Direct AOV vs. Metric View AOV | `151091.892028479486 = 151091.892028479486` | The composed numerator/denominator measure agrees with the SQL control. |
| AOV reconciliation query | `5 segments; 0 mismatches` | The notebook now performs the comparison in one query instead of relying on two result panes and optimism. |
| Filtered fulfilled revenue | `32,694,715,919.99 = 32,694,715,919.99` | A measure-level `FILTER (WHERE ...)` selects fulfilled orders without removing other statuses from the model. |
| Correct header total | `1,133,439,215,246.25` | The order-header economic concept. |
| Naive header total after line-item join | `5,667,029,083,716.96` | Fanout inflates the repeated header value, as explained in Lab 2. |
| One-to-many header controls by status | `F/O/P` match direct SQL exactly | Header revenue and order counts stay at header grain while line revenue remains a separate fact measure. |
| One-to-many line-price control | `3 statuses; 0 mismatches` | The line-item aggregate agrees with a direct `SUM(l_extendedprice)` grouped by order status. |
| Rolling-window sample | `0, 3137, 6196, 9264, 12383` | The first dates show the seven-day customer window changing by date. |
| Twelve-month offset control | `44 months; 0 mismatches` | `offset: -12 month` agrees with a hand-written `ADD_MONTHS(..., -12)` control wherever a prior month exists. |
| Metric View relationship metadata | `orders_to_customers / RELATIONSHIP / enforced NO` | The Beta relationship is discoverable metadata and is not an enforced uniqueness constraint. |
| Materialized monthly Metric View | `latest_refresh_status: Succeeded` | The managed materialization refreshed successfully. |

The materialized view reported `metric_view.materialization.enabled=true`, `mode=relaxed`, and `latest_refresh_status: Succeeded`; `EXPLAIN EXTENDED` also showed a Photon Parquet scan of the generated `revenue_by_month` materialization relation. These checks are intentionally direct SQL controls rather than screenshots, so they can be repeated after a source or runtime change.
