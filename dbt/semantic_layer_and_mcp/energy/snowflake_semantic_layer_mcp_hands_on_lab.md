# dbt Semantic Layer and MCP add-on: Energy — Snowflake

*November 2026. This worksheet continues the Energy dbt Platform add-on after you have written your marts and before you build them. It adds governed metrics with the dbt Semantic Layer, publishes them through your Production environment, and connects Claude Code on your lab workstation to the dbt remote MCP server.*

## Objective

Turn your **Crude price monitoring** marts into governed metrics that an AI assistant can query without writing its own SQL. You will add a semantic model and metrics on top of your fact, validate them in your development environment, publish them with a Production run, and then ask Claude Code business questions through the dbt MCP server.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `commodity_prices` into your own destination.
- Your instructor launched the lab with an **individual dbt project** for each attendee and **dbt Semantic Layer + remote MCP** turned on. LabStudios has already created, for you only:
  - a dbt Platform project with a **Development** environment that points at your personal dev target;
  - a **Production** environment with its deployment credential. It runs the `lab-work` branch (GitHub template labs) or `main` (dbt-managed repository labs);
  - a Semantic Layer configuration on that Production environment;
  - a lab **workstation** with Claude Code installed and configured.
- Complete **Parts 1–3** of either Energy edition, [manual](../../manual/energy/snowflake_manual_dbt_platform_hands_on_lab.md) or [dbt Wizard](../../wizard/energy/snowflake_wizard_dbt_platform_hands_on_lab.md), on the **`lab-work`** branch. Your Production environment runs `lab-work`, so your work must be there. (dbt-managed repository labs: see the variant at the end.)
- Stop after Part 3. You should have these files on `lab-work`, not yet built:
  - `models/marts/fct_energy_observations.sql`
  - `models/marts/mart_energy_review_summary.sql`
  - `models/marts/_energy__addon_models.yml`
  - `tests/assert_energy_addon.sql`
- Open your **1Password lab item** from your credential link. You will use these fields:

| 1Password field | Used in |
|---|---|
| Snowflake Dev Database, Snowflake Prod Database | Parts 1 and 4: confirm your targets |
| dbt Platform Project URL | Part 1: open your project |
| Gateway URL, Guacamole Username, Guacamole Password | Part 6: sign in to your workstation |
| dbt MCP Command | Part 6: connect Claude Code |

- Never paste a password, private key or token into SQL, YAML, Git or a Claude prompt. The only secret you type in this lab is your own dbt personal access token, into the `claude mcp add` command on your own workstation.

## Part 1: Confirm your branch and dev target

1. Open your **dbt Platform Project URL** and open **Studio**. Wait for the status to show **Parsed**. If it stays **Disconnected**, open the **⋯** menu, choose **Restart Studio** and wait again.
2. Confirm the branch selector shows **`lab-work`**. If not, choose **Change branch → `lab-work`**. (`main` is read-only in Studio.)
3. Confirm your dev target in the Studio command bar:

```bash
dbt show --inline "select '{{ target.database }}' as target_database, '{{ target.schema }}' as target_schema"
```

`target_database` must be your provisioned database ending in `_DEV`, and `target_schema` your personal dev schema. If either is wrong, fix your credentials before continuing (Part 1 of the edition worksheet).

> **Checkpoint:** Studio is **Parsed**, you are on `lab-work`, your four Part 3 files are present, and `dbt show` returns your own dev target.

## Part 2: Add the Semantic Layer files

A **semantic model** tells dbt what one row means, which columns people can group and filter by, and which numbers can be added up. **Metrics** are the named business numbers people ask for. Claude Code can only answer with the metrics you define here, and it reads each metric's description to decide which one fits a question.

Your fact keeps only WTI crude, from a wide source table that holds two dozen commodities as columns. The Semantic Layer works best with one row per thing you measure, so the provided model **unpivots** nine of those columns into one row per day and commodity, with a `commodity` dimension. It joins your fact back in to carry its WTI review flag, so your tested logic still decides which WTI quotes need review. UNPIVOT drops empty prices, so every row has a price.

Create these three files. They are provided in full; read them before saving.

### `models/marts/sem_energy_commodity_quotes.sql`

