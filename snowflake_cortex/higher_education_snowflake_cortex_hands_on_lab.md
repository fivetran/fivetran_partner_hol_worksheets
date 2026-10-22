# Higher Education: Snowflake Cortex add-on

Build an assistant for **student engagement review** using the data you transformed in dbt. Create a native semantic view, connect it to a Cortex Agent through Cortex Analyst, then check its answers against the mart.

**Lab sequence:** basic Fivetran lab → dbt Platform add-on → this optional Snowflake Cortex add-on. Allow approximately 30–45 minutes. This worksheet works after either Snowflake dbt edition; there is no BigQuery equivalent in this series.

## 1. Start from the completed dbt lab

Complete either the [Snowflake + dbt Wizard worksheet](../dbt/higher_education/snowflake_wizard_dbt_platform_hands_on_lab.md) or the [Snowflake manual worksheet](../dbt/higher_education/snowflake_manual_dbt_platform_hands_on_lab.md). The [template repository](https://github.com/dbt-Hands-on-Labs/dbt-higher-education-lab) supplies the starting project; the completed dbt exercise supplies the mart used here.

1. Confirm the dbt build and reconciliation tests passed after restoring `review_threshold: 40`. Use the restored **development** output, not the separate deployment schema.
2. Record `<DEV_DATABASE>` and `<DBT_SCHEMA>` from that build. `<DBT_SCHEMA>` is the mart schema (for example, `dbt_firstname_lastname`), without the staging suffix.
3. Confirm `fct_higher_education_observations` and `mart_higher_education_review_summary` were built. The semantic view queries **`<DEV_DATABASE>.<DBT_SCHEMA>.MART_HIGHER_EDUCATION_REVIEW_SUMMARY`** only.
4. Open Snowsight with your provisioned Snowflake lab user and role. This optional module requires Snowflake access for semantic-view authoring and agent chat, in addition to dbt Platform access.
5. The instructor must provision permission to read your mart, use the lab warehouse, create semantic views and agents in `<DEV_DATABASE>.APPS`, and use Cortex. Have the instructor resolve missing privileges; attendees do not issue grants. Confirm a usable default warehouse or select the assigned warehouse when adding the tool.

Use the instructor's compute and model allowance. Keep queries bounded and use the draft agent preview. Direct destination-data previews and SQL comparison queries below are instructor-led; attendees create the semantic view and agent, ask questions, and inspect agent traces.

## 2. Understand the transformed data

- **Grain:** One row per major_code group, identified by group_key. group_name is a display label; keep group_key to distinguish a missing group from a literal Unknown value.
- **Rule:** Flag an observation when record-level engagement score < 40. Null metrics remain unknown and are excluded from measured_count.
- **Interpretation:** Count observations, not distinct students; the illustrative threshold is not an academic decision. The summary has no observation dates or row-level details, so it cannot support trends or individual-record explanations.
- **Reading results:** Review share is a fraction from 0 to 1; label it as a percentage only after multiplying by 100. A group with no measured values has a null share. The summary cannot recover individual observations or dates.

A semantic fact is a value on a mart row; a semantic metric aggregates those values. A dbt model named `fct_...` is a separate concept. The YAML below uses the mart's actual output columns and defines aggregations at its grain.

## 3. Create the semantic view

1. In Snowsight, open **Projects → Workspaces**. Choose **Add new → Semantic view → Start blank** and open the **YAML** editor (labels can vary by account release).
2. Paste the complete definition below. Replace `<DEV_DATABASE>` and `<DBT_SCHEMA>` with your development identifiers; preserve the physical mart name. Use the ordinary unquoted lab identifiers, not the landing database or production schema.

```yaml
name: HIGHER_EDUCATION_ASSISTANT
description: Student engagement review over the completed dbt mart. Count observations, not distinct students;
  the illustrative threshold is not an academic decision. The summary has no observation dates or row-level
  details, so it cannot support trends or individual-record explanations.
tables:
- name: summary
  description: One row per major_code group, identified by group_key. group_name is a display label; keep
    group_key to distinguish a missing group from a literal Unknown value. Flag an observation when record-level
    engagement score < 40. Null metrics remain unknown and are excluded from measured_count.
  base_table:
    database: <DEV_DATABASE>
    schema: <DBT_SCHEMA>
    table: MART_HIGHER_EDUCATION_REVIEW_SUMMARY
  primary_key:
    columns:
    - group_key
  dimensions:
  - name: group_key
    expr: GROUP_KEY
    data_type: VARCHAR
  - name: group_name
    expr: GROUP_NAME
    data_type: VARCHAR
    description: 'Grouping field: major_code'
  facts:
  - name: observation_count
    expr: OBSERVATION_COUNT
    data_type: NUMBER
  - name: measured_count
    expr: MEASURED_COUNT
    data_type: NUMBER
  - name: missing_metric_count
    expr: MISSING_METRIC_COUNT
    data_type: NUMBER
  - name: review_count
    expr: REVIEW_COUNT
    data_type: NUMBER
  - name: mean_metric_value
    expr: MEAN_METRIC_VALUE
    data_type: FLOAT
    description: record-level engagement score; mean of measured observations in this group
  metrics:
  - name: total_observations
    expr: SUM(observation_count)
  - name: total_measured
    expr: SUM(measured_count)
  - name: total_missing
    expr: SUM(missing_metric_count)
  - name: total_review
    expr: SUM(review_count)
  - name: review_rate
    expr: SUM(review_count) / NULLIF(SUM(measured_count), 0)
  - name: observation_mean
    expr: SUM(mean_metric_value * measured_count) / NULLIF(SUM(measured_count), 0)
module_custom_instructions:
  sql_generation: Sum stored counts; COUNT(*) counts groups, not observations. Compute review share as
    SUM(review_count)/NULLIF(SUM(measured_count),0), never AVG of group shares. Reconstruct the observation
    mean as SUM(mean_metric_value*measured_count)/NULLIF(SUM(measured_count),0), never AVG of group means.
    This recovers an unweighted mean across measured observations; it does not introduce business weights.
    Null denominators mean unknown, not zero. Flag an observation when record-level engagement score <
    40. Null metrics remain unknown and are excluded from measured_count. Count observations, not distinct
    students; the illustrative threshold is not an academic decision. The summary has no observation dates
    or row-level details, so it cannot support trends or individual-record explanations.
verified_queries:
- name: baseline_totals
  question: How many observations meet the student engagement review rule? Show total observations, measured
    observations, missing metrics and review share.
  sql: SELECT SUM(observation_count) AS observations, SUM(measured_count) AS measured, SUM(missing_metric_count)
    AS missing, SUM(review_count) AS flagged, SUM(review_count) / NULLIF(SUM(measured_count), 0) AS review_share
    FROM __summary
- name: top_three
  question: Which three major_code groups have the most review observations? Include group keys, measured
    and missing counts, and review shares. Break ties by group key.
  sql: SELECT group_key, group_name, observation_count, measured_count, missing_metric_count, review_count,
    review_count / NULLIF(measured_count, 0) AS review_share FROM __summary ORDER BY review_count DESC,
    group_key LIMIT 3
```

3. Choose **Publish**, use `dev` as the workspace target label, and select **your development database → APPS**. Confirm the resulting object is `<DEV_DATABASE>.APPS.HIGHER_EDUCATION_ASSISTANT`.
4. Confirm the view validates and publishes successfully. Fix missing tables or columns before continuing. The `verified_queries` entries are reference examples for Analyst, not evidence of a live run in your account; you will compare them with current results in section 5.

Logical names such as `__summary` in those examples resolve through the semantic definition to your physical mart. Publishing the semantic view creates metadata over the dbt output; it does not rebuild or copy the mart.

## 4. Configure a draft Cortex Agent

1. Open **AI & ML → Agent Studio → Create agent**. Select `<DEV_DATABASE>.APPS`, name the object `HIGHER_EDUCATION_AGENT`, and use **Higher Education Assistant** as its display name.
2. Under **Configuration → General**, describe its purpose as: “Explore student engagement review from tested dbt output; answer only questions supported by this mart.”
3. Under **Tools**, turn off Web search and Code Execution if enabled, and leave Analytical search off. Under **Query structured data**, choose **Add semantic view**.
4. Select both your development database and **APPS** schema, then `HIGHER_EDUCATION_ASSISTANT`. Name the Cortex Analyst tool `higher_education_analytics`. Describe it as: “Query student engagement review using the documented mart grain, counts and rules.”
5. Use your verified lab default warehouse, or select the instructor-provided custom warehouse. Set query timeout to `60` seconds and add the tool.
6. Under **Instructions**, keep model selection `auto`. Set orchestration instructions to:

```text
Use higher_education_analytics for every quantitative answer. Query the current mart instead of relying on conversation memory. Sum stored counts; COUNT(*) counts groups, not observations. Compute review share as SUM(review_count)/NULLIF(SUM(measured_count),0), never AVG of group shares. Reconstruct the observation mean as SUM(mean_metric_value*measured_count)/NULLIF(SUM(measured_count),0), never AVG of group means. This recovers an unweighted mean across measured observations; it does not introduce business weights. Null denominators mean unknown, not zero. Flag an observation when record-level engagement score < 40. Null metrics remain unknown and are excluded from measured_count. Count observations, not distinct students; the illustrative threshold is not an academic decision. The summary has no observation dates or row-level details, so it cannot support trends or individual-record explanations.
```

7. Set response instructions to:

```text
Give concise answers grounded in successful tool results. Explain the grain, denominator and missing values when relevant. Identify these as lab observations, not live predictions. State when a request needs columns or history absent from the semantic view. Do not invent a result when a query fails.
```

8. If available, set a `120` second time limit and `8000` token limit for these small questions. Follow the instructor's account limits if they differ. Save the agent as a **draft** and open **Preview**. Agent publication, sharing and CoWork integration are outside this exercise.

## 5. Ask questions and verify answers

1. Ask the first question:

```text
How many observations meet the student engagement review rule? Show total observations, measured observations, missing metrics and review share.
```

2. Open **Show Traces → SQL Execution** (or the equivalent execution detail). Inspect the SQL, results and status. Confirm a successful Analyst query uses your semantic view/mart and the documented grain and denominators. A plausible answer alone is insufficient.
3. Ask the second question:

```text
Which three major_code groups have the most review observations? Include group keys, measured and missing counts, and review shares. Break ties by group key.
```

4. The instructor runs these two queries with your development database/schema and lab warehouse, then shares the results for comparison. They must use the same restored dbt output as your agent. Expected counts vary by synced data; compare actual values, not a hard-coded answer.

```sql
SELECT SUM(observation_count) AS observations, SUM(measured_count) AS measured, SUM(missing_metric_count) AS missing, SUM(review_count) AS flagged, SUM(review_count) / NULLIF(SUM(measured_count), 0) AS review_share FROM <DEV_DATABASE>.<DBT_SCHEMA>.MART_HIGHER_EDUCATION_REVIEW_SUMMARY;
```

```sql
SELECT group_key, group_name, observation_count, measured_count, missing_metric_count, review_count, review_count / NULLIF(measured_count, 0) AS review_share FROM <DEV_DATABASE>.<DBT_SCHEMA>.MART_HIGHER_EDUCATION_REVIEW_SUMMARY ORDER BY review_count DESC, group_key LIMIT 3;
```

5. Compare totals, missing-value treatment, ordering and any fractions with the agent's trace and final answer. Empty output or a null rate can be valid; explain it rather than substituting zero. For decimal results, allow display rounding but verify the denominator.
6. Ask a question outside the mart's scope:

```text
How will these metrics change next month, and which individual records explain the change?
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
