# dbt Semantic Layer and MCP add-on: Transportation — BigQuery

*November 2026. This worksheet continues the Transportation dbt Platform add-on after you have written your marts and before you build them. It adds governed metrics with the dbt Semantic Layer, publishes them through your Production environment, and connects Claude Code on your lab workstation to the dbt remote MCP server.*

## Objective

Turn your **Arrival delay review** marts into governed metrics that an AI assistant can query without writing its own SQL. You will add a semantic model and metrics on top of your fact, validate them in your development environment, publish them with a Production run, and then ask Claude Code business questions through the dbt MCP server.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `airline_delays` into your own destination.
- Your instructor launched the lab with an **individual dbt project** for each attendee and **dbt Semantic Layer + remote MCP** turned on. LabStudios has already created, for you only:
  - a dbt Platform project with a **Development** environment on the lab's BigQuery connection;
  - a **Production** environment whose deployment credential writes to your own prod dataset. It runs the `lab-work` branch (GitHub template labs) or `main` (dbt-managed repository labs);
  - a Semantic Layer configuration on that Production environment;
  - a lab **workstation** with Claude Code installed and configured.
- Complete **Parts 1–3** of either Transportation edition, [manual](../../transportation/bigquery_manual_dbt_platform_hands_on_lab.md) or [dbt Wizard](../../transportation/bigquery_wizard_dbt_platform_hands_on_lab.md), on the **`lab-work`** branch. Your Production environment runs `lab-work`, so your work must be there. (dbt-managed repository labs: see the variant at the end.)
- Stop after Part 3. You should have these files on `lab-work`, not yet built:
  - `models/marts/fct_transportation_observations.sql`
  - `models/marts/mart_transportation_review_summary.sql`
  - `models/marts/_transportation__addon_models.yml`
  - `tests/assert_transportation_addon.sql`
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

Your fact computes average delay per flight for each carrier, airport and month, but the summary averages those averages, which weights a tiny airport the same as a hub. The provided model keeps your fact as written and joins back to staging for the month and the raw counts (arriving flights, delayed flights, delay minutes, cancellations). A **ratio** metric can then compute delay minutes per flight properly weighted, next to the unweighted average, so you can compare the two.

Create these three files. They are provided in full; read them before saving.

### `models/marts/sem_transportation_airport_months.sql`

```sql
{{ config(materialized='table') }}
-- Semantic Layer input: your tested fact plus service_month and extra attributes from staging.
-- One row per airline_delays row, exactly like fct_transportation_observations.
select
    f.observation_id                                   as airport_month_id,
    date(cast(safe_cast(cast(s.year as string) as float64) as int64), cast(safe_cast(cast(s.month as string) as float64) as int64), 1) as service_month,
    f.group_name                                       as carrier_name,
    coalesce(cast(s.airport as string), 'Unknown')     as airport,
    safe_cast(cast(s.arr_flights as string) as float64) as arriving_flights,
    safe_cast(cast(s.arr_del15 as string) as float64)  as delayed_flights,
    safe_cast(cast(s.arr_delay as string) as float64)  as delay_minutes,
    safe_cast(cast(s.arr_cancelled as string) as float64) as cancelled_flights,
    f.metric_value                                     as average_delay_per_flight,
    f.requires_review
from {{ ref('fct_transportation_observations') }} f
join {{ ref('stg_transportation__airline_delays') }} s
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

### `models/marts/_transportation__semantic.yml`

```yaml
version: 2

models:
  - name: sem_transportation_airport_months
    description: "One row per carrier, airport and month. Semantic Layer input built from fct_transportation_observations."
    columns:
      - name: airport_month_id
        data_tests: [unique, not_null]
      - name: service_month
        description: "First day of the reporting month (year and month). Rows without a parseable month are excluded from time-based metrics."
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
  - name: airport_months
    description: "Monthly airline on-time statistics by carrier and airport, synced by Fivetran. Each row is an airport-month, not a flight."
    model: ref('sem_transportation_airport_months')
    defaults:
      agg_time_dimension: service_month
    entities:
      - name: airport_month
        type: primary
        expr: airport_month_id
    dimensions:
      - name: service_month
        type: time
        type_params:
          time_granularity: month
      - name: carrier_name
        type: categorical
        description: "Airline."
      - name: airport
        type: categorical
        description: "Airport code."
      - name: requires_review
        type: categorical
        description: "True for high-delay airport-months: average delay per flight at or above the review threshold (10 minutes)."
    measures:
      - name: airport_month_count
        agg: sum
        expr: "1"
      - name: rated_count
        agg: count
        expr: average_delay_per_flight
      - name: delay_per_flight_avg
        agg: average
        expr: average_delay_per_flight
      - name: arriving_flights_sum
        agg: sum
        expr: arriving_flights
      - name: delayed_flights_sum
        agg: sum
        expr: delayed_flights
      - name: delay_minutes_sum
        agg: sum
        expr: delay_minutes
      - name: cancelled_flights_sum
        agg: sum
        expr: cancelled_flights
      - name: high_delay_count
        agg: sum
        expr: "case when requires_review then 1 else 0 end"

