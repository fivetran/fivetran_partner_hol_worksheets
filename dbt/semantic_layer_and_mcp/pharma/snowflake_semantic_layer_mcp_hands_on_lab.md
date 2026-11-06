# dbt Semantic Layer and MCP add-on: Pharma — Snowflake

*November 2026. This worksheet continues the Pharma dbt Platform add-on after you have written your marts and before you build them. It adds governed metrics with the dbt Semantic Layer, publishes them through your Production environment, and connects Claude Code on your lab workstation to the dbt remote MCP server.*

## Objective

Turn your **Trial dropout review** marts into governed metrics that an AI assistant can query without writing its own SQL. You will add a semantic model and metrics on top of your fact, validate them in your development environment, publish them with a Production run, and then ask Claude Code business questions through the dbt MCP server.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `phr_records` into your own destination.
- Your instructor launched the lab with an **individual dbt project** for each attendee and **dbt Semantic Layer + remote MCP** turned on. LabStudios has already created, for you only:
  - a dbt Platform project with a **Development** environment that points at your personal dev target;
  - a **Production** environment with its deployment credential. It runs the `lab-work` branch (GitHub template labs) or `main` (dbt-managed repository labs);
  - a Semantic Layer configuration on that Production environment;
  - a lab **workstation** with Claude Code installed and configured.
- Complete **Parts 1–3** of either Pharma edition, [manual](../../pharma/snowflake_manual_dbt_platform_hands_on_lab.md) or [dbt Wizard](../../pharma/snowflake_wizard_dbt_platform_hands_on_lab.md), on the **`lab-work`** branch. Your Production environment runs `lab-work`, so your work must be there. (dbt-managed repository labs: see the variant at the end.)
- Stop after Part 3. You should have these files on `lab-work`, not yet built:
  - `models/marts/fct_pharma_observations.sql`
  - `models/marts/mart_pharma_review_summary.sql`
  - `models/marts/_pharma__addon_models.yml`
  - `tests/assert_pharma_addon.sql`
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

Your fact groups dropout rates by disease area and has no date. The provided model keeps your fact as written and joins back to staging for the enrollment date, trial status, sponsor and patient gender, plus the enrollment rate. It also keeps `trial_id`, so one metric can count distinct trials: records repeat trial context.

Create these three files. They are provided in full; read them before saving.

### `models/marts/sem_pharma_enrollment_records.sql`

```sql
{{ config(materialized='table') }}
-- Semantic Layer input: your tested fact plus enrollment_date and extra attributes from staging.
-- One row per phr_records row, exactly like fct_pharma_observations.
select
    f.observation_id                                   as enrollment_record_id,
    try_to_date(left(cast(s.enrollment_date as varchar), 10)) as enrollment_date,
    f.group_name                                       as disease_area,
    coalesce(cast(s.trial_status as varchar), 'Unknown') as trial_status,
    coalesce(cast(s.sponsor_name as varchar), 'Unknown') as sponsor_name,
    coalesce(cast(s.patient_gender as varchar), 'Unknown') as patient_gender,
    s.trial_id,
    f.metric_value                                     as dropout_rate,
    try_to_double(cast(s.enrollment_rate as varchar))  as enrollment_rate,
    f.requires_review
from {{ ref('fct_pharma_observations') }} f
join {{ ref('stg_pharma__phr_records') }} s
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

### `models/marts/_pharma__semantic.yml`

```yaml
version: 2

models:
  - name: sem_pharma_enrollment_records
    description: "One row per trial enrollment record. Semantic Layer input built from fct_pharma_observations."
    columns:
      - name: enrollment_record_id
        data_tests: [unique, not_null]
      - name: enrollment_date
        description: "Date of the enrollment record. Rows without a parseable date are excluded from time-based metrics."
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
  - name: enrollment_records
    description: "Trial enrollment records synced by Fivetran. Unweighted observation summaries, not trial-level clinical conclusions."
    model: ref('sem_pharma_enrollment_records')
    defaults:
      agg_time_dimension: enrollment_date
    entities:
      - name: enrollment_record
        type: primary
        expr: enrollment_record_id
    dimensions:
      - name: enrollment_date
        type: time
        type_params:
          time_granularity: day
      - name: disease_area
        type: categorical
        description: "Disease area of the trial."
      - name: trial_status
        type: categorical
        description: "Trial status."
      - name: sponsor_name
        type: categorical
        description: "Trial sponsor."
      - name: patient_gender
        type: categorical
        description: "Patient gender on the record."
      - name: requires_review
        type: categorical
        description: "True for high dropout: rate at or above the review threshold (20)."
    measures:
      - name: record_count
        agg: sum
        expr: "1"
      - name: rated_count
        agg: count
        expr: dropout_rate
      - name: trial_count
        agg: count_distinct
        expr: trial_id
      - name: dropout_avg
        agg: average
        expr: dropout_rate
      - name: dropout_max
        agg: max
        expr: dropout_rate
      - name: enrollment_rate_avg
        agg: average
        expr: enrollment_rate
      - name: high_dropout_count
        agg: sum
        expr: "case when requires_review then 1 else 0 end"

