# dbt Semantic Layer and MCP add-on: Consumer Packaged Goods — BigQuery

*November 2026. This worksheet continues the Consumer Packaged Goods dbt Platform add-on after you have written your marts and before you build them. It adds governed metrics with the dbt Semantic Layer, publishes them through your Production environment, and connects Claude Code on your lab workstation to the dbt remote MCP server.*

## Objective

Turn your **Stockout review** marts into governed metrics that an AI assistant can query without writing its own SQL. You will add a semantic model and metrics on top of your fact, validate them in your development environment, publish them with a Production run, and then ask Claude Code business questions through the dbt MCP server.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `cpg_records` into your own destination.
- Your instructor launched the lab with an **individual dbt project** for each attendee and **dbt Semantic Layer + remote MCP** turned on. LabStudios has already created, for you only:
  - a dbt Platform project with a **Development** environment on the lab's BigQuery connection;
  - a **Production** environment whose deployment credential writes to your own prod dataset. It runs the `lab-work` branch (GitHub template labs) or `main` (dbt-managed repository labs);
  - a Semantic Layer configuration on that Production environment;
  - a lab **workstation** with Claude Code installed and configured.
- Complete **Parts 1–3** of either Consumer Packaged Goods edition, [manual](../../manual/consumer_packaged_goods/bigquery_manual_dbt_platform_hands_on_lab.md) or [dbt Wizard](../../wizard/consumer_packaged_goods/bigquery_wizard_dbt_platform_hands_on_lab.md), on the **`lab-work`** branch. Your Production environment runs `lab-work`, so your work must be there. (dbt-managed repository labs: see the variant at the end.)
- Stop after Part 3. You should have these files on `lab-work`, not yet built:
  - `models/marts/fct_consumer_packaged_goods_observations.sql`
  - `models/marts/mart_consumer_packaged_goods_review_summary.sql`
  - `models/marts/_consumer_packaged_goods__addon_models.yml`
  - `tests/assert_consumer_packaged_goods_addon.sql`
- Open your **1Password lab item** from your credential link. You will use these fields:

| 1Password field | Used in |
|---|---|
| BigQuery dbt Dataset, BigQuery dbt Prod Dataset | Parts 1 and 4: confirm your targets |
| dbt Platform Project URL | Part 1: open your project |
| Gateway URL, Guacamole Username, Guacamole Password | Part 6: sign in to your workstation |
| dbt MCP Command | Part 6: connect Claude Code |

- Never paste a password, private key or token into SQL, YAML, Git or a Claude prompt. The only secret you type in this lab is your own dbt personal access token, into the `claude mcp add` command on your own workstation.

## Part 1: Confirm your branch and dev target

1. Open your **dbt Platform Project URL** and open **Studio**. Wait for the status to show **Connected**. If it stays **Disconnected**, check that you saved your personal BigQuery credentials (Part 1 of the edition worksheet), then open the **⋯** menu, choose **Restart Studio** and wait again.
2. Confirm the branch selector shows **`lab-work`**. If not, choose **Change branch → `lab-work`**. (`main` is read-only in Studio.)
3. Confirm your dev target in the Studio command bar:

```bash
dbt show --inline "select '{{ target.database }}' as target_database, '{{ target.schema }}' as target_schema"
```

`target_database` must be the lab's BigQuery project (`fivetran-hands-on-lab`), and `target_schema` your **BigQuery dbt Dataset**, such as `dbt_bqp042`. If either is wrong, fix your credentials before continuing (Part 1 of the edition worksheet).

> **Checkpoint:** Studio is **Connected**, you are on `lab-work`, your four Part 3 files are present, and `dbt show` returns your own dev target.

## Part 2: Add the Semantic Layer files

A **semantic model** tells dbt what one row means, which columns people can group and filter by, and which numbers can be added up. **Metrics** are the named business numbers people ask for. Claude Code can only answer with the metrics you define here, and it reads each metric's description to decide which one fits a question.

Your fact has one grouping column (product category) and no order date. The provided model keeps your fact as written and joins back to staging for the order date, subcategory, customer segment and order status, plus the overstock rate and order total, so stockouts can be trended and sliced the way a category manager would.

Create these three files. They are provided in full; read them before saving.

