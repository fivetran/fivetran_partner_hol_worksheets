# dbt Semantic Layer and MCP add-on: Retail — BigQuery

*November 2026. This worksheet continues the Retail dbt Platform add-on after you have written your marts and before you build them. It adds governed metrics with the dbt Semantic Layer, publishes them through your Production environment, and connects Claude Code on your lab workstation to the dbt remote MCP server.*

## Objective

Turn your **Retail inventory priorities** marts into governed metrics that an AI assistant can query without writing its own SQL. You will add a semantic model and metrics on top of your fact, validate them in your development environment, publish them with a Production run, and then ask Claude Code business questions through the dbt MCP server.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `inv_weekly_data`, `inv_products` and `inv_stores` into your own destination.
- Your instructor launched the lab with an **individual dbt project** for each attendee and **dbt Semantic Layer + remote MCP** turned on. LabStudios has already created, for you only:
  - a dbt Platform project with a **Development** environment on the lab's BigQuery connection;
  - a **Production** environment whose deployment credential writes to your own prod dataset. It runs the `lab-work` branch (GitHub template labs) or `main` (dbt-managed repository labs);
  - a Semantic Layer configuration on that Production environment;
  - a lab **workstation** with Claude Code installed and configured.
- Complete **Parts 1–3** of either Retail edition, [manual](../../retail/bigquery_manual_dbt_platform_hands_on_lab.md) or [dbt Wizard](../../retail/bigquery_wizard_dbt_platform_hands_on_lab.md), on the **`lab-work`** branch. Your Production environment runs `lab-work`, so your work must be there. (dbt-managed repository labs: see the variant at the end.)
- Stop after Part 3. You should have these files on `lab-work`, not yet built:
  - `models/marts/fct_retail_inventory_weekly.sql`
  - `models/marts/mart_retail_inventory_priorities.sql`
  - `models/marts/_retail__addon_models.yml`
  - `tests/assert_retail_addon.sql`
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

Your fact already has a week date and product and store attributes. Your mart keeps only the latest week for each product and store, which is right for a priority list but loses history. The provided model keeps every week from your fact and applies the same replenishment test your mart uses (days of supply below restock lead time), so metrics can show how replenishment pressure moves over time.

Create these three files. They are provided in full; read them before saving.

### `models/marts/sem_retail_inventory_weeks.sql`

```sql
{{ config(materialized='table') }}
-- Semantic Layer input: every week from your tested fact, with the same replenishment test as your mart.
-- One row per inv_weekly_data row, exactly like fct_retail_inventory_weekly.
select
    f.row_id                                           as inventory_week_id,
    f.inventory_date,
    coalesce(f.category, 'Unknown')                    as category,
    coalesce(f.region, 'Unknown')                      as region,
    coalesce(f.store_name, 'Unknown')                  as store_name,
    f.is_anomaly,
    safe_cast(cast(f.current_stock_level as string) as float64) as stock_level,
    safe_cast(cast(f.weekly_sales_volume as string) as float64) as sales_volume,
    safe_cast(cast(f.days_of_supply as string) as float64) as days_of_supply,
    safe_cast(cast(f.days_of_supply as string) as float64) < safe_cast(cast(f.restock_lead_time_days as string) as float64) as below_lead_time
from {{ ref('fct_retail_inventory_weekly') }} f
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

### `models/marts/_retail__semantic.yml`

```yaml
version: 2

models:
  - name: sem_retail_inventory_weeks
    description: "One row per product, store and week. Semantic Layer input built from fct_retail_inventory_weekly."
    columns:
      - name: inventory_week_id
        data_tests: [unique, not_null]
      - name: inventory_date
        description: "Week of the inventory observation. Rows without a parseable date are excluded from time-based metrics."
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
  - name: inventory_weeks
    description: "Weekly product/store inventory observations synced by Fivetran. Historical inventory, not a demand forecast."
    model: ref('sem_retail_inventory_weeks')
    defaults:
      agg_time_dimension: inventory_date
    entities:
      - name: inventory_week
        type: primary
        expr: inventory_week_id
    dimensions:
      - name: inventory_date
        type: time
        type_params:
          time_granularity: day
      - name: region
        type: categorical
        description: "Store region."
      - name: category
        type: categorical
        description: "Product category."
      - name: store_name
        type: categorical
        description: "Store name."
      - name: is_anomaly
        type: categorical
        description: "Anomaly flag supplied by the source."
      - name: below_lead_time
        type: categorical
        description: "True when days of supply is below restock lead time plus the review threshold (0 days)."
    measures:
      - name: week_count
        agg: sum
        expr: "1"
      - name: supply_count
        agg: count
        expr: days_of_supply
      - name: stock_level_avg
        agg: average
        expr: stock_level
      - name: sales_volume_sum
        agg: sum
        expr: sales_volume
      - name: days_of_supply_avg
        agg: average
        expr: days_of_supply
      - name: below_lead_time_count
        agg: sum
        expr: "case when below_lead_time then 1 else 0 end"
      - name: anomaly_count
        agg: sum
        expr: "case when is_anomaly then 1 else 0 end"

