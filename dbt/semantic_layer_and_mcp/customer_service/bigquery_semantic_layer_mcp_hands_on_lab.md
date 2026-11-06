# dbt Semantic Layer and MCP add-on: Customer Service — BigQuery

*November 2026. This worksheet continues the Customer Service dbt Platform add-on after you have written your marts and before you build them. It adds governed metrics with the dbt Semantic Layer, publishes them through your Production environment, and connects Claude Code on your lab workstation to the dbt remote MCP server.*

## Objective

Turn your **Response-length review** marts into governed metrics that an AI assistant can query without writing its own SQL. You will add a semantic model and metrics on top of your fact, validate them in your development environment, publish them with a Production run, and then ask Claude Code business questions through the dbt MCP server.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `interactions` into your own destination.
- Your instructor launched the lab with an **individual dbt project** for each attendee and **dbt Semantic Layer + remote MCP** turned on. LabStudios has already created, for you only:
  - a dbt Platform project with a **Development** environment on the lab's BigQuery connection;
  - a **Production** environment whose deployment credential writes to your own prod dataset. It runs the `lab-work` branch (GitHub template labs) or `main` (dbt-managed repository labs);
  - a Semantic Layer configuration on that Production environment;
  - a lab **workstation** with Claude Code installed and configured.
- Complete **Parts 1–3** of either Customer Service edition, [manual](../../manual/customer_service/bigquery_manual_dbt_platform_hands_on_lab.md) or [dbt Wizard](../../wizard/customer_service/bigquery_wizard_dbt_platform_hands_on_lab.md), on the **`lab-work`** branch. Your Production environment runs `lab-work`, so your work must be there. (dbt-managed repository labs: see the variant at the end.)
- Stop after Part 3. You should have these files on `lab-work`, not yet built:
  - `models/marts/fct_customer_service_observations.sql`
  - `models/marts/mart_customer_service_review_summary.sql`
  - `models/marts/_customer_service__addon_models.yml`
  - `tests/assert_customer_service_addon.sql`
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

Your fact groups interactions by category. The source has **no interaction date**, so this lab skips trends and focuses on breakdowns. The provided model keeps your fact as written and joins back to staging for the intent, the language flags and the length of the customer's message. The Semantic Layer still needs one time column, so the model uses the date Fivetran loaded the row; its description tells Claude not to trend over it.

Create these three files. They are provided in full; read them before saving.

### `models/marts/sem_customer_service_interactions.sql`

```sql
{{ config(materialized='table') }}
-- Semantic Layer input: your tested fact plus the intent, flags and instruction length from staging.
-- One row per interactions row, exactly like fct_customer_service_observations.
select
    f.observation_id                                   as interaction_id,
    date(f.synced_at)                                  as load_date,
    f.group_name                                       as category,
    coalesce(cast(s.intent as string), 'Unknown')      as intent,
    coalesce(cast(s.flags as string), 'Unknown')       as flags,
    f.metric_value                                     as response_length,
    length(cast(s.instruction as string))              as instruction_length,
    f.requires_review
from {{ ref('fct_customer_service_observations') }} f
join {{ ref('stg_customer_service__interactions') }} s
    on s.id = f.observation_id
```

### `models/marts/metricflow_time_spine.sql`

The Semantic Layer needs a calendar table to roll metrics up by day, month or year.

```sql
{{ config(materialized='table') }}
{{ dbt.date_spine(
    datepart="day",
    start_date="cast('1990-01-01' as date)",
    end_date="cast('2036-01-01' as date)"
) }}
```

### `models/marts/_customer_service__semantic.yml`