### `models/marts/sem_consumer_packaged_goods_order_lines.sql`

```sql
{{ config(materialized='table') }}
-- Semantic Layer input: your tested fact plus order_date and extra attributes from staging.
-- One row per cpg_records row, exactly like fct_consumer_packaged_goods_observations.
select
    f.observation_id                                   as order_line_id,
    safe_cast(left(cast(s.order_date as string), 10) as date) as order_date,
    f.group_name                                       as product_category,
    coalesce(cast(s.product_subcategory as string), 'Unknown') as product_subcategory,
    coalesce(cast(s.customer_segment as string), 'Unknown') as customer_segment,
    coalesce(cast(s.order_status as string), 'Unknown') as order_status,
    f.metric_value                                     as stockout_rate,
    safe_cast(cast(s.overstock_rate as string) as float64) as overstock_rate,
    safe_cast(cast(s.order_total as string) as float64) as order_total,
    f.requires_review
from {{ ref('fct_consumer_packaged_goods_observations') }} f
join {{ ref('stg_consumer_packaged_goods__cpg_records') }} s
    on s.record_id = f.observation_id
```

### `models/marts/metricflow_time_spine.sql`

The Semantic Layer needs a calendar table to roll metrics up by day, month or year and to compare a period with the one before it.

```sql
{{ config(materialized='table') }}
{{ dbt.date_spine(
    datepart="day",
    start_date="cast('1990-01-01' as date)",
    end_date="cast('2036-01-01' as date)"
) }}
```

### `models/marts/_consumer_packaged_goods__semantic.yml`

```yaml
version: 2

models:
  - name: sem_consumer_packaged_goods_order_lines
    description: "One row per order/product record. Semantic Layer input built from fct_consumer_packaged_goods_observations."
    columns:
      - name: order_line_id
        data_tests: [unique, not_null]
      - name: order_date
        description: "Date of the order line. Rows without a parseable date are excluded from time-based metrics."
        data_tests:
          - not_null:
              config:
                severity: warn
  - name: metricflow_time_spine
    description: "One row per calendar day, used by the Semantic Layer."
    time_spine:
      standard_granularity_column: date_day
    columns:
      - name: date_day
        granularity: day

semantic_models:
  - name: order_lines
    description: "Order/product records synced by Fivetran. Records are observations, not distinct products or orders."
    model: ref('sem_consumer_packaged_goods_order_lines')
    defaults:
      agg_time_dimension: order_date
    entities:
      - name: order_line
        type: primary
        expr: order_line_id
    dimensions:
      - name: order_date
        type: time
        type_params:
          time_granularity: day
      - name: product_category
        type: categorical
        description: "Product category."
      - name: customer_segment
        type: categorical
        description: "Customer segment on the order."
      - name: product_subcategory
        type: categorical
        description: "Product subcategory."
      - name: order_status
        type: categorical
        description: "Order status."
      - name: requires_review
        type: categorical
        description: "True when the stockout rate is at or above the review threshold (0.1)."
    measures:
      - name: record_count
        agg: sum
        expr: "1"
      - name: rated_count
        agg: count
        expr: stockout_rate
      - name: stockout_rate_avg
        agg: average
        expr: stockout_rate
      - name: stockout_rate_max
        agg: max
        expr: stockout_rate
      - name: overstock_rate_avg
        agg: average
        expr: overstock_rate
      - name: order_total_avg
        agg: average
        expr: order_total
      - name: high_stockout_count
        agg: sum
        expr: "case when requires_review then 1 else 0 end"

metrics:
  - name: order_records
    label: "Order records"
    description: "Number of order/product records. Not distinct orders or products."
    type: simple
    type_params:
      measure: record_count
  - name: records_with_stockout_rate
    label: "Records with a stockout rate"
    description: "Records that have a stockout rate. Denominator for high-stockout share."
    type: simple
    type_params:
      measure: rated_count
  - name: average_stockout_rate
    label: "Average stockout rate"
    description: "Unweighted average of record stockout rates (0-1)."
    type: simple
    type_params:
      measure: stockout_rate_avg
  - name: highest_stockout_rate
    label: "Highest stockout rate"
    description: "Highest stockout rate in the period."
    type: simple
    type_params:
      measure: stockout_rate_max
  - name: average_overstock_rate
    label: "Average overstock rate"
    description: "Unweighted average of record overstock rates (0-1)."
    type: simple
    type_params:
      measure: overstock_rate_avg
  - name: average_order_total
    label: "Average order total"
    description: "Average order total per record. An order can appear on several records, so this is not revenue."
    type: simple
    type_params:
      measure: order_total_avg
  - name: high_stockout_records
    label: "High-stockout records"
    description: "Records with a stockout rate at or above the review threshold (0.1). An illustrative flag."
    type: simple
    type_params:
      measure: high_stockout_count
  - name: high_stockout_share
    label: "High-stockout share"
    description: "High-stockout records divided by records with a stockout rate."
    type: ratio
    type_params:
      numerator: high_stockout_records
      denominator: records_with_stockout_rate
  - name: high_stockout_records_vs_prior_year
    label: "High-stockout records, change vs prior year"
    description: "High-stockout records minus the same metric one year earlier. Use with a monthly or yearly time grain."
    type: derived
    type_params:
      expr: current_period - prior_year
      metrics:
        - name: high_stockout_records
          alias: current_period
        - name: high_stockout_records
          offset_window: 1 year
          alias: prior_year

saved_queries:
  - name: monthly_stockout_review
    description: "Monthly average stockout rate, high-stockout records and high-stockout share by product category."
    query_params:
      metrics: [average_stockout_rate, high_stockout_records, high_stockout_share]
      group_by:
        - "TimeDimension('metric_time', 'month')"
        - "Dimension('order_line__product_category')"
```

