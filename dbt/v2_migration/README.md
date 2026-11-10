# dbt v1 to v2 migration worksheets

Start with a working v1 project, reproduce v2 compatibility errors, repair
those errors with dbt Wizard, check that the business result is unchanged, compare dbt
State decisions, and run the migrated code in a dedicated Production environment.

This series covers **21 industries and NewsAPI**, with Snowflake and BigQuery editions
for each dataset (44 worksheets). Each worksheet includes complete legacy starter files and a
small migration answer, so the intentional failure does not depend on an AI-generated
mistake. The starter retains custom metadata outside `config.meta` and generic-test
inputs outside `arguments`.

| Dataset | Snowflake | BigQuery |
|---|---|---|
| Agriculture — coffee price monitoring | [Worksheet](agriculture/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](agriculture/bigquery_v2_migration_hands_on_lab.md) |
| Attractions | [Worksheet](attractions/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](attractions/bigquery_v2_migration_hands_on_lab.md) |
| Construction | [Worksheet](construction/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](construction/bigquery_v2_migration_hands_on_lab.md) |
| Consumer Packaged Goods | [Worksheet](consumer_packaged_goods/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](consumer_packaged_goods/bigquery_v2_migration_hands_on_lab.md) |
| Customer Service | [Worksheet](customer_service/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](customer_service/bigquery_v2_migration_hands_on_lab.md) |
| Energy | [Worksheet](energy/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](energy/bigquery_v2_migration_hands_on_lab.md) |
| Financial Services | [Worksheet](financial_services/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](financial_services/bigquery_v2_migration_hands_on_lab.md) |
| Healthcare | [Worksheet](healthcare/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](healthcare/bigquery_v2_migration_hands_on_lab.md) |
| Higher Education | [Worksheet](higher_education/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](higher_education/bigquery_v2_migration_hands_on_lab.md) |
| Hospitality | [Worksheet](hospitality/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](hospitality/bigquery_v2_migration_hands_on_lab.md) |
| Insurance | [Worksheet](insurance/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](insurance/bigquery_v2_migration_hands_on_lab.md) |
| Manufacturing | [Worksheet](manufacturing/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](manufacturing/bigquery_v2_migration_hands_on_lab.md) |
| Media Entertainment | [Worksheet](media_entertainment/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](media_entertainment/bigquery_v2_migration_hands_on_lab.md) |
| NewsAPI / Connector SDK | [Worksheet](newsapi/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](newsapi/bigquery_v2_migration_hands_on_lab.md) |
| Pharma | [Worksheet](pharma/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](pharma/bigquery_v2_migration_hands_on_lab.md) |
| Retail | [Worksheet](retail/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](retail/bigquery_v2_migration_hands_on_lab.md) |
| Sales | [Worksheet](sales/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](sales/bigquery_v2_migration_hands_on_lab.md) |
| Supply Chain | [Worksheet](supply_chain/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](supply_chain/bigquery_v2_migration_hands_on_lab.md) |
| Telco | [Worksheet](telco/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](telco/bigquery_v2_migration_hands_on_lab.md) |
| Transportation | [Worksheet](transportation/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](transportation/bigquery_v2_migration_hands_on_lab.md) |
| Utilities | [Worksheet](utilities/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](utilities/bigquery_v2_migration_hands_on_lab.md) |
| Weather | [Worksheet](weather/snowflake_v2_migration_hands_on_lab.md) | [Worksheet](weather/bigquery_v2_migration_hands_on_lab.md) |

LabStudios provides each attendee's project, staging models, Development and Production
environments, and databases or datasets, with Wizard, dbt v2 and State available.
Part 3 walks through creating the project configuration, models, tests and logging macro
inline, one file at a time.
