# dbt Platform add-on worksheets

Four editions for each of 21 industry templates, plus four for NewsAPI: **88 worksheets**.
Each continues from a completed Fivetran source lab and covers the scope of the reference
worksheet's Parts 5–7: dbt Platform access, tested marts, State comparisons and a manual
deployment job. No Cortex exercise or attendee destination-console access is required.

The manual editions teach the implementation step by step and finish with complete SQL,
YAML, project configuration and reconciliation-test answers. Wizard editions provide the
same contracts as reviewed prompts. BigQuery uses the provisioned dbt runner connection;
attendees do not need BigQuery Studio or IAM access. Destination data previews are
instructor-led; local DuckDB previews remain hands-on.

Manual editions are in [`manual/`](manual/) and Wizard editions in [`wizard/dbt_platform/`](wizard/dbt_platform/),
one folder per industry in each (for example `manual/agriculture/` and `wizard/dbt_platform/agriculture/`).
Wizard CLI editions, run on a provided workstation, are in [`wizard/cli/`](wizard/cli/).

For BigQuery, each participant configures **Your profile → Credentials → their assigned
project → Edit** using their own dbt login. Enter the **BigQuery dbt dataset** from the
assigned 1Password lab item, keep the provisioned Service Account JSON connection and
default target name, then **Test connection** until **Complete** is green. Click **Save**,
confirm **Saved**, and return to Studio to wait for **Connected**. These are personal
credentials; participants must not edit the shared development environment or connection.

