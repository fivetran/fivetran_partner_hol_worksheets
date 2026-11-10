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

Manual editions are in [`manual/`](manual/) and Wizard editions in [`wizard/`](wizard/),
one folder per industry in each (for example `manual/agriculture/` and `wizard/agriculture/`).

For BigQuery, each participant configures **Your profile → Credentials → their assigned
project → Edit** using their own dbt login. Enter the **BigQuery dbt dataset** from the
assigned 1Password lab item, keep the provisioned Service Account JSON connection and
default target name, then **Test connection** until **Complete** is green. Click **Save**,
confirm **Saved**, and return to Studio to wait for **Connected**. These are personal
credentials; participants must not edit the shared development environment or connection.

| Industry / dataset | Snowflake + Wizard | BigQuery + Wizard | Snowflake manual | BigQuery manual |
|---|---|---|---|---|
| Agriculture | [Worksheet](wizard/agriculture/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/agriculture/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/agriculture/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/agriculture/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Attractions | [Worksheet](wizard/attractions/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/attractions/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/attractions/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/attractions/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Construction | [Worksheet](wizard/construction/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/construction/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/construction/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/construction/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Consumer Packaged Goods | [Worksheet](wizard/consumer_packaged_goods/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/consumer_packaged_goods/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/consumer_packaged_goods/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/consumer_packaged_goods/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Customer Service | [Worksheet](wizard/customer_service/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/customer_service/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/customer_service/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/customer_service/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Energy | [Worksheet](wizard/energy/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/energy/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/energy/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/energy/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Financial Services | [Worksheet](wizard/financial_services/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/financial_services/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/financial_services/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/financial_services/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Healthcare | [Worksheet](wizard/healthcare/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/healthcare/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/healthcare/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/healthcare/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Higher Education | [Worksheet](wizard/higher_education/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/higher_education/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/higher_education/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/higher_education/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Hospitality | [Worksheet](wizard/hospitality/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/hospitality/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/hospitality/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/hospitality/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Insurance | [Worksheet](wizard/insurance/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/insurance/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/insurance/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/insurance/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Manufacturing | [Worksheet](wizard/manufacturing/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/manufacturing/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/manufacturing/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/manufacturing/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Media Entertainment | [Worksheet](wizard/media_entertainment/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/media_entertainment/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/media_entertainment/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/media_entertainment/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Pharma | [Worksheet](wizard/pharma/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/pharma/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/pharma/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/pharma/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Retail | [Worksheet](wizard/retail/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/retail/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/retail/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/retail/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Sales | [Worksheet](wizard/sales/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/sales/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/sales/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/sales/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Supply Chain | [Worksheet](wizard/supply_chain/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/supply_chain/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/supply_chain/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/supply_chain/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Telco | [Worksheet](wizard/telco/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/telco/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/telco/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/telco/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Transportation | [Worksheet](wizard/transportation/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/transportation/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/transportation/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/transportation/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Utilities | [Worksheet](wizard/utilities/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/utilities/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/utilities/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/utilities/bigquery_manual_dbt_platform_hands_on_lab.md) |
| Weather | [Worksheet](wizard/weather/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/weather/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/weather/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/weather/bigquery_manual_dbt_platform_hands_on_lab.md) |
| NewsAPI | [Worksheet](wizard/newsapi/snowflake_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](wizard/newsapi/bigquery_wizard_dbt_platform_hands_on_lab.md) | [Worksheet](manual/newsapi/snowflake_manual_dbt_platform_hands_on_lab.md) | [Worksheet](manual/newsapi/bigquery_manual_dbt_platform_hands_on_lab.md) |

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