Notes on the definitions:
- Dimensions are referenced through their entity: `order_line__product_category`, `order_line__customer_segment`. `metric_time` is the shared time axis, backed by `order_date`.
- `high_stockout_share` is a **ratio** metric: dbt sums the numerator and denominator for each group first, then divides. That is why it is correct at any grain, unlike averaging a per-row share.
- `high_stockout_records_vs_prior_year` is a **derived** metric. Its `offset_window` reads the same metric one year back through the time spine.

> **Checkpoint:** the three new files are saved next to your Part 3 files.

## Part 3: Build and validate in development

1. Check the resolved targets before building:

```bash
dbt ls --select +mart_consumer_packaged_goods_review_summary +sem_consumer_packaged_goods_order_lines metricflow_time_spine --resource-type model --output json --output-keys name database schema
```

Every model must target your **BigQuery dbt Dataset** in the lab's project. Staging views use your dataset plus `_staging`.

2. Build your marts, the Semantic Layer input, the time spine and every test:

```bash
dbt build --select +mart_consumer_packaged_goods_review_summary +sem_consumer_packaged_goods_order_lines metricflow_time_spine
```

Require a successful result. Fix failing tests in your Part 3 models rather than removing them. The `order_date` test is a warning, not a failure: if it warns, some order lines lack a parseable date and are left out of time-based answers. Note the count and discuss it with your instructor.

3. Validate the semantic definitions:

```bash
dbt sl validate
dbt sl list metrics
```

Fix any errors. The usual cause is a measure or dimension `expr` that doesn't match a column in `sem_consumer_packaged_goods_order_lines`.

4. Query a metric in development:

```bash
dbt sl query --metrics average_stockout_rate,high_stockout_records --group-by metric_time__year --order-by metric_time__year
```

Record the most recent year in the results. You will use it to check Claude Code's answers.

| Check | Your value |
|---|---|
| Latest year with order lines | |
| `average_stockout_rate` in that year | |
| `high_stockout_records` in that year | |

> **Checkpoint:** the build passes, `dbt sl validate` succeeds, and `dbt sl query` returns one row per year.

## Part 4: Publish to Production

The Semantic Layer serves metrics from your **Production** environment's latest successful run, not from Studio. Until a Production run succeeds, Claude Code sees no metrics.