```sql
{{ config(materialized='table') }}
-- Semantic Layer input: one row per trading day and commodity.
-- Unpivots nine of the wide price columns and carries your fact's WTI review flag.
with daily as (
    select
        s.id,
        try_to_date(left(cast(s.date as varchar), 10))     as quote_date,
        try_to_double(cast(s.wti_crude as varchar))        as wti_crude,
        try_to_double(cast(s.brent_crude as varchar))      as brent_crude,
        try_to_double(cast(s.natural_gas as varchar))      as natural_gas,
        try_to_double(cast(s.gasoline as varchar))         as gasoline,
        try_to_double(cast(s.uls_diesel as varchar))       as uls_diesel,
        try_to_double(cast(s.gold as varchar))             as gold,
        try_to_double(cast(s.copper as varchar))           as copper,
        try_to_double(cast(s.corn as varchar))             as corn,
        try_to_double(cast(s.wheat as varchar))            as wheat
    from {{ ref('stg_energy__commodity_prices') }} s
),
unpivoted as (
    select id, quote_date, lower(commodity_code) as commodity, price
    from daily
    unpivot (price for commodity_code in
        (wti_crude, brent_crude, natural_gas, gasoline, uls_diesel, gold, copper, corn, wheat))
)
select
    l.id || ':' || l.commodity                             as commodity_quote_id,
    l.quote_date,
    l.commodity,
    case
        when l.commodity in ('wti_crude', 'brent_crude', 'natural_gas', 'gasoline', 'uls_diesel') then 'Energy'
        when l.commodity in ('gold', 'copper') then 'Metals'
        else 'Grains'
    end                                                    as commodity_group,
    l.price,
    case when l.commodity = 'wti_crude' then f.requires_review end as requires_review
from unpivoted l
left join {{ ref('fct_energy_observations') }} f
    on f.observation_id = l.id
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

### `models/marts/_energy__semantic.yml`

```yaml
version: 2

models:
  - name: sem_energy_commodity_quotes
    description: "One row per trading day and commodity. Semantic Layer input built from stg_energy__commodity_prices and fct_energy_observations."
    columns:
      - name: commodity_quote_id
        data_tests: [unique, not_null]
      - name: quote_date
        description: "Date of the commodity quote. Rows without a parseable date are excluded from time-based metrics."
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
  - name: commodity_quotes
    description: "Daily commodity prices synced by Fivetran, one row per day and commodity. Each commodity has its own unit (for example per barrel or per bushel), so compare a commodity only with itself."
    model: ref('sem_energy_commodity_quotes')
    defaults:
      agg_time_dimension: quote_date
    entities:
      - name: commodity_quote
        type: primary
        expr: commodity_quote_id
    dimensions:
      - name: quote_date
        type: time
        type_params:
          time_granularity: day
      - name: commodity
        type: categorical
        description: "Commodity code, for example wti_crude or brent_crude."
      - name: commodity_group
        type: categorical
        description: "Energy, Metals or Grains."
      - name: requires_review
        type: categorical
        description: "WTI rows only: true when the WTI price is at or above the review threshold (70). Empty for other commodities."
    measures:
      - name: quote_count
        agg: sum
        expr: "1"
      - name: price_avg
        agg: average
        expr: price
      - name: price_max
        agg: max
        expr: price
      - name: price_min
        agg: min
        expr: price
      - name: wti_review_count
        agg: sum
        expr: "case when requires_review then 1 else 0 end"

metrics:
  - name: quotes
    label: "Quotes"
    description: "Number of daily commodity quotes with a price."
    type: simple
    type_params:
      measure: quote_count
  - name: average_price
    label: "Average price"
    description: "Unweighted average daily price in each commodity's own unit. Only meaningful when grouped by commodity."
    type: simple
    type_params:
      measure: price_avg
  - name: highest_price
    label: "Highest price"
    description: "Highest daily price in each commodity's own unit. Group by commodity."
    type: simple
    type_params:
      measure: price_max
  - name: lowest_price
    label: "Lowest price"
    description: "Lowest daily price in each commodity's own unit. Group by commodity."
    type: simple
    type_params:
      measure: price_min
  - name: wti_average_price
    label: "WTI average price"
    description: "Average daily WTI crude price."
    type: simple
    filter: "{{ Dimension('commodity_quote__commodity') }} = 'wti_crude'"
    type_params:
      measure: price_avg
  - name: brent_average_price
    label: "Brent average price"
    description: "Average daily Brent crude price."
    type: simple
    filter: "{{ Dimension('commodity_quote__commodity') }} = 'brent_crude'"
    type_params:
      measure: price_avg
  - name: brent_wti_spread
    label: "Brent-WTI spread"
    description: "Brent average price minus WTI average price for the period. Positive means Brent traded above WTI."
    type: derived
    type_params:
      expr: brent - wti
      metrics:
        - name: brent_average_price
          alias: brent
        - name: wti_average_price
          alias: wti
  - name: wti_quotes
    label: "WTI quotes"
    description: "Daily WTI quotes with a price. Denominator for WTI review share."
    type: simple
    filter: "{{ Dimension('commodity_quote__commodity') }} = 'wti_crude'"
    type_params:
      measure: quote_count
  - name: wti_quotes_needing_review
    label: "WTI quotes needing review"
    description: "WTI quotes at or above the review threshold (70). An illustrative monitoring flag, not a trading signal."
    type: simple
    type_params:
      measure: wti_review_count
  - name: wti_review_share
    label: "WTI review share"
    description: "WTI quotes needing review divided by WTI quotes."
    type: ratio
    type_params:
      numerator: wti_quotes_needing_review
      denominator: wti_quotes
  - name: wti_average_price_vs_prior_year
    label: "WTI average price, change vs prior year"
    description: "WTI average price minus the same metric one year earlier. Use with a monthly or yearly time grain."
    type: derived
    type_params:
      expr: current_period - prior_year
      metrics:
        - name: wti_average_price
          alias: current_period
        - name: wti_average_price
          offset_window: 1 year
          alias: prior_year