metrics:
  - name: enrollment_records
    label: "Enrollment records"
    description: "Number of enrollment records. Not distinct trials or patients."
    type: simple
    type_params:
      measure: record_count
  - name: records_with_dropout_rate
    label: "Records with a dropout rate"
    description: "Records with a dropout rate. Denominator for high-dropout share."
    type: simple
    type_params:
      measure: rated_count
  - name: trials
    label: "Trials"
    description: "Distinct trials with at least one record."
    type: simple
    type_params:
      measure: trial_count
  - name: average_dropout_rate
    label: "Average dropout rate"
    description: "Unweighted average of record dropout rates. Not a trial-level rate."
    type: simple
    type_params:
      measure: dropout_avg
  - name: highest_dropout_rate
    label: "Highest dropout rate"
    description: "Highest dropout rate on any record."
    type: simple
    type_params:
      measure: dropout_max
  - name: average_enrollment_rate
    label: "Average enrollment rate"
    description: "Unweighted average enrollment rate."
    type: simple
    type_params:
      measure: enrollment_rate_avg
  - name: high_dropout_records
    label: "High-dropout records"
    description: "Records with a dropout rate at or above the review threshold (20). An illustrative flag, not a clinical conclusion."
    type: simple
    type_params:
      measure: high_dropout_count
  - name: high_dropout_share
    label: "High-dropout share"
    description: "High-dropout records divided by records with a dropout rate."
    type: ratio
    type_params:
      numerator: high_dropout_records
      denominator: records_with_dropout_rate
  - name: high_dropout_records_vs_prior_year
    label: "High-dropout records, change vs prior year"
    description: "High-dropout records minus the same metric one year earlier. Use with a monthly or yearly time grain."
    type: derived
    type_params:
      expr: current_period - prior_year
      metrics:
        - name: high_dropout_records
          alias: current_period
        - name: high_dropout_records
          offset_window: 1 year
          alias: prior_year

saved_queries:
  - name: monthly_dropout_review
    description: "Monthly average dropout rate, high-dropout records and high-dropout share by disease area."
    query_params:
      metrics: [average_dropout_rate, high_dropout_records, high_dropout_share]
      group_by:
        - "TimeDimension('metric_time', 'month')"
        - "Dimension('enrollment_record__disease_area')"
