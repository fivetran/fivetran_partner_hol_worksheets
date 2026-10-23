# Retail: Snowflake Cortex add-on

Build an assistant for **retail inventory priorities** using the data you transformed in dbt. Create a native semantic view, connect it to a Cortex Agent through Cortex Analyst, then check its answers against the mart.

**Lab sequence:** basic Fivetran lab → dbt Platform add-on → this optional Snowflake Cortex add-on. Allow approximately 30–45 minutes. This worksheet works after either Snowflake dbt edition; there is no BigQuery equivalent in this series.

## 1. Start from the completed dbt lab

Complete either the [Snowflake + dbt Wizard worksheet](../dbt/retail/snowflake_wizard_dbt_platform_hands_on_lab.md) or the [Snowflake manual worksheet](../dbt/retail/snowflake_manual_dbt_platform_hands_on_lab.md). The [template repository](https://github.com/dbt-Hands-on-Labs/dbt-retail-lab) supplies the starting project; the completed dbt exercise supplies the mart used here.

1. Confirm the dbt build and reconciliation tests passed after restoring `review_threshold: 0`. Use the restored **development** output, not the separate deployment schema.
2. Record `<DEV_DATABASE>` and `<DBT_SCHEMA>` from that build. `<DBT_SCHEMA>` is the mart schema (for example, `dbt_firstname_lastname`), without the staging suffix.
3. Confirm `fct_retail_inventory_weekly` and `mart_retail_inventory_priorities` were built. The semantic view queries **`<DEV_DATABASE>.<DBT_SCHEMA>.MART_RETAIL_INVENTORY_PRIORITIES`** only.
4. Open Snowsight with your provisioned Snowflake lab user and role. This optional module requires Snowflake access for semantic-view authoring and agent chat, in addition to dbt Platform access.
5. The instructor must provision permission to read your mart, use the lab warehouse, create semantic views and agents in `<DEV_DATABASE>.APPS`, and use Cortex. Have the instructor resolve missing privileges; attendees do not issue grants. Confirm a usable default warehouse or select the assigned warehouse when adding the tool.

Use the instructor's compute and model allowance. Keep queries bounded and use the draft agent preview. Direct destination-data previews and SQL comparison queries below are instructor-led; attendees create the semantic view and agent, ask questions, and inspect agent traces.

## 2. Understand the transformed data

- **Grain:** One latest observation per product/store, selected by inventory_date descending (nulls last), then row_id ascending.
- **Rule:** needs_replenishment is true when days_of_supply < restock_lead_time_days. The restored review_threshold is 0 additional lead-time days.
- **Interpretation:** Historical latest observations are not live inventory or a demand forecast. The mart does not contain a historical time series or revenue.
- **Reading results:** Stock and weekly sales are units, not currency. A product appearing in two stores counts twice. The transformed mart has no anomaly_type column; an anomaly explanation cannot be inferred from is_anomaly.

A semantic fact is a value on a mart row; a semantic metric aggregates those values. A dbt model named `fct_...` is a separate concept. The YAML below uses the mart's actual output columns and defines aggregations at its grain.

## 3. Create the semantic view

1. In Snowsight, open **Projects → Workspaces**. Choose **Add new → Semantic view → Start blank** and open the **YAML** editor (labels can vary by account release).
2. Paste the complete definition below. Replace `<DEV_DATABASE>` and `<DBT_SCHEMA>` with your development identifiers; preserve the physical mart name. Use the ordinary unquoted lab identifiers, not the landing database or production schema.

```yaml
name: RETAIL_ASSISTANT
description: Retail inventory priorities over the completed dbt mart. Historical latest observations are
  not live inventory or a demand forecast. The mart does not contain a historical time series or revenue.
tables:
- name: inventory
  description: One latest observation per product/store, selected by inventory_date descending (nulls
    last), then row_id ascending. needs_replenishment is true when days_of_supply < restock_lead_time_days.
    The restored review_threshold is 0 additional lead-time days.
  base_table:
    database: <DEV_DATABASE>
    schema: <DBT_SCHEMA>
    table: MART_RETAIL_INVENTORY_PRIORITIES
  primary_key:
    columns:
    - product_store_key
  dimensions:
  - name: product_store_key
    expr: PRODUCT_STORE_KEY
    data_type: VARCHAR
  - name: product_id
    expr: PRODUCT_ID
    data_type: VARCHAR
  - name: product_name
    expr: PRODUCT_NAME
    data_type: VARCHAR
  - name: store_id
    expr: STORE_ID
    data_type: VARCHAR
  - name: store_name
    expr: STORE_NAME
    data_type: VARCHAR
  - name: category
    expr: CATEGORY
    data_type: VARCHAR
  - name: region
    expr: REGION
    data_type: VARCHAR
  - name: needs_replenishment
    expr: NEEDS_REPLENISHMENT
    data_type: BOOLEAN
  - name: is_anomaly
    expr: IS_ANOMALY
    data_type: BOOLEAN
  time_dimensions:
  - name: inventory_date
    expr: INVENTORY_DATE
    data_type: DATE
    description: Date of the selected observation; not necessarily today.
  facts:
  - name: days_of_supply
    expr: DAYS_OF_SUPPLY
    data_type: NUMBER
  - name: restock_lead_time_days
    expr: RESTOCK_LEAD_TIME_DAYS
    data_type: NUMBER
  - name: current_stock_level
    expr: CURRENT_STOCK_LEVEL
    data_type: NUMBER
  - name: weekly_sales_volume
    expr: WEEKLY_SALES_VOLUME
    data_type: NUMBER
  - name: lead_time_gap_days
    expr: RESTOCK_LEAD_TIME_DAYS - DAYS_OF_SUPPLY
    data_type: NUMBER
  metrics:
  - name: product_store_count
    expr: COUNT(*)
  - name: replenishment_count
    expr: SUM(CASE WHEN needs_replenishment THEN 1 ELSE 0 END)
  - name: total_stock_units
    expr: SUM(current_stock_level)
  - name: average_days_of_supply
    expr: AVG(days_of_supply)
module_custom_instructions:
  sql_generation: needs_replenishment is true when days_of_supply < restock_lead_time_days. The restored
    review_threshold is 0 additional lead-time days. Historical latest observations are not live inventory
    or a demand forecast. The mart does not contain a historical time series or revenue. Count product/store
    rows, not distinct products. Include inventory_date in priority lists. Rank lead_time_gap_days descending,
    then product_id and store_id. Never filter historical data to CURRENT_DATE. Null comparisons are unknown,
    not evidence of adequate stock.
verified_queries:
- name: baseline_totals
  question: How many product/store combinations need replenishment? Show the observation date range for
    those combinations.
  sql: SELECT COUNT(*) AS replenishment_count, MIN(inventory_date) AS first_observation, MAX(inventory_date)
    AS last_observation FROM __inventory WHERE needs_replenishment = TRUE
- name: top_three
  question: Which three product/store combinations have the largest replenishment gap? Include IDs, names,
    inventory date, days of supply, restock lead time and gap. Break ties by product ID and store ID.
  sql: SELECT product_id, product_name, store_id, store_name, inventory_date, days_of_supply, restock_lead_time_days,
    lead_time_gap_days FROM __inventory WHERE needs_replenishment = TRUE ORDER BY lead_time_gap_days DESC,
    product_id, store_id LIMIT 3
```

3. Choose **Publish**, use `dev` as the workspace target label, and select **your development database → APPS**. Confirm the resulting object is `<DEV_DATABASE>.APPS.RETAIL_ASSISTANT`.
4. Confirm the view validates and publishes successfully. Fix missing tables or columns before continuing. The `verified_queries` entries are reference examples for Analyst, not evidence of a live run in your account; you will compare them with current results in section 5.

Logical names such as `__inventory` in those examples resolve through the semantic definition to your physical mart. Publishing the semantic view creates metadata over the dbt output; it does not rebuild or copy the mart.

## 4. Configure a draft Cortex Agent

1. Open **AI & ML → Agent Studio → Create agent**. Select `<DEV_DATABASE>.APPS`, name the object `RETAIL_AGENT`, and use **Retail Assistant** as its display name.
2. Under **Configuration → General**, describe its purpose as: “Explore retail inventory priorities from tested dbt output; answer only questions supported by this mart.”
3. Under **Tools**, turn off Web search and Code Execution if enabled, and leave Analytical search off. Under **Query structured data**, choose **Add semantic view**.
4. Select both your development database and **APPS** schema, then `RETAIL_ASSISTANT`. Name the Cortex Analyst tool `retail_analytics`. Describe it as: “Query retail inventory priorities using the documented mart grain, counts and rules.”
5. Use your verified lab default warehouse, or select the instructor-provided custom warehouse. Set the query timeout to `60` seconds and add the tool.
6. Under **Instructions**, keep model selection `auto`. Set orchestration instructions to:

```text
Use retail_analytics for every quantitative answer. Query the current mart instead of relying on conversation memory. needs_replenishment is true when days_of_supply < restock_lead_time_days. The restored review_threshold is 0 additional lead-time days. Historical latest observations are not live inventory or a demand forecast. The mart does not contain a historical time series or revenue. Count product/store rows, not distinct products. Include inventory_date in priority lists. Rank lead_time_gap_days descending, then product_id and store_id. Never filter historical data to CURRENT_DATE. Null comparisons are unknown, not evidence of adequate stock.
```

7. Set response instructions to:

```text
Give concise answers grounded in successful tool results. Explain the grain, denominator and missing values when relevant. Identify these as lab observations, not live predictions. State when a request needs columns or history absent from the semantic view. Do not invent a result when a query fails.
```

8. If available, set a time limit of `120` seconds and a token limit of `8000` for these small questions. Follow the instructor's account limits if they differ. Save the agent as a **draft** and open **Preview**. Agent publication, sharing and CoWork integration are outside this exercise.

## 5. Ask questions and verify answers

1. Ask the first question:

```text
How many product/store combinations need replenishment? Show the observation date range for those combinations.
```

2. Open **Show Traces → SQL Execution** (or the equivalent execution detail). Inspect the SQL, results and status. Confirm a successful Analyst query uses your semantic view/mart and the documented grain and denominators. A plausible answer alone is insufficient.
3. Ask the second question:

```text
Which three product/store combinations have the largest replenishment gap? Include IDs, names, inventory date, days of supply, restock lead time and gap. Break ties by product ID and store ID.
```

4. The instructor runs these two queries with your development database/schema and lab warehouse, then shares the results for comparison. They must use the same restored dbt output as your agent. Expected counts vary by synced data; compare actual values, not a hard-coded answer.

```sql
SELECT COUNT(*) AS replenishment_count, MIN(inventory_date) AS first_observation, MAX(inventory_date) AS last_observation FROM <DEV_DATABASE>.<DBT_SCHEMA>.MART_RETAIL_INVENTORY_PRIORITIES WHERE needs_replenishment = TRUE;
```

```sql
SELECT product_id, product_name, store_id, store_name, inventory_date, days_of_supply, restock_lead_time_days, (restock_lead_time_days - days_of_supply) AS lead_time_gap_days FROM <DEV_DATABASE>.<DBT_SCHEMA>.MART_RETAIL_INVENTORY_PRIORITIES WHERE needs_replenishment = TRUE ORDER BY lead_time_gap_days DESC, product_id, store_id LIMIT 3;
```

5. Compare totals, missing-value treatment, ordering and any fractions with the agent's trace and final answer. Empty output or a null rate can be valid; explain it rather than substituting zero. For decimal results, allow display rounding but verify the denominator.
6. Ask a question outside the mart's scope:

```text
Forecast next month’s revenue and explain the historical replenishment trend.
```

7. Confirm the agent explains the missing data or limitation instead of inventing an answer. If it does not, refine its instructions, save, and retry. AI responses can vary; this is a check to perform, not a guaranteed outcome.

## 6. Finish and troubleshoot

You are finished when your semantic view is valid, the draft agent answers both supported questions with successful query traces, its results match the instructor's SQL, and it handles the unsupported question appropriately. Keep the draft and view for the instructor's review.

- **Missing mart or columns:** compare against the completed dbt worksheet's output contract and rerun its build if necessary. The starter repository alone does not contain the finished mart.
- **Wrong counts:** check the development database/schema, restored threshold, latest successful build and selected tool. Do not compare deployment output with development output.
- **Permission or warehouse error:** ask the instructor to check the assigned role, APPS privileges, mart access and lab warehouse.
- **Unsupported answer or wrong aggregation:** inspect the SQL trace, correct the semantic definition or instructions, republish the view, and retry the draft agent.

## References

- [Reference worksheet, Part 8](../transformations/fivetran_dbt_snowflake_hands_on_lab.md#part-8-ask-questions-with-snowflake-cortex)
- [Snowflake semantic-view YAML specification](https://docs.snowflake.com/en/user-guide/views-semantic/semantic-view-yaml-spec)
- [Create and manage Cortex Agents](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents-manage)
- [Add-on index and validation scope](README.md)
