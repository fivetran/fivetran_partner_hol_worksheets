# dbt Semantic Layer and MCP add-on: Agriculture — BigQuery

*November 2026. This worksheet continues the Agriculture dbt Platform add-on after you have written your marts and before you build them. It adds governed metrics with the dbt Semantic Layer, publishes them through your Production environment, and connects Claude Code on your lab workstation to the dbt remote MCP server.*

## Objective

Turn your **Coffee price monitoring** marts into governed metrics that an AI assistant can query without writing its own SQL. You will add a semantic model and metrics on top of your fact, validate them in your development environment, publish them with a Production run, and then ask Claude Code business questions through the dbt MCP server.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `coffee_prices` into your own destination.
- Your instructor launched the lab with an **individual dbt project** for each attendee and **dbt Semantic Layer + remote MCP** turned on. LabStudios has already created, for you only:
  - a dbt Platform project with a **Development** environment on the lab's BigQuery connection;
  - a **Production** environment whose deployment credential writes to your own prod dataset. It runs the `lab-work` branch (GitHub template labs) or `main` (dbt-managed repository labs);
  - a Semantic Layer configuration on that Production environment;
  - a lab **workstation** with Claude Code installed and configured.
- Complete **Parts 1–3** of either Agriculture edition, [manual](../../manual/agriculture/bigquery_manual_dbt_platform_hands_on_lab.md) or [dbt Wizard](../../wizard/agriculture/bigquery_wizard_dbt_platform_hands_on_lab.md), on the **`lab-work`** branch. Your Production environment runs `lab-work`, so your work must be there. (dbt-managed repository labs: see the variant at the end.)
- Stop after Part 3. You should have these files on `lab-work`, not yet built:
  - `models/marts/fct_agriculture_observations.sql`
  - `models/marts/mart_agriculture_review_summary.sql`
  - `models/marts/_agriculture__addon_models.yml`
  - `tests/assert_agriculture_addon.sql`
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

Your fact has one grouping column (currency) and no quote date. Metrics are far more useful over time, so you add one more model that keeps your fact as written and joins back to staging for the quote date and the day's trading figures.

Create these three files. They are provided in full; read them before saving.

### `models/marts/sem_agriculture_quotes.sql`

```sql
{{ config(materialized='table') }}
-- Semantic Layer input: your tested fact plus the quote date and daily trading figures.
-- One row per coffee_prices.id, exactly like fct_agriculture_observations.
select
    f.observation_id                                   as quote_id,
    safe_cast(left(cast(s.date as string), 10) as date) as quote_date,
    f.group_name                                       as currency,
    f.metric_value                                     as close_price,
    safe_cast(cast(s.high as string) as float64)       as high_price,
    safe_cast(cast(s.low as string) as float64)        as low_price,
    safe_cast(cast(s.volume as string) as float64)     as traded_volume,
    f.requires_review
from {{ ref('fct_agriculture_observations') }} f
join {{ ref('stg_agriculture__coffee_prices') }} s
    on s.id = f.observation_id
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

### `models/marts/_agriculture__semantic.yml`

```yaml
version: 2

models:
  - name: sem_agriculture_quotes
    description: "One row per coffee price quote. Semantic Layer input built from fct_agriculture_observations."
    columns:
      - name: quote_id
        data_tests: [unique, not_null]
      - name: quote_date
        description: "Trading date of the quote. Rows without a parseable date are excluded from time-based metrics."
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
  - name: coffee_quotes
    description: "Daily coffee futures quotes synced by Fivetran. Prices are closing quotes in the source currency, not revenue."
    model: ref('sem_agriculture_quotes')
    defaults:
      agg_time_dimension: quote_date
    entities:
      - name: quote
        type: primary
        expr: quote_id
    dimensions:
      - name: quote_date
        type: time
        type_params:
          time_granularity: day
      - name: currency
        type: categorical
        description: "Quote currency. 'Unknown' when the source left it empty."
      - name: requires_review
        type: categorical
        description: "True when the closing price is at or above the review threshold (150)."
    measures:
      - name: quote_count
        agg: sum
        expr: "1"
      - name: priced_quote_count
        agg: count
        expr: close_price
      - name: close_price_avg
        agg: average
        expr: close_price
      - name: close_price_max
        agg: max
        expr: close_price
      - name: close_price_min
        agg: min
        expr: close_price
      - name: daily_range_avg
        agg: average
        expr: high_price - low_price
      - name: traded_volume_sum
        agg: sum
        expr: traded_volume
      - name: review_quote_count
        agg: sum
        expr: "case when requires_review then 1 else 0 end"

