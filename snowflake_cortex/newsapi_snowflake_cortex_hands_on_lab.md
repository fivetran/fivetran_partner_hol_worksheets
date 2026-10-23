# NewsAPI: Snowflake Cortex add-on

Build an assistant for **news source coverage** using the data you transformed in dbt. Create a native semantic view, connect it to a Cortex Agent through Cortex Analyst, then check its answers against the mart.

**Lab sequence:** basic Fivetran lab → dbt Platform add-on → this optional Snowflake Cortex add-on. Allow approximately 30–45 minutes. This worksheet works after either Snowflake dbt edition; there is no BigQuery equivalent in this series.

## 1. Start from the completed dbt lab

Complete either the [Snowflake + dbt Wizard worksheet](../dbt/newsapi/snowflake_wizard_dbt_platform_hands_on_lab.md) or the [Snowflake manual worksheet](../dbt/newsapi/snowflake_manual_dbt_platform_hands_on_lab.md). The [template repository](https://github.com/dbt-Hands-on-Labs/dbt-newsapi-lab) supplies the starting project; the completed dbt exercise supplies the mart used here.

1. Confirm the dbt build and reconciliation tests passed after restoring `review_threshold: 40`. Use the restored **development** output, not the separate deployment schema.
2. Record `<DEV_DATABASE>` and `<DBT_SCHEMA>` from that build. `<DBT_SCHEMA>` is the mart schema (for example, `dbt_firstname_lastname`), without the staging suffix.
3. Confirm `fct_newsapi_articles` and `mart_newsapi_source_coverage` were built. The semantic view queries **`<DEV_DATABASE>.<DBT_SCHEMA>.MART_NEWSAPI_SOURCE_COVERAGE`** only.
4. Open Snowsight with your provisioned Snowflake lab user and role. This optional module requires Snowflake access for semantic-view authoring and agent chat, in addition to dbt Platform access.
5. The instructor must provision permission to read your mart, use the lab warehouse, create semantic views and agents in `<DEV_DATABASE>.APPS`, and use Cortex. Have the instructor resolve missing privileges; attendees do not issue grants. Confirm a usable default warehouse or select the assigned warehouse when adding the tool.

Use the instructor's compute and model allowance. Keep queries bounded and use the draft agent preview. Direct destination-data previews and SQL comparison queries below are instructor-led; attendees create the semantic view and agent, ask questions, and inspect agent traces.

## 2. Understand the transformed data

- **Grain:** One row per source_key after article URL deduplication. The missing: bucket may combine multiple publishers; example_source_name is only an example name.
- **Rule:** short_title_count counts non-null titles shorter than 40 characters after restoring the dbt baseline. Missing titles are counted separately.
- **Interpretation:** Coverage is limited to the configured API endpoints, filters and synced rows; it is not the complete news universe. Publication dates are not sync times. This summary contains no article bodies and cannot summarize news stories or infer sentiment.
- **Reading results:** The standard connector syncs top_headlines and sources. Include everything only if the completed dbt lab enabled and built it. Counts depend on filters, available data and deduplication; no fixed expected count applies. Short titles are a formatting-review signal, not a quality or credibility score.

A semantic fact is a value on a mart row; a semantic metric aggregates those values. A dbt model named `fct_...` is a separate concept. The YAML below uses the mart's actual output columns and defines aggregations at its grain.

## 3. Create the semantic view

1. In Snowsight, open **Projects → Workspaces**. Choose **Add new → Semantic view → Start blank** and open the **YAML** editor (labels can vary by account release).
2. Paste the complete definition below. Replace `<DEV_DATABASE>` and `<DBT_SCHEMA>` with your development identifiers; preserve the physical mart name. Use the ordinary unquoted lab identifiers, not the landing database or production schema.

```yaml
name: NEWSAPI_ASSISTANT
description: News source coverage over the completed dbt mart. Coverage is limited to the configured API
  endpoints, filters and synced rows; it is not the complete news universe. Publication dates are not
  sync times. This summary contains no article bodies and cannot summarize news stories or infer sentiment.
tables:
- name: coverage
  description: 'One row per source_key after article URL deduplication. The missing: bucket may combine
    multiple publishers; example_source_name is only an example name. short_title_count counts non-null
    titles shorter than 40 characters after restoring the dbt baseline. Missing titles are counted separately.'
  base_table:
    database: <DEV_DATABASE>
    schema: <DBT_SCHEMA>
    table: MART_NEWSAPI_SOURCE_COVERAGE
  primary_key:
    columns:
    - source_key
  dimensions:
  - name: source_key
    expr: SOURCE_KEY
    data_type: VARCHAR
  - name: example_source_name
    expr: EXAMPLE_SOURCE_NAME
    data_type: VARCHAR
    description: Example label only; not a canonical publisher identity.
  time_dimensions:
  - name: first_publication
    expr: FIRST_PUBLICATION
    data_type: TIMESTAMP_NTZ
  - name: last_publication
    expr: LAST_PUBLICATION
    data_type: TIMESTAMP_NTZ
  facts:
  - name: article_count
    expr: ARTICLE_COUNT
    data_type: NUMBER
  - name: matched_articles
    expr: MATCHED_ARTICLES
    data_type: NUMBER
  - name: short_title_count
    expr: SHORT_TITLE_COUNT
    data_type: NUMBER
  - name: missing_title_count
    expr: MISSING_TITLE_COUNT
    data_type: NUMBER
  metrics:
  - name: total_articles
    expr: SUM(article_count)
  - name: directory_coverage
    expr: SUM(matched_articles) / NULLIF(SUM(article_count), 0)
  - name: short_title_share
    expr: SUM(short_title_count) / NULLIF(SUM(article_count - missing_title_count), 0)
  - name: total_missing_titles
    expr: SUM(missing_title_count)
module_custom_instructions:
  sql_generation: short_title_count counts non-null titles shorter than 40 characters after restoring
    the dbt baseline. Missing titles are counted separately. Coverage is limited to the configured API
    endpoints, filters and synced rows; it is not the complete news universe. Publication dates are not
    sync times. This summary contains no article bodies and cannot summarize news stories or infer sentiment.
    Sum stored article counts, not COUNT(*), which counts source buckets. Use source_key as identity and
    example_source_name only as a label. Short-title share is SUM(short_title_count)/NULLIF(SUM(article_count-missing_title_count),0).
    Directory coverage is SUM(matched_articles)/NULLIF(SUM(article_count),0). Missing source IDs can combine
    publishers; do not call buckets distinct publishers. Never require all articles to match the directory.
verified_queries:
- name: baseline_totals
  question: How many synced articles are represented? Show directory-matched articles, directory coverage,
    short titles, missing titles and short-title share among known titles.
  sql: SELECT SUM(article_count) AS articles, SUM(matched_articles) AS matched, SUM(matched_articles)
    / NULLIF(SUM(article_count), 0) AS directory_coverage, SUM(short_title_count) AS short_titles, SUM(missing_title_count)
    AS missing_titles, SUM(short_title_count) / NULLIF(SUM(article_count - missing_title_count), 0) AS
    short_title_share FROM __coverage
- name: top_three
  question: Which three source buckets contain the most articles? Include source key, example name, matched
    articles and publication range. Break ties by source key.
  sql: SELECT source_key, example_source_name, article_count, matched_articles, first_publication, last_publication
    FROM __coverage ORDER BY article_count DESC, source_key LIMIT 3
```

3. Choose **Publish**, use `dev` as the workspace target label, and select **your development database → APPS**. Confirm the resulting object is `<DEV_DATABASE>.APPS.NEWSAPI_ASSISTANT`.
4. Confirm the view validates and publishes successfully. Fix missing tables or columns before continuing. The `verified_queries` entries are reference examples for Analyst, not evidence of a live run in your account; you will compare them with current results in section 5.

Logical names such as `__coverage` in those examples resolve through the semantic definition to your physical mart. Publishing the semantic view creates metadata over the dbt output; it does not rebuild or copy the mart.

## 4. Configure a draft Cortex Agent

1. Open **AI & ML → Agent Studio → Create agent**. Select `<DEV_DATABASE>.APPS`, name the object `NEWSAPI_AGENT`, and use **NewsAPI Assistant** as its display name.
2. Under **Configuration → General**, describe its purpose as: “Explore news source coverage from tested dbt output; answer only questions supported by this mart.”
3. Under **Tools**, turn off Web search and Code Execution if enabled, and leave Analytical search off. Under **Query structured data**, choose **Add semantic view**.
4. Select both your development database and **APPS** schema, then `NEWSAPI_ASSISTANT`. Name the Cortex Analyst tool `newsapi_analytics`. Describe it as: “Query news source coverage using the documented mart grain, counts and rules.”
5. Use your verified lab default warehouse, or select the instructor-provided custom warehouse. Set the query timeout to `60` seconds and add the tool.
6. Under **Instructions**, keep model selection `auto`. Set orchestration instructions to:

```text
Use newsapi_analytics for every quantitative answer. Query the current mart instead of relying on conversation memory. short_title_count counts non-null titles shorter than 40 characters after restoring the dbt baseline. Missing titles are counted separately. Coverage is limited to the configured API endpoints, filters and synced rows; it is not the complete news universe. Publication dates are not sync times. This summary contains no article bodies and cannot summarize news stories or infer sentiment. Sum stored article counts, not COUNT(*), which counts source buckets. Use source_key as identity and example_source_name only as a label. Short-title share is SUM(short_title_count)/NULLIF(SUM(article_count-missing_title_count),0). Directory coverage is SUM(matched_articles)/NULLIF(SUM(article_count),0). Missing source IDs can combine publishers; do not call buckets distinct publishers. Never require all articles to match the directory.
```

7. Set response instructions to:

```text
Give concise answers grounded in successful tool results. Explain the grain, denominator and missing values when relevant. Identify these as lab observations, not live predictions. State when a request needs columns or history absent from the semantic view. Do not invent a result when a query fails.
```

8. If available, set a time limit of `120` seconds and a token limit of `8000` for these small questions. Follow the instructor's account limits if they differ. Save the agent as a **draft** and open **Preview**. Agent publication, sharing and CoWork integration are outside this exercise.

## 5. Ask questions and verify answers

1. Ask the first question:

```text
How many synced articles are represented? Show directory-matched articles, directory coverage, short titles, missing titles and short-title share among known titles.
```

2. Open **Show Traces → SQL Execution** (or the equivalent execution detail). Inspect the SQL, results and status. Confirm a successful Analyst query uses your semantic view/mart and the documented grain and denominators. A plausible answer alone is insufficient.
3. Ask the second question:

```text
Which three source buckets contain the most articles? Include source key, example name, matched articles and publication range. Break ties by source key.
```

4. The instructor runs these two queries with your development database/schema and lab warehouse, then shares the results for comparison. They must use the same restored dbt output as your agent. Expected counts vary by synced data; compare actual values, not a hard-coded answer.

```sql
SELECT SUM(article_count) AS articles, SUM(matched_articles) AS matched, SUM(matched_articles) / NULLIF(SUM(article_count), 0) AS directory_coverage, SUM(short_title_count) AS short_titles, SUM(missing_title_count) AS missing_titles, SUM(short_title_count) / NULLIF(SUM(article_count - missing_title_count), 0) AS short_title_share FROM <DEV_DATABASE>.<DBT_SCHEMA>.MART_NEWSAPI_SOURCE_COVERAGE;
```

```sql
SELECT source_key, example_source_name, article_count, matched_articles, first_publication, last_publication FROM <DEV_DATABASE>.<DBT_SCHEMA>.MART_NEWSAPI_SOURCE_COVERAGE ORDER BY article_count DESC, source_key LIMIT 3;
```

5. Compare totals, missing-value treatment, ordering and any fractions with the agent's trace and final answer. Empty output or a null rate can be valid; explain it rather than substituting zero. For decimal results, allow display rounding but verify the denominator.
6. Ask a question outside the mart's scope:

```text
Summarize today’s global news and rank publisher credibility.
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