metrics:
  - name: inventory_weeks
    label: "Inventory weeks"
    description: "Number of product/store/week observations."
    type: simple
    type_params:
      measure: week_count
  - name: weeks_with_supply
    label: "Weeks with days of supply"
    description: "Observations with days of supply. Denominator for below-lead-time share."
    type: simple
    type_params:
      measure: supply_count
  - name: average_stock_level
    label: "Average stock level"
    description: "Average current stock level per observation."
    type: simple
    type_params:
      measure: stock_level_avg
  - name: total_sales_volume
    label: "Total sales volume"
    description: "Sum of weekly sales volume in units."
    type: simple
    type_params:
      measure: sales_volume_sum
  - name: average_days_of_supply
    label: "Average days of supply"
    description: "Unweighted average days of supply."
    type: simple
    type_params:
      measure: days_of_supply_avg
  - name: weeks_below_lead_time
    label: "Weeks below lead time"
    description: "Observations where days of supply is below restock lead time. The same test as needs_replenishment in your mart."
    type: simple
    type_params:
      measure: below_lead_time_count
  - name: below_lead_time_share
    label: "Below-lead-time share"
    description: "Weeks below lead time divided by weeks with days of supply."
    type: ratio
    type_params:
      numerator: weeks_below_lead_time
      denominator: weeks_with_supply
  - name: anomaly_weeks
    label: "Anomaly weeks"
    description: "Observations the source flagged as anomalies."
    type: simple
    type_params:
      measure: anomaly_count
  - name: weeks_below_lead_time_vs_prior_year
    label: "Weeks below lead time, change vs prior year"
    description: "Weeks below lead time minus the same metric one year earlier. Use with a monthly or yearly time grain."
    type: derived
    type_params:
      expr: current_period - prior_year
      metrics:
        - name: weeks_below_lead_time
          alias: current_period
        - name: weeks_below_lead_time
          offset_window: 1 year
          alias: prior_year

saved_queries:
  - name: monthly_replenishment_review
    description: "Monthly total sales volume, weeks below lead time and below-lead-time share by region."
    query_params:
      metrics: [total_sales_volume, weeks_below_lead_time, below_lead_time_share]
      group_by:
        - "TimeDimension('metric_time', 'month')"
        - "Dimension('inventory_week__region')"