metrics:
  - name: airport_months
    label: "Airport-months"
    description: "Number of carrier/airport/month rows. Not flights."
    type: simple
    type_params:
      measure: airport_month_count
  - name: rated_airport_months
    label: "Rated airport-months"
    description: "Airport-months with an average delay. Denominator for high-delay share."
    type: simple
    type_params:
      measure: rated_count
  - name: arriving_flights
    label: "Arriving flights"
    description: "Sum of arriving flights."
    type: simple
    type_params:
      measure: arriving_flights_sum
  - name: delayed_flights
    label: "Delayed flights"
    description: "Sum of flights arriving 15 or more minutes late."
    type: simple
    type_params:
      measure: delayed_flights_sum
  - name: delay_minutes
    label: "Delay minutes"
    description: "Sum of arrival delay minutes."
    type: simple
    type_params:
      measure: delay_minutes_sum
  - name: cancelled_flights
    label: "Cancelled flights"
    description: "Sum of cancelled flights."
    type: simple
    type_params:
      measure: cancelled_flights_sum
  - name: delay_minutes_per_flight
    label: "Delay minutes per flight"
    description: "Delay minutes divided by arriving flights. Weighted by flights: the network-wide per-flight average."
    type: ratio
    type_params:
      numerator: delay_minutes
      denominator: arriving_flights
  - name: average_delay_per_flight_unweighted
    label: "Average delay per flight (unweighted)"
    description: "Average of each airport-month delay per flight. Every airport-month counts equally, so small airports weigh as much as hubs."
    type: simple
    type_params:
      measure: delay_per_flight_avg
  - name: high_delay_airport_months
    label: "High-delay airport-months"
    description: "Airport-months with an average delay per flight at or above the review threshold (10 minutes)."
    type: simple
    type_params:
      measure: high_delay_count
  - name: high_delay_share
    label: "High-delay share"
    description: "High-delay airport-months divided by rated airport-months."
    type: ratio
    type_params:
      numerator: high_delay_airport_months
      denominator: rated_airport_months
  - name: high_delay_airport_months_vs_prior_year
    label: "High-delay airport-months, change vs prior year"
    description: "High-delay airport-months minus the same metric one year earlier. Use with a monthly or yearly time grain."
    type: derived
    type_params:
      expr: current_period - prior_year
      metrics:
        - name: high_delay_airport_months
          alias: current_period
        - name: high_delay_airport_months
          offset_window: 1 year
          alias: prior_year

saved_queries:
  - name: monthly_delay_review
    description: "Monthly arriving flights, delay minutes per flight and high-delay share by carrier."
    query_params:
      metrics: [arriving_flights, delay_minutes_per_flight, high_delay_share]
      group_by:
        - "TimeDimension('metric_time', 'month')"
        - "Dimension('airport_month__carrier_name')"
