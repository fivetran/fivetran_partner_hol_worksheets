# Fivetran Partner Hands on Lab Worksheets

Public repository of hands-on lab worksheets for the technical trainings Fivetran offers to
partners — Systems Integrators (SIs) and Global Systems Integrators (GSIs). Each worksheet
walks a partner through a guided, step-by-step lab for one training.

## Contents

One folder per training, each containing that training's `<training>_hands_on_lab.md`
worksheet. Some trainings also have variants:

- `bigquery_` — BigQuery destination variants, following the existing BigQuery Technical
  Foundations worksheet. Combined variants retain their source/workstation suffixes.
- `ws_` — for labs run on a provided browser-accessible Linux workstation instead of the
  attendee's own laptop.
- `mysql_`, `sql_server_`, `oracle_` — for labs where the source database is MySQL, SQL
  Server, or Oracle instead of the default PostgreSQL. The setup is the same apart from the
  connector chosen and the update method (Fivetran Teleport Sync).

- [`activations/`](activations/activations_hands_on_lab.md) — Activations
- [`connector_sdk/`](connector_sdk/connector_sdk_hands_on_lab.md) — Connector SDK
  - [Workstation variant](connector_sdk/ws_connector_sdk_hands_on_lab.md)
  - [AI Plugin variant](connector_sdk/connector_sdk_ai_hands_on_lab.md) — build the connector with the Claude AI Plugin (workstation)
- [`hybrid_deployment/`](hybrid_deployment/hybrid_deployment_hands_on_lab.md) — Hybrid Deployment
  - [Workstation variant](hybrid_deployment/ws_hybrid_deployment_hands_on_lab.md)
  - Database variants: [MySQL](hybrid_deployment/mysql_hybrid_deployment_hands_on_lab.md), [SQL Server](hybrid_deployment/sql_server_hybrid_deployment_hands_on_lab.md), [Oracle](hybrid_deployment/oracle_hybrid_deployment_hands_on_lab.md)
  - Workstation + database variants: [MySQL](hybrid_deployment/mysql_ws_hybrid_deployment_hands_on_lab.md), [SQL Server](hybrid_deployment/sql_server_ws_hybrid_deployment_hands_on_lab.md), [Oracle](hybrid_deployment/oracle_ws_hybrid_deployment_hands_on_lab.md)
- [`managed_data_lake_service/`](managed_data_lake_service/managed_data_lake_service_hands_on_lab.md) — Managed Data Lake Service
  - Database variants: [MySQL](managed_data_lake_service/mysql_managed_data_lake_service_hands_on_lab.md), [SQL Server](managed_data_lake_service/sql_server_managed_data_lake_service_hands_on_lab.md), [Oracle](managed_data_lake_service/oracle_managed_data_lake_service_hands_on_lab.md)
- [`operations_management/`](operations_management/operations_management_hands_on_lab.md) — Operations Management
  - Database variants: [MySQL](operations_management/mysql_operations_management_hands_on_lab.md), [SQL Server](operations_management/sql_server_operations_management_hands_on_lab.md), [Oracle](operations_management/oracle_operations_management_hands_on_lab.md)
- [`programmatic_management/`](programmatic_management/programmatic_management_hands_on_lab.md) — Programmatic Management
  - Database variants: [MySQL](programmatic_management/mysql_programmatic_management_hands_on_lab.md), [SQL Server](programmatic_management/sql_server_programmatic_management_hands_on_lab.md), [Oracle](programmatic_management/oracle_programmatic_management_hands_on_lab.md)
- [`technical_foundations/`](technical_foundations/technical_foundations_hands_on_lab.md) — Technical Foundations
  - Database variants: [MySQL](technical_foundations/mysql_technical_foundations_hands_on_lab.md), [SQL Server](technical_foundations/sql_server_technical_foundations_hands_on_lab.md), [Oracle](technical_foundations/oracle_technical_foundations_hands_on_lab.md)
  - [AI variant](technical_foundations/technical_foundations_ai_hands_on_lab.md) — Parts 1–3 as above, then build a RAG chatbot with Snowflake Cortex and Streamlit
- [`transformations/`](transformations/transformations_hands_on_lab.md) — Transformations
  - [Workstation variant](transformations/ws_transformations_hands_on_lab.md)

## BigQuery destination worksheets