saved_queries:
  - name: monthly_crude_prices
    description: "Monthly WTI and Brent average prices and the Brent-WTI spread."
    query_params:
      metrics: [wti_average_price, brent_average_price, brent_wti_spread]
      group_by:
        - "TimeDimension('metric_time', 'month')"
```

Notes on the definitions:
- Dimensions are referenced through their entity: `commodity_quote__commodity`, `commodity_quote__commodity_group`. `metric_time` is the shared time axis, backed by `quote_date`.
- `wti_review_share` is a **ratio** metric: dbt sums the numerator and denominator for each group first, then divides. That is why it is correct at any grain, unlike averaging a per-row share.
- `wti_average_price_vs_prior_year` is a **derived** metric. Its `offset_window` reads the same metric one year back through the time spine.
- `wti_average_price`, `brent_average_price` and `wti_quotes` are simple metrics with a **filter**: the same measure, restricted to one commodity. `brent_wti_spread` is a derived metric that subtracts one filtered metric from the other.

> **Checkpoint:** the three new files are saved next to your Part 3 files.

## Part 3: Build and validate in development

1. Check the resolved targets before building:

```bash
dbt ls --select +mart_energy_review_summary +sem_energy_commodity_quotes metricflow_time_spine --resource-type model --output json --output-keys name database schema
```

Every model must target your `_DEV` database. Staging views use your dev schema plus `_staging`.

2. Build your marts, the Semantic Layer input, the time spine and every test:

```bash
dbt build --select +mart_energy_review_summary +sem_energy_commodity_quotes metricflow_time_spine
```

Require a successful result. Fix failing tests in your Part 3 models rather than removing them. The `quote_date` test is a warning, not a failure: if it warns, some quotes lack a parseable date and are left out of time-based answers. Note the count and discuss it with your instructor.

3. Validate the semantic definitions:

```bash
dbt sl validate
dbt sl list metrics
```

Fix any errors. The usual cause is a measure or dimension `expr` that doesn't match a column in `sem_energy_commodity_quotes`.

4. Query a metric in development:

```bash
dbt sl query --metrics wti_average_price,wti_quotes_needing_review --group-by metric_time__year --order-by metric_time__year
```

Record the most recent year in the results. You will use it to check Claude Code's answers.

| Check | Your value |
|---|---|
| Latest year with quotes | |
| `wti_average_price` in that year | |
| `wti_quotes_needing_review` in that year | |

> **Checkpoint:** the build passes, `dbt sl validate` succeeds, and `dbt sl query` returns one row per year.

## Part 4: Publish to Production

The Semantic Layer serves metrics from your **Production** environment's latest successful run, not from Studio. Until a Production run succeeds, Claude Code sees no metrics.

1. **Commit and sync** on `lab-work`: `dbt_project.yml`, your four Part 3 files and the three Semantic Layer files. Exclude credentials and unrelated changes.
2. Open **Orchestration → Environments → Production**. Confirm it runs the custom branch `lab-work`. Do not change its connection or credential; LabStudios configured them for you.
3. From the Production environment, choose **Create job → Deploy job**:
   - **Name:** `semantic_layer_build_<firstname>_<lastname>`
   - **Commands:** replace the default with:

   ```bash
   dbt build --select +mart_energy_review_summary +sem_energy_commodity_quotes metricflow_time_spine
   ```

   - Turn off schedules and job-completion triggers. Save.
4. Click **Run now** and wait for **Success**. Open the run and confirm:
   - the commit is your latest on `lab-work`;
   - the models were built in your **prod** database (ending `_PROD`), schema `DBT_PROD`;
   - the tests ran and passed.
5. If the run fails, read the failing step. A failure in **Clone** means the commit isn't on `lab-work`; a profile or credential error goes to your instructor.

> **Checkpoint:** a Production run succeeded from your latest `lab-work` commit. Your metrics are published.

## Part 5: Create your token and find your user ID

1. **Create a personal access token:** **Account settings → API tokens → Personal tokens → Create personal access token**. Name it `lab-mcp`. Copy it now; dbt shows it only once. Treat it like a password: don't paste it anywhere except the command in Part 6.
2. **Find your dbt user ID:** **Account settings → Users**, then select your own name. The number at the end of the page URL is your user ID.

Use a personal access token, not a **Semantic Layer credential**. The Semantic Layer configuration page offers to create credentials and service tokens; you don't need either for this lab.

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
| 1 | *"What metrics, dimensions and saved queries are available in my dbt project? Explain each metric in one line."* | `list_metrics`, `get_dimensions`, `list_saved_queries` | It lists your 11 metrics using **your** descriptions, and finds `monthly_crude_prices`. |
| 2 | *"Show the WTI and Brent average prices and the Brent-WTI spread by month for the most recent 12 months in the data."* | `query_metrics` with the saved query or the three metrics by `metric_time__month` | Twelve monthly rows ending in your latest year from Part 3. The spread equals Brent minus WTI in each row. |
| 3 | *"Which commodities are in the data? For each energy commodity, show the average, highest and lowest price in the most recent year."* | `get_dimension_values` for `commodity_quote__commodity`, then `query_metrics` with a `where` filter on `commodity_quote__commodity_group` | One row per energy commodity. It should not compare prices across commodities, because their units differ. |
| 4 | *"How is wti_review_share calculated? Show me the SQL the Semantic Layer runs for it, and which dbt models it depends on."* | `get_metrics_compiled_sql`, plus a lineage or model-details tool | The SQL reads your **prod** `sem_energy_commodity_quotes` table and divides two sums. The lineage runs from the `commodity_prices` source through staging and your fact. |
| 5 | *"For the most recent year in the data, compare the WTI average price each month with the same month one year earlier. Then tell me whether the models behind this metric passed their tests in the last production run."* | `query_metrics` with `wti_average_price_vs_prior_year` by month, then a model-health tool | Monthly changes, positive or negative, and a statement that your Production run's tests passed. |

Then ask one question your metrics can't answer: *"What will WTI cost next quarter?"* There is no forecast metric; these are historical quotes. A good answer says so instead of inventing a number. Discuss with your instructor why a governed metric layer matters for AI answers.

Claude Code's exact wording and tool order vary between runs. What matters is that numbers come from `query_metrics` and match your Part 3 baseline at the same grain.

**Stretch (optional):** change the review rule. Edit `review_threshold` in `dbt_project.yml` from `70` to `90`, commit, rerun your Production job, and ask Claude Code for the WTI review share in the most recent year. The WTI review counts change only after the Production run succeeds, because the Semantic Layer reads Production. Restore `70` afterwards and rerun the job.

> **Checkpoint:** Claude Code answered the five questions from your metrics, and declined or qualified the question it couldn't answer.

## What you did

You reshaped a wide price table into one row per day and commodity, then layered governed metrics on it: simple metrics with filters, ratio and derived metrics, and a saved query. You validated them in development, published them through a Production run, and had an AI assistant answer business questions through the dbt MCP server from those definitions instead of ad-hoc SQL.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Studio stays **Disconnected** | Studio started before your credential was saved | **⋯ → Restart Studio** |
| `dbt sl validate` says a column doesn't exist | A measure or dimension `expr` doesn't match `sem_energy_commodity_quotes` | Compare the YAML with the model's column names |
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
- [Metric filters](https://docs.getdbt.com/docs/build/metrics-overview#filters)
- [Saved queries](https://docs.getdbt.com/docs/build/saved-queries)
- [MetricFlow time spine](https://docs.getdbt.com/docs/build/metricflow-time-spine)
- [dbt remote MCP server](https://docs.getdbt.com/docs/dbt-ai/setup-remote-mcp)