1. **Commit and sync** on `lab-work`: `dbt_project.yml`, your four Part 3 files and the three Semantic Layer files. Exclude credentials and unrelated changes.
2. Open **Orchestration → Environments → Production**. Confirm it runs the custom branch `lab-work`. Do not change its connection or credential; LabStudios configured them for you.
3. From the Production environment, choose **Create job → Deploy job**:
   - **Name:** `semantic_layer_build_<firstname>_<lastname>`
   - **Commands:** replace the default with:

   ```bash
   dbt build --select +mart_consumer_packaged_goods_review_summary +sem_consumer_packaged_goods_order_lines metricflow_time_spine
   ```

   - Turn off schedules and job-completion triggers. Save.
4. Click **Run now** and wait for **Success**. Open the run and confirm:
   - the commit is your latest on `lab-work`;
   - the models were built in your **BigQuery dbt Prod Dataset** (your dev dataset plus `_prod`, such as `dbt_bqp042_prod`);
   - the tests ran and passed.
5. If the run fails, read the failing step. A failure in **Clone** means the commit isn't on `lab-work`; a profile or credential error goes to your instructor.

> **Checkpoint:** a Production run succeeded from your latest `lab-work` commit. Your metrics are published.

## Part 5: Create your token and find your user ID

1. **Create a personal access token:** **Account settings → API tokens → Personal tokens → Create personal access token**. Name it `lab-mcp`. Copy it now; dbt shows it only once. Treat it like a password: don't paste it anywhere except the command in Part 6.
2. **Find your dbt user ID:** **Account settings → Users**, then select your own name. The number at the end of the page URL is your user ID.

Use a personal access token, not a **Semantic Layer credential**. The Semantic Layer configuration page offers to create credentials and service tokens; you don't need either for this lab. On BigQuery the Semantic Layer runs queries with your Production environment's credential, so you don't upload a service account key either.

> **Checkpoint:** you have a token (kept private) and your numeric user ID.

## Part 6: Connect Claude Code to your metrics

1. Open the **Gateway URL** from your 1Password item and sign in with your **Guacamole Username** and **Guacamole Password**. Your Linux desktop opens in the browser.
2. Open **Terminal**.
3. Copy the **dbt MCP Command** from your 1Password item. It already contains the lab's dbt host and both of your environment IDs. Paste it into the terminal, but don't press Enter yet. It looks like this:

```bash
claude mcp add --transport http dbt https://rg855.us1.dbt.com/api/ai/v1/mcp/ --header "Authorization: Token <YOUR_PERSONAL_ACCESS_TOKEN>" --header "x-dbt-prod-environment-id: <from your item>" --header "x-dbt-dev-environment-id: <from your item>" --header "x-dbt-user-id: <YOUR_DBT_USER_ID>" --header "x-dbt-disable-toolsets: sql"
```

4. Replace `<YOUR_PERSONAL_ACCESS_TOKEN>` and `<YOUR_DBT_USER_ID>` with your values. Keep every other part as it is.
   - The host must match the address bar of your dbt Platform project. If your instructor's lab uses a different dbt account, your item's command already has the right host.
   - `x-dbt-disable-toolsets: sql` turns off the MCP server's SQL tools, so every number Claude gives you comes from a governed metric rather than SQL it wrote itself. If the command in your item doesn't end with it, add it.
5. Press **Enter**. Start Claude Code with `claude`, then type `/mcp`. Confirm **dbt** shows as connected. Claude Code is already set up for this lab; you don't need an Anthropic account or API key.

> **Checkpoint:** `/mcp` lists **dbt** as connected.

## Part 7: Ask questions about your metrics

Ask each question in plain language. Claude Code shows each tool call before it runs; read them. You should see it discover your metrics, then query them through the Semantic Layer, instead of guessing table names. Approve tool calls when asked.