```

Notes on the definitions:
- Dimensions are referenced through their entity: `enrollment_record__disease_area`, `enrollment_record__trial_status`. `metric_time` is the shared time axis, backed by `enrollment_date`.
- `high_dropout_share` is a **ratio** metric: dbt sums the numerator and denominator for each group first, then divides. That is why it is correct at any grain, unlike averaging a per-row share.
- `high_dropout_records_vs_prior_year` is a **derived** metric. Its `offset_window` reads the same metric one year back through the time spine.

> **Checkpoint:** the three new files are saved next to your Part 3 files.

## Part 3: Build and validate in development

1. Check the resolved targets before building:

```bash
dbt ls --select +mart_pharma_review_summary +sem_pharma_enrollment_records metricflow_time_spine --resource-type model --output json --output-keys name database schema
```

Every model must target your `_DEV` database. Staging views use your dev schema plus `_staging`.

2. Build your marts, the Semantic Layer input, the time spine and every test:

```bash
dbt build --select +mart_pharma_review_summary +sem_pharma_enrollment_records metricflow_time_spine
```

Require a successful result. Fix failing tests in your Part 3 models rather than removing them. The `enrollment_date` test is a warning, not a failure: if it warns, some records lack a parseable date and are left out of time-based answers. Note the count and discuss it with your instructor.

3. Validate the semantic definitions:

```bash
dbt sl validate
dbt sl list metrics
```

Fix any errors. The usual cause is a measure or dimension `expr` that doesn't match a column in `sem_pharma_enrollment_records`.

4. Query a metric in development:

```bash
dbt sl query --metrics average_dropout_rate,high_dropout_records --group-by metric_time__year --order-by metric_time__year
```

Record the most recent year in the results. You will use it to check Claude Code's answers.

| Check | Your value |
|---|---|
| Latest year with records | |
| `average_dropout_rate` in that year | |
| `high_dropout_records` in that year | |

> **Checkpoint:** the build passes, `dbt sl validate` succeeds, and `dbt sl query` returns one row per year.

## Part 4: Publish to Production

The Semantic Layer serves metrics from your **Production** environment's latest successful run, not from Studio. Until a Production run succeeds, Claude Code sees no metrics.

1. **Commit and sync** on `lab-work`: `dbt_project.yml`, your four Part 3 files and the three Semantic Layer files. Exclude credentials and unrelated changes.
2. Open **Orchestration → Environments → Production**. Confirm it runs the custom branch `lab-work`. Do not change its connection or credential; LabStudios configured them for you.
3. From the Production environment, choose **Create job → Deploy job**:
   - **Name:** `semantic_layer_build_<firstname>_<lastname>`
   - **Commands:** replace the default with:

   ```bash
   dbt build --select +mart_pharma_review_summary +sem_pharma_enrollment_records metricflow_time_spine
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
| 1 | *"What metrics, dimensions and saved queries are available in my dbt project? Explain each metric in one line."* | `list_metrics`, `get_dimensions`, `list_saved_queries` | It lists your 9 metrics using **your** descriptions, and finds `monthly_dropout_review`. |
| 2 | *"Show the average dropout rate, average enrollment rate and enrollment records by month for the most recent 12 months in the data."* | `query_metrics` grouped by `metric_time__month` | Twelve monthly rows ending in your latest year from Part 3. It should describe the rates as unweighted. |
| 3 | *"Which trial statuses and sponsors are in the data? For each disease area, show the high-dropout share and the number of distinct trials, only for disease areas with at least 30 records with a dropout rate."* | `get_dimension_values` for `enrollment_record__trial_status`, then `query_metrics` with `high_dropout_share`, `trials` and `records_with_dropout_rate` | A share between 0 and 1 and a trial count per disease area. Trials are fewer than records. |
| 4 | *"How is high_dropout_share calculated? Show me the SQL the Semantic Layer runs for it, and which dbt models it depends on."* | `get_metrics_compiled_sql`, plus a lineage or model-details tool | The SQL reads your **prod** `sem_pharma_enrollment_records` table and divides two sums. The lineage runs from the `phr_records` source through staging and your fact. |
| 5 | *"For the most recent year in the data, compare high-dropout records each month with the same month one year earlier. Then tell me whether the models behind this metric passed their tests in the last production run."* | `query_metrics` with `high_dropout_records_vs_prior_year` by month, then a model-health tool | Monthly changes, positive or negative, and a statement that your Production run's tests passed. |

Then ask one question your metrics can't answer: *"Is the drug in our largest trial effective?"* There is no efficacy or outcome metric; the data covers enrollment and dropout only. A good answer says so instead of inventing a number. Discuss with your instructor why a governed metric layer matters for AI answers.

Claude Code's exact wording and tool order vary between runs. What matters is that numbers come from `query_metrics` and match your Part 3 baseline at the same grain.

**Stretch (optional):** change the review rule. Edit `review_threshold` in `dbt_project.yml` from `20` to `30`, commit, rerun your Production job, and ask question 3 again. The shares change only after the Production run succeeds, because the Semantic Layer reads Production. Restore `20` afterwards and rerun the job.

> **Checkpoint:** Claude Code answered the five questions from your metrics, and declined or qualified the question it couldn't answer.

## What you did

You layered governed metrics onto your tested marts: a semantic model, simple, ratio and derived metrics, and a saved query. You validated them in development, published them through a Production run, and had an AI assistant answer business questions through the dbt MCP server from those definitions instead of ad-hoc SQL.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Studio stays **Disconnected** | Studio started before your credential was saved | **⋯ → Restart Studio** |
| `dbt sl validate` says a column doesn't exist | A measure or dimension `expr` doesn't match `sem_pharma_enrollment_records` | Compare the YAML with the model's column names |
| Time-based questions return nothing | `enrollment_date` is null for those rows, or the time spine doesn't cover the dates | Check the `enrollment_date` warning count; ask the instructor about the date format |
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
