## Data model overview

This model separates reporting KPIs, raw energy activity, data quality controls, and governance metadata. Dimensions filter facts through one-to-many relationships; most filters should flow from dimensions to facts.

## Fact tables

- fact_kpi_value
  - What it is: the central reporting fact, one row per site, period, KPI (and publish state), with the calculated KPI value.
  - Typical questions it answers: KPI totals, latest values, period change, certified vs draft comparisons, KPI-level slicing.
  - Key joins (typical): dim_time (time_id), dim_site (site_id), dim_kpi (kpi_id), dim_certified_snapshot (certified_snapshot_id when present), dim_factor (factor_id when present), dim_source (source_id if present in your model).

- fact_electricity
  - What it is: electricity activity data (meter-level) used to support energy reporting and explain drivers.
  - Typical questions it answers: electricity usage by site and month, trends, contributions by site, source/system coverage.
  - Key joins: dim_time (time_id), dim_site (site_id), dim_source (source_id).

- fact_fuel
  - What it is: fuel activity data (fuel_type level) used to support energy reporting.
  - Typical questions it answers: fuel usage by site and month, split by fuel type, trends, drivers.
  - Key joins: dim_time (time_id), dim_site (site_id), dim_source (source_id).

- fact_exception
  - What it is: record-level validation and data quality exceptions tied to a KPI, site, and period.
  - Typical questions it answers: open issues, blocked issues, exceptions by gate and severity, ageing, operational workload.
  - Key joins: dim_time (time_id), dim_site (site_id), dim_kpi (kpi_id), dim_owner (owner_id).

- fact_submission_expectation
  - What it is: expected vs received submissions (completeness controls) by site and period.
  - Typical questions it answers: validated coverage %, missing submissions, timeliness controls, coverage trends.
  - Key joins: dim_time (time_id), dim_site (site_id), dim_source (source/system when used).

## Dimension tables

- dim_time
  - What it is: reporting calendar, one row per reporting month.
  - Used for: period slicers, month labels/sort, consistent time grain across all facts.
  - Joins to: fact_kpi_value, fact_electricity, fact_fuel, fact_exception, fact_submission_expectation (time_id).

- dim_site
  - What it is: site master data (name, region, business unit, active dates).
  - Used for: site slicers, site groupings, rollups.
  - Joins to: fact_kpi_value, fact_electricity, fact_fuel, fact_exception, fact_submission_expectation (site_id).

- dim_kpi
  - What it is: KPI dictionary and methodology metadata (unit, grain, scope, method_flag, definition_version, factor_set_id mapping if present).
  - Used for: KPI slicers, KPI definition cards, method context, emissions-only gating logic.
  - Joins to: fact_kpi_value, fact_exception (kpi_id), and to dim_factor_set (factor_set_id) if configured.

- dim_factor_set
  - What it is: grouping layer for factors (a named set used by a KPI or calculation method).
  - Used for: linking KPI selection to the right set of emission factors.
  - Joins to: dim_factor (factor_set_id), and optionally dim_kpi (factor_set_id).

- dim_factor
  - What it is: emission factor register (factor values, geography, units in/out, effective dates, version, change reason).
  - Used for: factor transparency tables, factor-change indicators, audit traceability.
  - Joins to: dim_factor_set (factor_set_id), and to fact_kpi_value (factor_id) where factor lineage is captured.

- dim_source
  - What it is: data source metadata (system, source type, ingestion method).
  - Used for: lineage, source-level filtering, explaining coverage gaps.
  - Joins to: fact_electricity, fact_fuel, fact_submission_expectation, and any other fact that includes source_id.

- dim_owner
  - What it is: ownership/queue metadata for operational follow-up (roles, queues).
  - Used for: exception ownership, queue breakdowns, responsibility views.
  - Joins to: fact_exception (owner_id).

- dim_certified_snapshot
  - What it is: certification and locking metadata for external reporting snapshots (period_label, locked_at, notes).
  - Used for: certified snapshot card, audit readiness context.
  - Joins to: fact_kpi_value (certified_snapshot_id) when the KPI record is tied to a snapshot.

- dim_lens
  - What it is: reporting lens selector (e.g., GRI, CDP).
  - Used for: changing the framing of questions or KPI groupings when you build lens logic.
  - Joins: intentionally standalone unless you create a mapping table.

- dim_data_responsibility
  - What it is: data responsibility mapping (who owns what dataset/KPI/site/process).
  - Used for: methods page responsibility table and governance views.
  - Joins: depends on your design (often to dim_owner and/or dim_source, sometimes to dim_kpi).

- dim_energy_source
  - What it is: energy category reference (electricity, natural gas, fleet fuel) for consistent labelling.
  - Used for: energy source breakdown visuals on the Topic View page.
  - Joins: depends on your design; often disconnected unless you add a bridge/mapping.

- dim_site_all
  - What it is: helper table to support an “All Sites” option.
  - Used for: slicer UX and selection logic.
  - Joins: should stay disconnected or be used only in controlled patterns to avoid ambiguous paths.

## Modelling note to keep it stable

If you see ambiguous path errors, you likely have two valid routes between the same tables (common around factor_set and factor). Keep a single active filter path; mark the alternate relationship inactive and activate it only inside measures when needed.