```yaml
version: 2

models:
  - name: sem_customer_service_interactions
    description: "One row per customer service interaction. Semantic Layer input built from fct_customer_service_observations."
    columns:
      - name: interaction_id
        data_tests: [unique, not_null]
  - name: metricflow_time_spine
    description: "One row per calendar day, used by the Semantic Layer."
    time_spine:
      standard_granularity_column: date_day
    columns:
      - name: date_day
        granularity: day

semantic_models:
  - name: interactions
    description: "Customer service interactions synced by Fivetran. No interaction date, sentiment or satisfaction score exists in the source."
    model: ref('sem_customer_service_interactions')
    defaults:
      agg_time_dimension: load_date
    entities:
      - name: interaction
        type: primary
        expr: interaction_id
    dimensions:
      - name: load_date
        type: time
        description: "Date Fivetran loaded the row. The source has no interaction date, so do not use this for trends."
        type_params:
          time_granularity: day
      - name: category
        type: categorical
        description: "Interaction category."
      - name: intent
        type: categorical
        description: "Customer intent within the category."
      - name: flags
        type: categorical
        description: "Language-variation flags from the source dataset."
      - name: requires_review
        type: categorical
        description: "True for short responses: fewer than the review threshold (100) characters."
    measures:
      - name: interaction_count
        agg: sum
        expr: "1"
      - name: measured_count
        agg: count
        expr: response_length
      - name: response_length_avg
        agg: average
        expr: response_length
      - name: response_length_min
        agg: min
        expr: response_length
      - name: instruction_length_avg
        agg: average
        expr: instruction_length
      - name: intent_count
        agg: count_distinct
        expr: intent
      - name: short_response_count
        agg: sum
        expr: "case when requires_review then 1 else 0 end"

metrics:
  - name: interactions
    label: "Interactions"
    description: "Number of interactions."
    type: simple
    type_params:
      measure: interaction_count
  - name: measured_interactions
    label: "Measured interactions"
    description: "Interactions with a response. Denominator for short-response share."
    type: simple
    type_params:
      measure: measured_count
  - name: intents
    label: "Intents"
    description: "Distinct customer intents."
    type: simple
    type_params:
      measure: intent_count
  - name: average_response_length
    label: "Average response length"
    description: "Average response length in characters."
    type: simple
    type_params:
      measure: response_length_avg
  - name: shortest_response_length
    label: "Shortest response length"
    description: "Shortest response in characters."
    type: simple
    type_params:
      measure: response_length_min
  - name: average_instruction_length
    label: "Average customer message length"
    description: "Average length of the customer's message in characters."
    type: simple
    type_params:
      measure: instruction_length_avg
  - name: short_responses
    label: "Short responses"
    description: "Responses shorter than the review threshold (100 characters). Candidates for review, not evidence of poor quality."
    type: simple
    type_params:
      measure: short_response_count
  - name: short_response_share
    label: "Short-response share"
    description: "Short responses divided by measured interactions."
    type: ratio
    type_params:
      numerator: short_responses
      denominator: measured_interactions
  - name: response_to_message_ratio
    label: "Response-to-message length ratio"
    description: "Average response length divided by average customer message length. Above 1 means agents write more than customers."
    type: derived
    type_params:
      expr: average_response_length / average_instruction_length
      metrics:
        - name: average_response_length
          alias: average_response_length
        - name: average_instruction_length
          alias: average_instruction_length

saved_queries:
  - name: category_response_review
    description: "Interactions, average response length and short-response share by category."
    query_params:
      metrics: [interactions, average_response_length, short_response_share]
      group_by:
        - "Dimension('interaction__category')"
```

Notes on the definitions:
- Dimensions are referenced through their entity: `interaction__category`, `interaction__intent`. `metric_time` is the shared time axis, backed by `load_date`.
- `short_response_share` is a **ratio** metric: dbt sums the numerator and denominator for each group first, then divides. That is why it is correct at any grain, unlike averaging a per-row share.
- `response_to_message_ratio` is a **derived** metric built from two other metrics. dbt computes both for each group, then divides.
- `load_date` exists only because the Semantic Layer requires a time column. It is the Fivetran load date, not when the customer wrote in.

> **Checkpoint:** the three new files are saved next to your Part 3 files.

## Part 3: Build and validate in development

1. Check the resolved targets before building:

```bash
dbt ls --select +mart_customer_service_review_summary +sem_customer_service_interactions metricflow_time_spine --resource-type model --output json --output-keys name database schema
```

Every model must target your **BigQuery dbt Dataset** in the lab's project. Staging views use your dataset plus `_staging`.

2. Build your marts, the Semantic Layer input, the time spine and every test:

```bash
dbt build --select +mart_customer_service_review_summary +sem_customer_service_interactions metricflow_time_spine
```

Require a successful result. Fix failing tests in your Part 3 models rather than removing them. There is no date test: the source has no interaction date, and `load_date` is always filled.

3. Validate the semantic definitions:

```bash
dbt sl validate
dbt sl list metrics
```

Fix any errors. The usual cause is a measure or dimension `expr` that doesn't match a column in `sem_customer_service_interactions`.

