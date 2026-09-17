# Fivetran Partner Hands on Lab Worksheets

Public repository of hands-on lab worksheets for the technical trainings Fivetran offers to
partners — Systems Integrators (SIs) and Global Systems Integrators (GSIs). Each worksheet
walks a partner through a guided, step-by-step lab for one training.

## Contents

One folder per training, each containing that training's `<training>_hands_on_lab.md`
worksheet. Some trainings also have variants:

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

## Source of Truth

These worksheets are copies published from Fivetran's internal training content repository,
which also builds the accompanying slide decks and exams. This repo exists to give partners a
lightweight, versioned place to access just the lab worksheets.