metrics:
  - name: quotes
    label: "Quotes"
    description: "Number of daily quotes, priced or not."
    type: simple
    type_params:
      measure: quote_count
  - name: priced_quotes
    label: "Priced quotes"
    description: "Quotes that have a closing price. Denominator for review share."
    type: simple
    type_params:
      measure: priced_quote_count
  - name: average_close_price
    label: "Average closing price"
    description: "Unweighted average of daily closing prices in the source currency. Not a volume-weighted price."
    type: simple
    type_params:
      measure: close_price_avg
  - name: highest_close_price
    label: "Highest closing price"
    description: "Highest daily closing price in the period."
    type: simple
    type_params:
      measure: close_price_max
  - name: lowest_close_price
    label: "Lowest closing price"
    description: "Lowest daily closing price in the period."
    type: simple
    type_params:
      measure: close_price_min
  - name: average_daily_range
    label: "Average daily trading range"
    description: "Average of (high - low) per trading day. A volatility indicator, not a forecast."
    type: simple
    type_params:
      measure: daily_range_avg
  - name: traded_volume
    label: "Traded volume"
    description: "Sum of daily traded volume as reported by the source."
    type: simple
    type_params:
      measure: traded_volume_sum
  - name: quotes_needing_review
    label: "Quotes needing review"
    description: "Quotes with a closing price at or above the review threshold (150). An illustrative monitoring flag, not a trading signal."
    type: simple
    type_params:
      measure: review_quote_count
  - name: review_share
    label: "Review share"
    description: "Quotes needing review divided by priced quotes. Quotes without a price are excluded from the denominator."
    type: ratio
    type_params:
      numerator: quotes_needing_review
      denominator: priced_quotes
  - name: quotes_needing_review_vs_prior_year
    label: "Quotes needing review, change vs prior year"
    description: "Quotes needing review minus the same metric one year earlier. Use with a monthly or yearly time grain."
    type: derived
    type_params:
      expr: current_period - prior_year
      metrics:
        - name: quotes_needing_review
          alias: current_period
        - name: quotes_needing_review
          offset_window: 1 year
          alias: prior_year

saved_queries:
  - name: monthly_price_review
    description: "Monthly average closing price, quotes needing review and review share by currency."
    query_params:
      metrics: [average_close_price, quotes_needing_review, review_share]
      group_by:
        - "TimeDimension('metric_time', 'month')"
        - "Dimension('quote__currency')"