4. Query a metric in development:

```bash
dbt sl query --metrics interactions,short_response_share --group-by interaction__category --order-by -interactions
```

Record the category with the most interactions. You will use it to check Claude Code's answers.

| Check | Your value |
|---|---|
| Category with the most interactions | |
| `interactions` in that category | |
| `short_response_share` in that category | |

> **Checkpoint:** the build passes, `dbt sl validate` succeeds, and `dbt sl query` returns one row per category.

## Part 4: Publish to Production

The Semantic Layer serves metrics from your **Production** environment's latest successful run, not from Studio. Until a Production run succeeds, Claude Code sees no metrics.

1. **Commit and sync** on `lab-work`: `dbt_project.yml`, your four Part 3 files and the three Semantic Layer files. Exclude credentials and unrelated changes.
2. Open **Orchestration → Environments → Production**. Confirm it runs the custom branch `lab-work`. Do not change its connection or credential; LabStudios configured them for you.
3. From the Production environment, choose **Create job → Deploy job**:
   - **Name:** `semantic_layer_build_<firstname>_<lastname>`
   - **Commands:** replace the default with:

   ```bash
   dbt build --select +mart_customer_service_review_summary +sem_customer_service_interactions metricflow_time_spine
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
| 1 | *"What metrics, dimensions and saved queries are available in my dbt project? Explain each metric in one line, and tell me whether there is a date I can trend by."* | `list_metrics`, `get_dimensions`, `list_saved_queries` | It lists your 9 metrics using **your** descriptions, finds `category_response_review`, and says `load_date` is a load date, not an interaction date. |
| 2 | *"Rank the categories by number of interactions, and show the average response length and short-response share for each."* | `query_metrics` grouped by `interaction__category` | One row per category. The top category matches your Part 3 baseline. |
| 3 | *"Which intents exist in the category with the most interactions? Show the five intents there with the highest short-response share, among intents with at least 20 interactions."* | `get_dimension_values` for `interaction__intent`, then `query_metrics` with a `where` filter on `interaction__category` | Five intents, each with at least 20 interactions. Ask it to show the filter it used. |
| 4 | *"How is short_response_share calculated? Show me the SQL the Semantic Layer runs for it, and which dbt models it depends on."* | `get_metrics_compiled_sql`, plus a lineage or model-details tool | The SQL reads `sem_customer_service_interactions` in your **prod dataset** and divides two sums. The lineage runs from the `interactions` source through staging and your fact. |
| 5 | *"Which categories have the highest response-to-message length ratio, and what does that ratio measure? Then tell me whether the models behind it passed their tests in the last production run."* | `query_metrics` with `response_to_message_ratio` by category, then a model-health tool | A ratio per category with a plain-language explanation from your description, and a statement that your Production run's tests passed. |

Then ask one question your metrics can't answer: *"How has customer satisfaction changed over the last six months?"* There is no satisfaction metric and no interaction date. A good answer says so instead of inventing a number. Discuss with your instructor why a governed metric layer matters for AI answers.

Claude Code's exact wording and tool order vary between runs. What matters is that numbers come from `query_metrics` and match your Part 3 baseline at the same grain.

**Stretch (optional):** change the review rule. Edit `review_threshold` in `dbt_project.yml` from `100` to `200`, commit, rerun your Production job, and ask question 2 again. The short-response shares change only after the Production run succeeds, because the Semantic Layer reads Production. Restore `100` afterwards and rerun the job.

> **Checkpoint:** Claude Code answered the five questions from your metrics, and declined or qualified the question it couldn't answer.

## What you did

You layered governed metrics onto your tested marts: a semantic model, simple, ratio and derived metrics, and a saved query, on a dataset with no business date. You validated them in development, published them through a Production run, and had an AI assistant answer business questions through the dbt MCP server from those definitions instead of ad-hoc SQL.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Studio stays **Disconnected** | Your personal BigQuery credentials aren't saved, or Studio started before they were | Save them (edition Part 1), then **⋯ → Restart Studio** |
| `dbt sl validate` says a column doesn't exist | A measure or dimension `expr` doesn't match `sem_customer_service_interactions` | Compare the YAML with the model's column names |
| Claude trends interactions by month | It used `load_date`, the only time column | Remind it the source has no interaction date; ask for a breakdown instead |
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