```

Notes on the definitions:
- Dimensions are referenced through their entity: `inventory_week__region`, `inventory_week__category`. `metric_time` is the shared time axis, backed by `inventory_date`.
- `below_lead_time_share` is a **ratio** metric: dbt sums the numerator and denominator for each group first, then divides. That is why it is correct at any grain, unlike averaging a per-row share.
- `weeks_below_lead_time_vs_prior_year` is a **derived** metric. Its `offset_window` reads the same metric one year back through the time spine.

> **Checkpoint:** the three new files are saved next to your Part 3 files.

## Part 3: Build and validate in development

1. Check the resolved targets before building:

```bash
dbt ls --select +mart_retail_inventory_priorities +sem_retail_inventory_weeks metricflow_time_spine --resource-type model --output json --output-keys name database schema
```

Every model must target your **BigQuery dbt Dataset** in the lab's project. Staging views use your dataset plus `_staging`.

2. Build your marts, the Semantic Layer input, the time spine and every test:

```bash
dbt build --select +mart_retail_inventory_priorities +sem_retail_inventory_weeks metricflow_time_spine
```

Require a successful result. Fix failing tests in your Part 3 models rather than removing them. The `inventory_date` test is a warning, not a failure: if it warns, some inventory weeks lack a parseable date and are left out of time-based answers. Note the count and discuss it with your instructor.

3. Validate the semantic definitions:

```bash
dbt sl validate
dbt sl list metrics
```

Fix any errors. The usual cause is a measure or dimension `expr` that doesn't match a column in `sem_retail_inventory_weeks`.

4. Query a metric in development:

```bash
dbt sl query --metrics total_sales_volume,weeks_below_lead_time --group-by metric_time__year --order-by metric_time__year
```

Record the most recent year in the results. You will use it to check Claude Code's answers.

| Check | Your value |
|---|---|
| Latest year with inventory weeks | |
| `total_sales_volume` in that year | |
| `weeks_below_lead_time` in that year | |

> **Checkpoint:** the build passes, `dbt sl validate` succeeds, and `dbt sl query` returns one row per year.

## Part 4: Publish to Production

The Semantic Layer serves metrics from your **Production** environment's latest successful run, not from Studio. Until a Production run succeeds, Claude Code sees no metrics.

1. **Commit and sync** on `lab-work`: `dbt_project.yml`, your four Part 3 files and the three Semantic Layer files. Exclude credentials and unrelated changes.
2. Open **Orchestration → Environments → Production**. Confirm it runs the custom branch `lab-work`. Do not change its connection or credential; LabStudios configured them for you.
3. From the Production environment, choose **Create job → Deploy job**:
   - **Name:** `semantic_layer_build_<firstname>_<lastname>`
   - **Commands:** replace the default with:

   ```bash
   dbt build --select +mart_retail_inventory_priorities +sem_retail_inventory_weeks metricflow_time_spine
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
| 1 | *"What metrics, dimensions and saved queries are available in my dbt project? Explain each metric in one line."* | `list_metrics`, `get_dimensions`, `list_saved_queries` | It lists your 9 metrics using **your** descriptions, and finds `monthly_replenishment_review`. |
| 2 | *"Show total sales volume, average stock level and average days of supply by month for the most recent 12 months in the data."* | `query_metrics` grouped by `metric_time__month` | Twelve monthly rows ending in your latest year from Part 3. |
| 3 | *"Which regions and categories are in the data? For each region and category, show the below-lead-time share and anomaly weeks, only where there are at least 50 weeks with days of supply."* | `get_dimension_values` for `inventory_week__region`, then `query_metrics` with `below_lead_time_share`, `anomaly_weeks` and `weeks_with_supply` | A share between 0 and 1 per combination, with the 50-week filter applied. Ask how it applied the filter. |
| 4 | *"How is below_lead_time_share calculated? Show me the SQL the Semantic Layer runs for it, and which dbt models it depends on."* | `get_metrics_compiled_sql`, plus a lineage or model-details tool | The SQL reads `sem_retail_inventory_weeks` in your **prod dataset** and divides two sums. The lineage runs from the `inv_weekly_data` source through staging and your fact. |
| 5 | *"For the most recent year in the data, compare weeks below lead time each month with the same month one year earlier. Then tell me whether the models behind this metric passed their tests in the last production run."* | `query_metrics` with `weeks_below_lead_time_vs_prior_year` by month, then a model-health tool | Monthly changes, positive or negative, and a statement that your Production run's tests passed. |

Then ask one question your metrics can't answer: *"How many units of each product will we sell next month?"* There is no forecast metric; this is historical inventory. A good answer says so instead of inventing a number. Discuss with your instructor why a governed metric layer matters for AI answers.

Claude Code's exact wording and tool order vary between runs. What matters is that numbers come from `query_metrics` and match your Part 3 baseline at the same grain.

**Stretch (optional):** change the review rule. Edit `review_threshold` in `dbt_project.yml` from `0` to `3`, commit, rerun your Production job, and ask question 3 again. The shares change only after the Production run succeeds, because the Semantic Layer reads Production. Restore `0` afterwards and rerun the job.

> **Checkpoint:** Claude Code answered the five questions from your metrics, and declined or qualified the question it couldn't answer.

## What you did

You layered governed metrics onto your tested marts: a semantic model, simple, ratio and derived metrics, and a saved query. You validated them in development, published them through a Production run, and had an AI assistant answer business questions through the dbt MCP server from those definitions instead of ad-hoc SQL.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Studio stays **Disconnected** | Your personal BigQuery credentials aren't saved, or Studio started before they were | Save them (edition Part 1), then **⋯ → Restart Studio** |
| `dbt sl validate` says a column doesn't exist | A measure or dimension `expr` doesn't match `sem_retail_inventory_weeks` | Compare the YAML with the model's column names |
| Time-based questions return nothing | `inventory_date` is null for those rows, or the time spine doesn't cover the dates | Check the `inventory_date` warning count; ask the instructor about the date format |
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