| # | Ask Claude Code | Expect it to use | What to check |
|---|---|---|---|
| 1 | *"What metrics, dimensions and saved queries are available in my dbt project? Explain each metric in one line."* | `list_metrics`, `get_dimensions`, `list_saved_queries` | It lists your 9 metrics using **your** descriptions, and finds `monthly_stockout_review`. |
| 2 | *"Show the average stockout rate, average overstock rate and high-stockout records by month for the most recent 12 months in the data."* | `query_metrics` grouped by `metric_time__month` | Twelve monthly rows ending in your latest year from Part 3. Rates are between 0 and 1. |
| 3 | *"Which customer segments are in the data? For each product category and customer segment, show the high-stockout share, only where there are at least 30 records with a stockout rate."* | `get_dimension_values` for `order_line__customer_segment`, then `query_metrics` with `high_stockout_share` | A share between 0 and 1 per combination, with the 30-record filter applied. Ask how it applied the filter. |
| 4 | *"How is high_stockout_share calculated? Show me the SQL the Semantic Layer runs for it, and which dbt models it depends on."* | `get_metrics_compiled_sql`, plus a lineage or model-details tool | The SQL reads `sem_consumer_packaged_goods_order_lines` in your **prod dataset** and divides two sums. The lineage runs from the `cpg_records` source through staging and your fact. |
| 5 | *"For the most recent year in the data, compare high-stockout records each month with the same month one year earlier. Then tell me whether the models behind this metric passed their tests in the last production run."* | `query_metrics` with `high_stockout_records_vs_prior_year` by month, then a model-health tool | Monthly changes, positive or negative, and a statement that your Production run's tests passed. |

Then ask one question your metrics can't answer: *"Which products will stock out next month?"* There is no forecast metric; these are historical rates. A good answer says so instead of inventing a number. Discuss with your instructor why a governed metric layer matters for AI answers.

Claude Code's exact wording and tool order vary between runs. What matters is that numbers come from `query_metrics` and match your Part 3 baseline at the same grain.

**Stretch (optional):** change the review rule. Edit `review_threshold` in `dbt_project.yml` from `0.1` to `0.2`, commit, rerun your Production job, and ask question 3 again. The shares change only after the Production run succeeds, because the Semantic Layer reads Production. Restore `0.1` afterwards and rerun the job.

> **Checkpoint:** Claude Code answered the five questions from your metrics, and declined or qualified the question it couldn't answer.

## What you did

You layered governed metrics onto your tested marts: a semantic model, simple, ratio and derived metrics, and a saved query. You validated them in development, published them through a Production run, and had an AI assistant answer business questions through the dbt MCP server from those definitions instead of ad-hoc SQL.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Studio stays **Disconnected** | Your personal BigQuery credentials aren't saved, or Studio started before they were | Save them (edition Part 1), then **⋯ → Restart Studio** |
| `dbt sl validate` says a column doesn't exist | A measure or dimension `expr` doesn't match `sem_consumer_packaged_goods_order_lines` | Compare the YAML with the model's column names |
| Time-based questions return nothing | `order_date` is null for those rows, or the time spine doesn't cover the dates | Check the `order_date` warning count; ask the instructor about the date format |
| Production run fails at **Clone** | Your commit isn't on `lab-work` | Commit and sync on `lab-work`, then rerun |
| Production run fails with **Failed to generate profile** | Production credential problem | Ask the instructor; don't edit the credential |
| Claude Code sees no metrics | No successful Production run yet, or it ran an older commit | Commit on `lab-work`, rerun the Production job |
| `/mcp` shows dbt as failed | Token or user ID typo, or the token was deleted | `claude mcp remove dbt`, then run the add command again with correct values |
| Claude writes SQL instead of using metrics | The `x-dbt-disable-toolsets: sql` header is missing | `claude mcp remove dbt`, then run the add command again with the header |

## Variant: dbt-managed repository labs

If your lab uses a **dbt-managed repository** instead of GitHub, there is no `lab-work` branch and Production runs `main`. Use the branch your instructor gives you in Studio for Parts 1–3 of the edition worksheet and for Part 2 here. In Part 4, commit, then choose **Merge this branch to main** before you run the Production job. Everything else is the same.

## References

- [dbt Semantic Layer](https://docs.getdbt.com/docs/use-dbt-semantic-layer/dbt-sl)
- [Semantic models](https://docs.getdbt.com/docs/build/semantic-models), [metrics](https://docs.getdbt.com/docs/build/metrics-overview), [ratio metrics](https://docs.getdbt.com/docs/build/ratio) and [derived metrics](https://docs.getdbt.com/docs/build/derived)
- [Saved queries](https://docs.getdbt.com/docs/build/saved-queries)
- [MetricFlow time spine](https://docs.getdbt.com/docs/build/metricflow-time-spine)
- [dbt remote MCP server](https://docs.getdbt.com/docs/dbt-ai/setup-remote-mcp)