```

Notes on the definitions:
- Dimensions are referenced through their entity: `airport_month__carrier_name`, `airport_month__airport`. `metric_time` is the shared time axis, backed by `service_month`.
- `delay_minutes_per_flight` is a **ratio** metric: dbt sums the numerator and denominator for each group first, then divides. That is why it is correct at any grain, unlike averaging a per-row share.
- `high_delay_airport_months_vs_prior_year` is a **derived** metric. Its `offset_window` reads the same metric one year back through the time spine.
- `service_month` has a **month** grain, so ask for months, quarters or years, not days.

> **Checkpoint:** the three new files are saved next to your Part 3 files.

## Part 3: Build and validate in development

1. Check the resolved targets before building:

```bash
dbt ls --select +mart_transportation_review_summary +sem_transportation_airport_months metricflow_time_spine --resource-type model --output json --output-keys name database schema
```

Every model must target your **BigQuery dbt Dataset** in the lab's project. Staging views use your dataset plus `_staging`.

2. Build your marts, the Semantic Layer input, the time spine and every test:

```bash
dbt build --select +mart_transportation_review_summary +sem_transportation_airport_months metricflow_time_spine
```

Require a successful result. Fix failing tests in your Part 3 models rather than removing them. The `service_month` test is a warning, not a failure: if it warns, some airport-months lack a parseable date and are left out of time-based answers. Note the count and discuss it with your instructor.

3. Validate the semantic definitions:

```bash
dbt sl validate
dbt sl list metrics
```

Fix any errors. The usual cause is a measure or dimension `expr` that doesn't match a column in `sem_transportation_airport_months`.

4. Query a metric in development:

```bash
dbt sl query --metrics delay_minutes_per_flight,high_delay_airport_months --group-by metric_time__year --order-by metric_time__year
```

Record the most recent year in the results. You will use it to check Claude Code's answers.

| Check | Your value |
|---|---|
| Latest year with airport-months | |
| `delay_minutes_per_flight` in that year | |
| `high_delay_airport_months` in that year | |

> **Checkpoint:** the build passes, `dbt sl validate` succeeds, and `dbt sl query` returns one row per year.

## Part 4: Publish to Production

The Semantic Layer serves metrics from your **Production** environment's latest successful run, not from Studio. Until a Production run succeeds, Claude Code sees no metrics.

1. **Commit and sync** on `lab-work`: `dbt_project.yml`, your four Part 3 files and the three Semantic Layer files. Exclude credentials and unrelated changes.
2. Open **Orchestration → Environments → Production**. Confirm it runs the custom branch `lab-work`. Do not change its connection or credential; LabStudios configured them for you.
3. From the Production environment, choose **Create job → Deploy job**:
   - **Name:** `semantic_layer_build_<firstname>_<lastname>`
   - **Commands:** replace the default with:

   ```bash
   dbt build --select +mart_transportation_review_summary +sem_transportation_airport_months metricflow_time_spine
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
| 1 | *"What metrics, dimensions and saved queries are available in my dbt project? Explain each metric in one line."* | `list_metrics`, `get_dimensions`, `list_saved_queries` | It lists your 11 metrics using **your** descriptions, and finds `monthly_delay_review`. |
| 2 | *"Show arriving flights, delayed flights and delay minutes per flight by month for the most recent 12 months in the data."* | `query_metrics` grouped by `metric_time__month` | Twelve monthly rows ending in your latest year from Part 3. |
| 3 | *"Which carriers are in the data? Compare each carrier's delay minutes per flight with its unweighted average delay per flight, only for carriers with at least 1,000 arriving flights. Why do the two numbers differ?"* | `get_dimension_values` for `airport_month__carrier_name`, then `query_metrics` with both metrics and `arriving_flights` | Two numbers per carrier, and an explanation from your descriptions: one is weighted by flights, the other treats every airport-month equally. |
| 4 | *"How is delay_minutes_per_flight calculated? Show me the SQL the Semantic Layer runs for it, and which dbt models it depends on."* | `get_metrics_compiled_sql`, plus a lineage or model-details tool | The SQL reads `sem_transportation_airport_months` in your **prod dataset** and divides two sums. The lineage runs from the `airline_delays` source through staging and your fact. |
| 5 | *"For the most recent year in the data, compare high-delay airport-months each month with the same month one year earlier. Then tell me whether the models behind this metric passed their tests in the last production run."* | `query_metrics` with `high_delay_airport_months_vs_prior_year` by month, then a model-health tool | Monthly changes, positive or negative, and a statement that your Production run's tests passed. |

Then ask one question your metrics can't answer: *"Which airline is the safest?"* There is no safety metric; the data covers on-time performance only. A good answer says so instead of inventing a number. Discuss with your instructor why a governed metric layer matters for AI answers.

Claude Code's exact wording and tool order vary between runs. What matters is that numbers come from `query_metrics` and match your Part 3 baseline at the same grain.

**Stretch (optional):** change the review rule. Edit `review_threshold` in `dbt_project.yml` from `10` to `15`, commit, rerun your Production job, and ask question 5 again. The high-delay counts change only after the Production run succeeds, because the Semantic Layer reads Production. Restore `10` afterwards and rerun the job.

> **Checkpoint:** Claude Code answered the five questions from your metrics, and declined or qualified the question it couldn't answer.

## What you did

You layered governed metrics onto your tested marts: a semantic model, simple, ratio and derived metrics, and a saved query. You validated them in development, published them through a Production run, and had an AI assistant answer business questions through the dbt MCP server from those definitions instead of ad-hoc SQL.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Studio stays **Disconnected** | Your personal BigQuery credentials aren't saved, or Studio started before they were | Save them (edition Part 1), then **⋯ → Restart Studio** |
| `dbt sl validate` says a column doesn't exist | A measure or dimension `expr` doesn't match `sem_transportation_airport_months` | Compare the YAML with the model's column names |
| Time-based questions return nothing | `service_month` is null for those rows, or the time spine doesn't cover the dates | Check the `service_month` warning count; ask the instructor about the date format |
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