These editions use instructor-provided BigQuery project, service-account JSON, dataset
prefix and location credentials. Hybrid Deployment also requires a GCS staging bucket.
Transformation labs require dbt access to the landing and output datasets. See each
worksheet's credentials and instructor prerequisites before running a session.
Attendees do not receive BigQuery Studio access. All BigQuery destination data previews are instructor-led;
attendees verify destination setup, syncs and transformation runs through Fivetran and
the lab tools. Local DuckDB previews remain hands-on.

The new editions have been adapted for BigQuery and checked statically; end-to-end
validation with provisioned lab resources is still required before delivery.

| Training | BigQuery worksheets |
|---|---|
| Connector SDK | [AI Plugin](connector_sdk/bigquery_connector_sdk_ai_hands_on_lab.md), [Standard](connector_sdk/bigquery_connector_sdk_hands_on_lab.md), [Workstation](connector_sdk/bigquery_ws_connector_sdk_hands_on_lab.md) |
| Hybrid Deployment | [Standard](hybrid_deployment/bigquery_hybrid_deployment_hands_on_lab.md), [MySQL](hybrid_deployment/bigquery_mysql_hybrid_deployment_hands_on_lab.md), [MySQL + Workstation](hybrid_deployment/bigquery_mysql_ws_hybrid_deployment_hands_on_lab.md), [Oracle](hybrid_deployment/bigquery_oracle_hybrid_deployment_hands_on_lab.md), [Oracle + Workstation](hybrid_deployment/bigquery_oracle_ws_hybrid_deployment_hands_on_lab.md), [SQL Server](hybrid_deployment/bigquery_sql_server_hybrid_deployment_hands_on_lab.md), [SQL Server + Workstation](hybrid_deployment/bigquery_sql_server_ws_hybrid_deployment_hands_on_lab.md), [Workstation](hybrid_deployment/bigquery_ws_hybrid_deployment_hands_on_lab.md) |
| Operations Management | [MySQL](operations_management/bigquery_mysql_operations_management_hands_on_lab.md), [Standard](operations_management/bigquery_operations_management_hands_on_lab.md), [Oracle](operations_management/bigquery_oracle_operations_management_hands_on_lab.md), [SQL Server](operations_management/bigquery_sql_server_operations_management_hands_on_lab.md) |
| Programmatic Management | [MySQL](programmatic_management/bigquery_mysql_programmatic_management_hands_on_lab.md), [Oracle](programmatic_management/bigquery_oracle_programmatic_management_hands_on_lab.md), [Standard](programmatic_management/bigquery_programmatic_management_hands_on_lab.md), [SQL Server](programmatic_management/bigquery_sql_server_programmatic_management_hands_on_lab.md) |
| Technical Foundations | [MySQL](technical_foundations/bigquery_mysql_technical_foundations_hands_on_lab.md), [Oracle](technical_foundations/bigquery_oracle_technical_foundations_hands_on_lab.md), [SQL Server](technical_foundations/bigquery_sql_server_technical_foundations_hands_on_lab.md), [Standard](technical_foundations/bigquery_technical_foundations_hands_on_lab.md) |
| Transformations | [Standard](transformations/bigquery_transformations_hands_on_lab.md), [Workstation](transformations/bigquery_ws_transformations_hands_on_lab.md) |

Activations, Managed Data Lake Service and the Technical Foundations AI worksheet are
not included. The existing standard BigQuery Technical Foundations worksheet is reused.
The retail dbt Platform lab is also excluded because attendees do not have access to
BigQuery Studio.

## dbt Platform add-ons

The [`dbt/`](dbt/README.md) folder contains 88 add-on worksheets: Snowflake and BigQuery,
with and without dbt Wizard, for each of 21 industries plus NewsAPI. Each covers dbt
Platform setup, tested marts, State comparisons and a manually triggered deployment job.
Manual editions include complete answers at the end. Destination previews are instructor-led.

## Snowflake Cortex add-ons

The [`snowflake_cortex/`](snowflake_cortex/README.md) folder contains 22 optional AI
worksheets, one per industry/NewsAPI dbt track. Each works after either Snowflake dbt
edition and uses its transformed mart to build a semantic view and Cortex Agent.
The sequence is **basic lab → dbt Platform → Snowflake Cortex**; no BigQuery AI
counterpart is included. These add-ons require provisioned Snowflake access.

## Source of Truth

These worksheets are copies published from Fivetran's internal training content repository,
which also builds the accompanying slide decks and exams. This repo exists to give partners a
lightweight, versioned place to access just the lab worksheets.
