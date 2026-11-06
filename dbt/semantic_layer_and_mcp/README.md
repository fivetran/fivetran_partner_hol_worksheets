# dbt Semantic Layer and MCP add-on worksheets

One worksheet per industry and warehouse. Each continues a dbt Platform add-on edition
([manual or Wizard](../README.md)) after Part 3, once the attendee has written the marts and
tests but not built them. Attendees then:

1. add a provided Semantic Layer input model, a time spine and a semantic model with
   simple, ratio and derived metrics and a saved query;
2. build and validate them in their Development environment (`dbt sl validate`, `dbt sl query`);
3. run a job in the Production environment LabStudios created for them, which publishes the metrics;
4. create a dbt personal access token and find their dbt user ID;
5. connect Claude Code on their lab workstation to the dbt remote MCP server and ask five
   questions that exercise metric discovery, queries, dimension values, compiled SQL,
   lineage and model health.

The lab must be launched in LabStudios with an individual dbt project per attendee and
**dbt Semantic Layer + remote MCP** turned on. LabStudios creates the Production environment
and credential, configures the Semantic Layer, provisions a workstation per attendee and puts
the `dbt MCP Command` in each attendee's 1Password item. Attendees work on the `lab-work`
branch, which Production runs.

| Industry / dataset | Business question | Snowflake | BigQuery | Variant |
|---|---|---|---|---|
| Agriculture | Coffee price monitoring | [Worksheet](agriculture/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](agriculture/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Attractions | Air passenger demand | [Worksheet](attractions/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](attractions/bigquery_semantic_layer_mcp_hands_on_lab.md) | Month grain |
| Construction | Construction risk review | [Worksheet](construction/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](construction/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Consumer Packaged Goods | Stockout review | [Worksheet](consumer_packaged_goods/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](consumer_packaged_goods/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Customer Service | Response-length review | [Worksheet](customer_service/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](customer_service/bigquery_semantic_layer_mcp_hands_on_lab.md) | No trend: the source has no interaction date |
| Energy | Crude price monitoring | [Worksheet](energy/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](energy/bigquery_semantic_layer_mcp_hands_on_lab.md) | Unpivot: one row per day and commodity |
| Financial Services | Loan amount review | [Worksheet](financial_services/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](financial_services/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Healthcare | Claim payment reconciliation | [Worksheet](healthcare/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](healthcare/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Higher Education | Student engagement review | [Worksheet](higher_education/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](higher_education/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Hospitality | Guest transport review | [Worksheet](hospitality/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](hospitality/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Insurance | Insurance claim severity | [Worksheet](insurance/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](insurance/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Manufacturing | Material waste review | [Worksheet](manufacturing/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](manufacturing/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Media & Entertainment | Catalog vintage | [Worksheet](media_entertainment/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](media_entertainment/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Pharma | Trial dropout review | [Worksheet](pharma/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](pharma/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Retail | Retail inventory priorities | [Worksheet](retail/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](retail/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Sales | Renewal review | [Worksheet](sales/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](sales/bigquery_semantic_layer_mcp_hands_on_lab.md) | Quarter grain |
| Supply Chain | Forecast error review | [Worksheet](supply_chain/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](supply_chain/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Telco | Churn review | [Worksheet](telco/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](telco/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Transportation | Arrival delay review | [Worksheet](transportation/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](transportation/bigquery_semantic_layer_mcp_hands_on_lab.md) | Month grain; weighted vs unweighted delay |
| Utilities | Meter consumption review | [Worksheet](utilities/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](utilities/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| Weather | High-temperature review | [Worksheet](weather/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](weather/bigquery_semantic_layer_mcp_hands_on_lab.md) |  |
| NewsAPI | News source coverage | [Worksheet](newsapi/snowflake_semantic_layer_mcp_hands_on_lab.md) | [Worksheet](newsapi/bigquery_semantic_layer_mcp_hands_on_lab.md) | Week-over-week instead of year-over-year |