```

Notes on the definitions:
- Dimensions are referenced through their entity: `quote__currency`, `quote__requires_review`. `metric_time` is the shared time axis, backed by `quote_date`.
- `review_share` is a **ratio** metric: dbt sums the numerator and denominator for each group first, then divides. That is why it is correct at any grain, unlike averaging a per-row share.
- `quotes_needing_review_vs_prior_year` is a **derived** metric. Its `offset_window` reads the same metric one year back through the time spine.

> **Checkpoint:** the three new files are saved next to your Part 3 files.

## Part 3: Build and validate in development

1. Check the resolved targets before building:

```bash
dbt ls --select +mart_agriculture_review_summary +sem_agriculture_quotes metricflow_time_spine --resource-type model --output json --output-keys name database schema
```

Every model must target your **BigQuery dbt Dataset** in the lab's project. Staging views use your dataset plus `_staging`.

2. Build your marts, the Semantic Layer input, the time spine and every test:

```bash
dbt build --select +mart_agriculture_review_summary +sem_agriculture_quotes metricflow_time_spine
```

Require a successful result. Fix failing tests in your Part 3 models rather than removing them. The `quote_date` test is a warning, not a failure: if it warns, some quotes lack a parseable date and are left out of time-based answers. Note the count and discuss it with your instructor.

3. Validate the semantic definitions:

```bash
dbt sl validate
dbt sl list metrics
```

Fix any errors. The usual cause is a measure or dimension `expr` that doesn't match a column in `sem_agriculture_quotes`.

4. Query a metric in development:

```bash
dbt sl query --metrics average_close_price,quotes_needing_review --group-by metric_time__year --order-by metric_time__year
```

Record the most recent year in the results. You will use it to check Claude Code's answers.

| Check | Your value |
|---|---|
| Latest year with quotes | |
| `average_close_price` in that year | |
| `quotes_needing_review` in that year | |

> **Checkpoint:** the build passes, `dbt sl validate` succeeds, and `dbt sl query` returns one row per year.

## Part 4: Publish to Production

The Semantic Layer serves metrics from your **Production** environment's latest successful run, not from Studio. Until a Production run succeeds, Claude Code sees no metrics.

1. **Commit and sync** on `lab-work`: `dbt_project.yml`, your four Part 3 files and the three Semantic Layer files. Exclude credentials and unrelated changes.
2. Open **Orchestration → Environments → Production**. Confirm it runs the custom branch `lab-work`. Do not change its connection or credential; LabStudios configured them for you.
3. From the Production environment, choose **Create job → Deploy job**:
   - **Name:** `semantic_layer_build_<firstname>_<lastname>`
   - **Commands:** replace the default with:

   ```bash
   dbt build --select +mart_agriculture_review_summary +sem_agriculture_quotes metricflow_time_spine
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
| 1 | *"What metrics, dimensions and saved queries are available in my dbt project? Explain each metric in one line."* | `list_metrics`, `get_dimensions`, `list_saved_queries` | It lists your 10 metrics using **your** descriptions, and finds `monthly_price_review`. |
| 2 | *"Show the average closing price, highest closing price and average daily trading range by month for the most recent 12 months in the data."* | `query_metrics` grouped by `metric_time__month` | Twelve monthly rows ending in your latest year from Part 3. The monthly values should be consistent with your yearly figure. |
| 3 | *"Which currencies are in the data, and what share of priced quotes needed review in each? Only include currencies with at least 50 priced quotes."* | `get_dimension_values` for `quote__currency`, then `query_metrics` with `review_share` and `priced_quotes` | It reports a share between 0 and 1 per currency and applies the 50-quote filter. Ask how it applied the filter. |
| 4 | *"How is review_share calculated? Show me the SQL the Semantic Layer runs for it, and which dbt models it depends on."* | `get_metrics_compiled_sql`, plus a lineage or model-details tool | The SQL reads `sem_agriculture_quotes` in your **prod dataset** and divides two sums. The lineage runs from the `coffee_prices` source through staging and your fact. |
| 5 | *"For the most recent year in the data, compare quotes needing review each month with the same month one year earlier. Then tell me whether the models behind this metric passed their tests in the last production run."* | `query_metrics` with `quotes_needing_review_vs_prior_year` by month, then a model-health tool | Monthly changes, positive or negative, and a statement that your Production run's tests passed. |

Then ask one question your metrics can't answer: *"What was our coffee revenue last year?"* There is no revenue metric, and prices are quotes, not sales. A good answer says so instead of inventing a number. Discuss with your instructor why a governed metric layer matters for AI answers.

Claude Code's exact wording and tool order vary between runs. What matters is that numbers come from `query_metrics` and match your Part 3 baseline at the same grain.

**Stretch (optional):** change the review rule. Edit `review_threshold` in `dbt_project.yml` from `150` to `200`, commit, rerun your Production job, and ask question 3 again. The shares change only after the Production run succeeds, because the Semantic Layer reads Production. Restore `150` afterwards and rerun the job.

> **Checkpoint:** Claude Code answered the five questions from your metrics, and declined or qualified the question it couldn't answer.

## What you did

You layered governed metrics onto your tested marts: a semantic model, simple, ratio and derived metrics, and a saved query. You validated them in development, published them through a Production run, and had an AI assistant answer business questions through the dbt MCP server from those definitions instead of ad-hoc SQL.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Studio stays **Disconnected** | Your personal BigQuery credentials aren't saved, or Studio started before they were | Save them (edition Part 1), then **⋯ → Restart Studio** |
| `dbt sl validate` says a column doesn't exist | A measure or dimension `expr` doesn't match `sem_agriculture_quotes` | Compare the YAML with the model's column names |
| Time-based questions return nothing | `quote_date` is null for those rows, or the time spine doesn't cover the dates | Check the `quote_date` warning count; ask the instructor about the date format |
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
