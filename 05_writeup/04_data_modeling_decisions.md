The data model follows a fact-dimension design, optimised for reporting and auditability rather than raw ingestion.

Fact tables

fact_electricity
Raw electricity consumption at meter and site level.

fact_fuel
Natural gas and fleet fuel consumption.

fact_kpi_value
Calculated KPI outputs, including factor lineage and method flags.

fact_exception
Data quality and validation issues, tracked independently of KPI values.

fact_submission_expectation
Expected vs received data submissions for coverage analysis.

Dimension tables

Time, site, KPI, source, owner, energy source

Emission factors and factor sets with versions and effective dates

Certified snapshots for locked reporting periods

This separation allows:

Stable reporting even when raw data changes

KPI-specific filtering without duplicating logic

Transparent linkage between activity data, factors, and results

The model intentionally avoids snowflaking beyond what is needed for clarity and governance.