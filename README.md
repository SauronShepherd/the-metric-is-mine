# The Metric is Mine — Databricks labs bundle

This repository packages the three hands-on notebooks from the **The Metric is Mine** article as a Databricks Declarative Automation Bundle.

The bundle deploys one serial job with three SQL-only notebook tasks:

1. Build and query a sales Metric View, including an AOV control, parameters, and definition inspection.
2. Reproduce join fanout, model a one-to-many relationship, and reconcile the result with direct SQL controls.
3. Query a rolling window, review semiadditive balance syntax, and optionally create a materialized monthly aggregate.

## Requirements

- Databricks CLI 1.22 or newer.
- A configured CLI profile with access to the target workspace.
- A serverless or Pro SQL Warehouse. Notebook tasks in this bundle contain SQL cells only.
- `SELECT` on `samples.tpch.orders`, `samples.tpch.customer`, and `samples.tpch.lineitem`.
- Permission to create a schema and Metric Views in the configured catalog.
- Materialization in Lab 3 additionally requires the workspace and runtime capabilities listed in the [Metric View feature matrix](https://docs.databricks.com/aws/en/uc-semantics/metric-views/feature-availability).

## Deploy and run

The checked-in `dev` target defaults to the writable `workspace.metric_views_lab` schema. `warehouse_id` intentionally has no default: supply your own CLI profile and SQL Warehouse ID at deploy time.

```bash
databricks bundle validate --target dev --profile DATABRICKS_PROFILE --var="warehouse_id=YOUR_SQL_WAREHOUSE_ID"
databricks bundle deploy --target dev --profile DATABRICKS_PROFILE --var="warehouse_id=YOUR_SQL_WAREHOUSE_ID"
databricks bundle run metric_views_labs --target dev --profile DATABRICKS_PROFILE --var="warehouse_id=YOUR_SQL_WAREHOUSE_ID"
```

The notebooks are synced under the bundle workspace path. The job runs them in order so that each lab leaves its objects available for inspection. The run should finish with three successful tasks. Lab 1 should show matching order and distinct-order counts, a zero-row duplicate-key check, and AOV controls that agree with the Metric View. Lab 2 should show a larger naive joined header total than the direct header control, while the one-to-many Metric View keeps header and line-item measures at their own grains. Lab 3 should return a rolling customer metric and leave the materialization task queryable even if its managed refresh is still pending.

To use another catalog, schema, or warehouse, override the variables and update the notebook identifiers before running:

```bash
databricks bundle deploy --target dev --profile DATABRICKS_PROFILE \
  --var="catalog=your_catalog" \
  --var="schema=your_schema" \
  --var="warehouse_id=YOUR_SQL_WAREHOUSE_ID"
```

The notebooks currently use the checked-in default `workspace.metric_views_lab` in their SQL identifiers. If you override the catalog or schema variables, replace those identifiers in the notebook copies as part of the same change; bundle variables do not rewrite SQL identifiers inside an `.ipynb` file.

## What the labs demonstrate

- A Metric View is a governed definition, not automatically a precomputed aggregate table.
- Measures are evaluated at query time with `MEASURE(...)`; dimensions/fields choose the requested grain.
- Averages must carry their numerator and denominator instead of averaging precomputed averages.
- Join cardinality is a correctness contract. `RELY` is an optimizer promise, not a runtime uniqueness check.
- Windows and semiadditive rules have their own runtime and YAML-version gates.
- Materialization changes freshness, cost, and refresh behavior; verify the plan rather than guessing from elapsed time.

Further reading: [YAML reference](https://docs.databricks.com/aws/en/uc-semantics/metric-views/yaml-reference), [querying Metric Views](https://docs.databricks.com/aws/en/uc-semantics/metric-views/query), [joins](https://docs.databricks.com/aws/en/uc-semantics/metric-views/joins), and [materialization](https://docs.databricks.com/aws/en/uc-semantics/metric-views/materialization).