| Industry / dataset | Snowflake + Wizard | BigQuery + Wizard | Snowflake manual | BigQuery manual |
|---|---|---|---|---|
| Agriculture | [Worksheet](wizard/dbt_platform/agriculture/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/agriculture/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/agriculture/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/agriculture/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Attractions | [Worksheet](wizard/dbt_platform/attractions/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/attractions/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/attractions/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/attractions/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Construction | [Worksheet](wizard/dbt_platform/construction/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/construction/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/construction/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/construction/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Consumer Packaged Goods | [Worksheet](wizard/dbt_platform/consumer_packaged_goods/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/consumer_packaged_goods/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/consumer_packaged_goods/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/consumer_packaged_goods/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Customer Service | [Worksheet](wizard/dbt_platform/customer_service/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/customer_service/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/customer_service/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/customer_service/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Energy | [Worksheet](wizard/dbt_platform/energy/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/energy/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/energy/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/energy/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Financial Services | [Worksheet](wizard/dbt_platform/financial_services/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/financial_services/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/financial_services/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/financial_services/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Healthcare | [Worksheet](wizard/dbt_platform/healthcare/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/healthcare/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/healthcare/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/healthcare/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Higher Education | [Worksheet](wizard/dbt_platform/higher_education/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/higher_education/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/higher_education/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/higher_education/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Hospitality | [Worksheet](wizard/dbt_platform/hospitality/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/hospitality/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/hospitality/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/hospitality/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Insurance | [Worksheet](wizard/dbt_platform/insurance/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/insurance/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/insurance/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/insurance/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Manufacturing | [Worksheet](wizard/dbt_platform/manufacturing/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/manufacturing/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/manufacturing/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/manufacturing/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Media Entertainment | [Worksheet](wizard/dbt_platform/media_entertainment/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/media_entertainment/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/media_entertainment/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/media_entertainment/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Pharma | [Worksheet](wizard/dbt_platform/pharma/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/pharma/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/pharma/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/pharma/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Retail | [Worksheet](wizard/dbt_platform/retail/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/retail/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/retail/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/retail/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Sales | [Worksheet](wizard/dbt_platform/sales/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/sales/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/sales/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/sales/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Supply Chain | [Worksheet](wizard/dbt_platform/supply_chain/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/supply_chain/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/supply_chain/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/supply_chain/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Telco | [Worksheet](wizard/dbt_platform/telco/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/telco/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/telco/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/telco/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Transportation | [Worksheet](wizard/dbt_platform/transportation/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/transportation/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/transportation/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/transportation/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Utilities | [Worksheet](wizard/dbt_platform/utilities/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/utilities/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/utilities/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/utilities/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Weather | [Worksheet](wizard/dbt_platform/weather/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/weather/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/weather/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/weather/bigquery_manual_dbt_platform_hands_on_lab.md) |
| NewsAPI | [Worksheet](wizard/dbt_platform/newsapi/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/dbt_platform/newsapi/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/newsapi/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/newsapi/bigquery_manual_dbt_platform_hands_on_lab.md) |

## dbt Wizard CLI on a workstation

[`wizard/cli/`](wizard/cli/) has a Snowflake and a BigQuery edition for each industry and
NewsAPI: **44 worksheets**. They use the same business questions, model requirements and
Wizard prompts as the dbt Platform Wizard editions, but run on a provided Linux workstation:
the attendee clones their own `lab-dbt-` repository, creates the key file their pre-written
`~/.dbt/profiles.yml` points at, builds with dbt Fusion (`dbtf`), generates the marts with
the dbt Wizard CLI (`wizard`), tests a controlled threshold change and pushes a branch.
Snowflake editions then view the marts in the Snowflake web UI; BigQuery editions use
DBeaver on the workstation with the same service-account key, since attendees have no
BigQuery Studio access. dbt State and deployment jobs are dbt Platform features and are
not part of the CLI editions.

| Industry / dataset | Snowflake + Wizard CLI | BigQuery + Wizard CLI |
|---|---|---|
| Agriculture | [Worksheet](wizard/cli/agriculture/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/agriculture/bigquery_wizard_cli_hands_on_lab.md) |
| Attractions | [Worksheet](wizard/cli/attractions/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/attractions/bigquery_wizard_cli_hands_on_lab.md) |
| Construction | [Worksheet](wizard/cli/construction/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/construction/bigquery_wizard_cli_hands_on_lab.md) |
| Consumer Packaged Goods | [Worksheet](wizard/cli/consumer_packaged_goods/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/consumer_packaged_goods/bigquery_wizard_cli_hands_on_lab.md) |
| Customer Service | [Worksheet](wizard/cli/customer_service/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/customer_service/bigquery_wizard_cli_hands_on_lab.md) |
| Energy | [Worksheet](wizard/cli/energy/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/energy/bigquery_wizard_cli_hands_on_lab.md) |
| Financial Services | [Worksheet](wizard/cli/financial_services/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/financial_services/bigquery_wizard_cli_hands_on_lab.md) |
| Healthcare | [Worksheet](wizard/cli/healthcare/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/healthcare/bigquery_wizard_cli_hands_on_lab.md) |
| Higher Education | [Worksheet](wizard/cli/higher_education/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/higher_education/bigquery_wizard_cli_hands_on_lab.md) |
| Hospitality | [Worksheet](wizard/cli/hospitality/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/hospitality/bigquery_wizard_cli_hands_on_lab.md) |
| Insurance | [Worksheet](wizard/cli/insurance/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/insurance/bigquery_wizard_cli_hands_on_lab.md) |
| Manufacturing | [Worksheet](wizard/cli/manufacturing/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/manufacturing/bigquery_wizard_cli_hands_on_lab.md) |
| Media Entertainment | [Worksheet](wizard/cli/media_entertainment/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/media_entertainment/bigquery_wizard_cli_hands_on_lab.md) |
| Pharma | [Worksheet](wizard/cli/pharma/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/pharma/bigquery_wizard_cli_hands_on_lab.md) |
| Retail | [Worksheet](wizard/cli/retail/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/retail/bigquery_wizard_cli_hands_on_lab.md) |
| Sales | [Worksheet](wizard/cli/sales/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/sales/bigquery_wizard_cli_hands_on_lab.md) |
| Supply Chain | [Worksheet](wizard/cli/supply_chain/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/supply_chain/bigquery_wizard_cli_hands_on_lab.md) |
| Telco | [Worksheet](wizard/cli/telco/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/telco/bigquery_wizard_cli_hands_on_lab.md) |
| Transportation | [Worksheet](wizard/cli/transportation/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/transportation/bigquery_wizard_cli_hands_on_lab.md) |
| Utilities | [Worksheet](wizard/cli/utilities/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/utilities/bigquery_wizard_cli_hands_on_lab.md) |
| Weather | [Worksheet](wizard/cli/weather/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/weather/bigquery_wizard_cli_hands_on_lab.md) |
| NewsAPI / Connector SDK | [Worksheet](wizard/cli/newsapi/snowflake_wizard_cli_hands_on_lab.md) | [Worksheet](wizard/cli/newsapi/bigquery_wizard_cli_hands_on_lab.md) |

## Lake Compute on a workstation

[`lake_compute/workstation/`](lake_compute/workstation/) has two standalone worksheets per
industry for Managed Data Lake Service labs on Amazon S3 with Lake Compute as the
destination: no warehouse and no dbt Platform. Both cover the S3 destination and
PostgreSQL connector setup, the Lake Compute token, the workstation login, environment
variables, `profiles.yml` and `dbt_project.yml`, cloning the person's generated `lab-dbt-`
repository, building the industry's two tested marts with dbt Fusion (`dbtf`) on Lake
Compute, and querying the results with the DuckDB CLI. The marts are the same contracts as
the manual dbt Platform editions, materialized as tables and written in DuckDB SQL.

- **Manual** editions ([`manual/`](lake_compute/workstation/manual/)) have attendees write
  the marts themselves and end with complete answers, validated locally in DuckDB.
- **Wizard** editions ([`wizard/`](lake_compute/workstation/wizard/)) have attendees
  generate the marts with the dbt Wizard CLI (`wizard`), using a prompt that states Lake
  Compute's limits (DuckDB SQL, tables only, no hooks, grants or unit tests). They need a
  lab with Lake Compute and the Wizard CLI enabled together.

| Industry | Manual | dbt Wizard CLI |
|---|---|---|
| Agriculture | [Worksheet](lake_compute/workstation/manual/agriculture/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/agriculture/lake_compute_workstation_hands_on_lab.md) |
| Attractions | [Worksheet](lake_compute/workstation/manual/attractions/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/attractions/lake_compute_workstation_hands_on_lab.md) |
| Construction | [Worksheet](lake_compute/workstation/manual/construction/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/construction/lake_compute_workstation_hands_on_lab.md) |
| Consumer Packaged Goods | [Worksheet](lake_compute/workstation/manual/consumer_packaged_goods/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/consumer_packaged_goods/lake_compute_workstation_hands_on_lab.md) |
| Customer Service | [Worksheet](lake_compute/workstation/manual/customer_service/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/customer_service/lake_compute_workstation_hands_on_lab.md) |
| Energy | [Worksheet](lake_compute/workstation/manual/energy/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/energy/lake_compute_workstation_hands_on_lab.md) |
| Financial Services | [Worksheet](lake_compute/workstation/manual/financial_services/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/financial_services/lake_compute_workstation_hands_on_lab.md) |
| Healthcare | [Worksheet](lake_compute/workstation/manual/healthcare/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/healthcare/lake_compute_workstation_hands_on_lab.md) |
| Higher Education | [Worksheet](lake_compute/workstation/manual/higher_education/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/higher_education/lake_compute_workstation_hands_on_lab.md) |
| Hospitality | [Worksheet](lake_compute/workstation/manual/hospitality/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/hospitality/lake_compute_workstation_hands_on_lab.md) |
| Insurance | [Worksheet](lake_compute/workstation/manual/insurance/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/insurance/lake_compute_workstation_hands_on_lab.md) |
| Manufacturing | [Worksheet](lake_compute/workstation/manual/manufacturing/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/manufacturing/lake_compute_workstation_hands_on_lab.md) |
| Media Entertainment | [Worksheet](lake_compute/workstation/manual/media_entertainment/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/media_entertainment/lake_compute_workstation_hands_on_lab.md) |
| Pharma | [Worksheet](lake_compute/workstation/manual/pharma/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/pharma/lake_compute_workstation_hands_on_lab.md) |
| Retail | [Worksheet](lake_compute/workstation/manual/retail/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/retail/lake_compute_workstation_hands_on_lab.md) |
| Sales | [Worksheet](lake_compute/workstation/manual/sales/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/sales/lake_compute_workstation_hands_on_lab.md) |
| Supply Chain | [Worksheet](lake_compute/workstation/manual/supply_chain/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/supply_chain/lake_compute_workstation_hands_on_lab.md) |
| Telco | [Worksheet](lake_compute/workstation/manual/telco/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/telco/lake_compute_workstation_hands_on_lab.md) |
| Transportation | [Worksheet](lake_compute/workstation/manual/transportation/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/transportation/lake_compute_workstation_hands_on_lab.md) |
| Utilities | [Worksheet](lake_compute/workstation/manual/utilities/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/utilities/lake_compute_workstation_hands_on_lab.md) |
| Weather | [Worksheet](lake_compute/workstation/manual/weather/lake_compute_workstation_hands_on_lab.md) | [Worksheet](lake_compute/workstation/wizard/weather/lake_compute_workstation_hands_on_lab.md) |

## Choosing and provisioning a template

Use the corresponding standard `dbt-<industry>-lab` template in
[dbt-Hands-on-Labs](https://github.com/orgs/dbt-Hands-on-Labs/repositories). Insurance uses
`dbt-insurance-lab`, whose staging contract matches these exercises; the separate
`dbt-insurance-lab-hol` template has prebuilt marts and is not a second industry.

For Connector SDK, use [dbt-newsapi-lab](https://github.com/dbt-Hands-on-Labs/dbt-newsapi-lab).
It supports the standard two-table lab and optional Everything data from the AI Plugin
lab. Verify the generated connector's column names before enabling the optional table.
An administrator must make this template selectable for the Connector SDK add-on in
LabStudios; publishing it does not change application configuration or existing labs.

Before delivery, the instructor provisions source data, dbt project/connection, developer
and job/environment permissions, Wizard access where applicable, State and dedicated
deployment profiles. Keep each learner's development and deployment targets separate.
The worksheet identifiers come from the assigned lab resources, not from another session.

BigQuery editions use `dbt show` for the Studio aggregate check and a supplied
`dbt run-operation` macro for the deployment job. Create and validate the macro
before committing the exercise files. The helper preserves each industry's own
aggregate columns and logs the effective target. BigQuery scalar reconciliation
checks include a single-row `FROM` clause before `WHERE`.

## Validation scope

See [validation evidence](validation.md) for template revisions, local SQL/seed checks,
project parsing and the boundary between local verification and a live lab run.

## Continue with Snowflake Cortex

After completing either Snowflake edition, continue with the matching
[Snowflake Cortex add-on](../snowflake_cortex/README.md). It uses the restored development
mart to create a semantic view and draft Cortex Agent. This optional step requires
provisioned Snowflake access; there is no BigQuery AI counterpart in this series.

## Continue with the dbt Semantic Layer and MCP

On labs launched with **dbt Semantic Layer + remote MCP**, attendees stop after Part 3 of
either edition and continue with the matching
[Semantic Layer and MCP add-on](semantic_layer_and_mcp/README.md) instead of Parts 4–6. It
adds governed metrics, publishes them through the attendee's Production environment and
connects Claude Code on the lab workstation to the dbt remote MCP server.

## Migrate from dbt v1 to v2

The [v2 migration worksheets](v2_migration/README.md) start with deliberately legacy
industry or NewsAPI code on v1, use Wizard to repair compatibility issues on v2, compare State
decisions, and finish in a dedicated Production environment. Snowflake and BigQuery
editions cover all 21 industries and NewsAPI (44 migration worksheets), separate from
the 88 add-on worksheets above.

## Lake Compute sidecar on dbt Platform

[`lake_compute/dbt_platform/`](lake_compute/dbt_platform/) has two standalone worksheets per
industry for Managed Data Lake Service labs on Amazon S3 with Snowflake as the destination and
Lake Compute as a second dbt adapter (the Lake Compute sidecar), run in dbt Platform. Both
cover the S3 destination with Lake Compute turned on, the PostgreSQL connector, the Snowflake
catalog integration and the catalog-linked landing database, the Lake Compute token, the
person's key-pair development credentials and the `DBT_ENV_SECRET_LAKE_COMPUTE_TOKEN`
variable, staging on Snowflake as Iceberg tables, the industry's two tested marts on Lake
Compute (`adapter='lakecompute'`, DuckDB SQL), a manually run Production job and querying
every layer from Snowflake. The marts are the same contracts as the other editions. They need
a lab with the Lake Compute sidecar and the dbt Platform add-on in a per-person project mode;
LabStudios sets the environments to dbt v2 and adds the extended attributes and the two flag
variables at launch.

- **Manual** editions ([`manual/`](lake_compute/dbt_platform/manual/)) have attendees write
  the marts themselves and end with complete answers.
- **Wizard** editions ([`wizard/`](lake_compute/dbt_platform/wizard/)) have attendees
  generate the marts with dbt Wizard in Studio, using a prompt that puts the marts on Lake
  Compute and states its limits.

| Industry | Manual | dbt Wizard |
|---|---|---|
| Agriculture | [Worksheet](lake_compute/dbt_platform/manual/agriculture/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/agriculture/lake_compute_dbt_platform_hands_on_lab.md) |
| Attractions | [Worksheet](lake_compute/dbt_platform/manual/attractions/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/attractions/lake_compute_dbt_platform_hands_on_lab.md) |
| Construction | [Worksheet](lake_compute/dbt_platform/manual/construction/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/construction/lake_compute_dbt_platform_hands_on_lab.md) |
| Consumer Packaged Goods | [Worksheet](lake_compute/dbt_platform/manual/consumer_packaged_goods/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/consumer_packaged_goods/lake_compute_dbt_platform_hands_on_lab.md) |
| Customer Service | [Worksheet](lake_compute/dbt_platform/manual/customer_service/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/customer_service/lake_compute_dbt_platform_hands_on_lab.md) |
| Energy | [Worksheet](lake_compute/dbt_platform/manual/energy/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/energy/lake_compute_dbt_platform_hands_on_lab.md) |
| Financial Services | [Worksheet](lake_compute/dbt_platform/manual/financial_services/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/financial_services/lake_compute_dbt_platform_hands_on_lab.md) |
| Healthcare | [Worksheet](lake_compute/dbt_platform/manual/healthcare/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/healthcare/lake_compute_dbt_platform_hands_on_lab.md) |
| Higher Education | [Worksheet](lake_compute/dbt_platform/manual/higher_education/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/higher_education/lake_compute_dbt_platform_hands_on_lab.md) |
| Hospitality | [Worksheet](lake_compute/dbt_platform/manual/hospitality/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/hospitality/lake_compute_dbt_platform_hands_on_lab.md) |
| Insurance | [Worksheet](lake_compute/dbt_platform/manual/insurance/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/insurance/lake_compute_dbt_platform_hands_on_lab.md) |
| Manufacturing | [Worksheet](lake_compute/dbt_platform/manual/manufacturing/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/manufacturing/lake_compute_dbt_platform_hands_on_lab.md) |
| Media Entertainment | [Worksheet](lake_compute/dbt_platform/manual/media_entertainment/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/media_entertainment/lake_compute_dbt_platform_hands_on_lab.md) |
| Pharma | [Worksheet](lake_compute/dbt_platform/manual/pharma/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/pharma/lake_compute_dbt_platform_hands_on_lab.md) |
| Retail | [Worksheet](lake_compute/dbt_platform/manual/retail/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/retail/lake_compute_dbt_platform_hands_on_lab.md) |
| Sales | [Worksheet](lake_compute/dbt_platform/manual/sales/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/sales/lake_compute_dbt_platform_hands_on_lab.md) |
| Supply Chain | [Worksheet](lake_compute/dbt_platform/manual/supply_chain/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/supply_chain/lake_compute_dbt_platform_hands_on_lab.md) |
| Telco | [Worksheet](lake_compute/dbt_platform/manual/telco/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/telco/lake_compute_dbt_platform_hands_on_lab.md) |
| Transportation | [Worksheet](lake_compute/dbt_platform/manual/transportation/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/transportation/lake_compute_dbt_platform_hands_on_lab.md) |
| Utilities | [Worksheet](lake_compute/dbt_platform/manual/utilities/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/utilities/lake_compute_dbt_platform_hands_on_lab.md) |
| Weather | [Worksheet](lake_compute/dbt_platform/manual/weather/lake_compute_dbt_platform_hands_on_lab.md) | [Worksheet](lake_compute/dbt_platform/wizard/weather/lake_compute_dbt_platform_hands_on_lab.md) |

## Lake Compute sidecar on a workstation

[`lake_compute/sidecar_workstation/`](lake_compute/sidecar_workstation/) has two standalone
worksheets per industry for the Lake Compute sidecar (Managed Data Lake Service on Amazon S3
with Snowflake as the destination and Lake Compute as a second dbt adapter), run with dbt
Fusion (`dbtf`) on the provided workstation. Both cover the S3 destination with Lake Compute
turned on, the PostgreSQL connector, the Snowflake catalog integration and the catalog-linked
landing database, the Lake Compute token, the workstation login, the person's Snowflake key
file (`~/.dbt/snowflake_key.p8`, created from their 1Password item) and the
`LAKE_COMPUTE_TOKEN` variable, the pre-written two-output `profiles.yml`, cloning the person's
generated `lab-dbt-` repository, staging on Snowflake as Iceberg tables, the industry's two
tested marts on Lake Compute (`adapter='lakecompute'`, DuckDB SQL) and querying every layer
from Snowflake. The marts are the same contracts as the other editions. They need a lab with
the Lake Compute sidecar and the workstation; LabStudios writes the profile and the
`~/.bashrc` block with the two experimental-adapter flags at launch. Use these, not the
[Lake Compute on a workstation](#lake-compute-on-a-workstation) editions, on a sidecar lab.

- **Manual** editions ([`manual/`](lake_compute/sidecar_workstation/manual/)) have attendees
  write the marts themselves and end with complete answers.
- **Wizard** editions ([`wizard/`](lake_compute/sidecar_workstation/wizard/)) have attendees
  generate the marts with the dbt Wizard CLI (`wizard`), using a prompt that puts the marts
  on Lake Compute and states its limits. They need a sidecar lab with the dbt Wizard CLI
  add-on.

| Industry | Manual | dbt Wizard CLI |
|---|---|---|
| Agriculture | [Worksheet](lake_compute/sidecar_workstation/manual/agriculture/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/agriculture/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Attractions | [Worksheet](lake_compute/sidecar_workstation/manual/attractions/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/attractions/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Construction | [Worksheet](lake_compute/sidecar_workstation/manual/construction/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/construction/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Consumer Packaged Goods | [Worksheet](lake_compute/sidecar_workstation/manual/consumer_packaged_goods/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/consumer_packaged_goods/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Customer Service | [Worksheet](lake_compute/sidecar_workstation/manual/customer_service/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/customer_service/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Energy | [Worksheet](lake_compute/sidecar_workstation/manual/energy/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/energy/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Financial Services | [Worksheet](lake_compute/sidecar_workstation/manual/financial_services/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/financial_services/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Healthcare | [Worksheet](lake_compute/sidecar_workstation/manual/healthcare/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/healthcare/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Higher Education | [Worksheet](lake_compute/sidecar_workstation/manual/higher_education/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/higher_education/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Hospitality | [Worksheet](lake_compute/sidecar_workstation/manual/hospitality/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/hospitality/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Insurance | [Worksheet](lake_compute/sidecar_workstation/manual/insurance/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/insurance/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Manufacturing | [Worksheet](lake_compute/sidecar_workstation/manual/manufacturing/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/manufacturing/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Media Entertainment | [Worksheet](lake_compute/sidecar_workstation/manual/media_entertainment/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/media_entertainment/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Pharma | [Worksheet](lake_compute/sidecar_workstation/manual/pharma/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/pharma/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Retail | [Worksheet](lake_compute/sidecar_workstation/manual/retail/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/retail/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Sales | [Worksheet](lake_compute/sidecar_workstation/manual/sales/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/sales/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Supply Chain | [Worksheet](lake_compute/sidecar_workstation/manual/supply_chain/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/supply_chain/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Telco | [Worksheet](lake_compute/sidecar_workstation/manual/telco/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/telco/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Transportation | [Worksheet](lake_compute/sidecar_workstation/manual/transportation/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/transportation/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Utilities | [Worksheet](lake_compute/sidecar_workstation/manual/utilities/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/utilities/lake_compute_sidecar_workstation_hands_on_lab.md) |
| Weather | [Worksheet](lake_compute/sidecar_workstation/manual/weather/lake_compute_sidecar_workstation_hands_on_lab.md) | [Worksheet](lake_compute/sidecar_workstation/wizard/weather/lake_compute_sidecar_workstation_hands_on_lab.md) |
