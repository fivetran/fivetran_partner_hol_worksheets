# dbt Platform add-on: Financial Services — BigQuery without dbt Wizard

*October 2026. This worksheet starts after the source lab's initial sync and follows the dbt Platform portion of the Fivetran + dbt lab. It ends with a deployment job; no Cortex or destination-console exercise is included.*

## Objective

Build **Loan amount review** from the synced Financial Services data. Configure your personal development target, create two tested marts without dbt Wizard, compare dbt State decisions, then run a job from your own branch in an isolated deployment target.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `loan` into your own destination. Use **Fivetran → Connection → Schema** to confirm names and selected columns.
- Accept the invitation to the provisioned dbt Platform project, with a Developer seat and Git access to a generated copy of [dbt-financial-services-lab](https://github.com/dbt-Hands-on-Labs/dbt-financial-services-lab). The instructor provides project-scoped environment/job creation permissions and an approved deployment connection profile.
- The instructor enables dbt v2 and State for the session. If a required feature is unavailable, resolve it with the instructor; do not start a trial or change shared account settings.
- The connection must read the landing data and create models in your development/deployment targets and their `_staging` schemas/datasets. Credentials belong only in the provisioned credential forms, never in SQL, YAML, prompts or Git.
- All destination data previews are instructor-led. No BigQuery Studio access is provided or needed. Perform the validation commands below in **dbt Studio**. Local DuckDB previews from the SDK lab remain hands-on.

## Part 1: Access dbt Platform and isolate your work

1. Accept your dbt Platform lab invitation, complete registration and select the project named `LAB_DBT...` assigned by your instructor. Do not create a trial or use another lab's project.
2. Open **Your profile → Credentials → your assigned project**. Alternatively, from Studio's **Status: Disconnected** menu, open **Edit environment**, find **Connection → User credentials**, and follow the **profile page** link. Use that link only to reach your personal credentials; do not edit the shared environment or connection. Personal credentials apply to your dbt login, so each participant must use their own login.
3. Under **Credentials for project**, click **Edit**. Verify that the **Project ID** is `fivetran-hands-on-lab` and **Service Account JSON** is selected. LabStudios configures the runner service account on the existing BigQuery connection; do not upload your Fivetran destination key. If the project or authentication method differs, ask the instructor to check the provisioned connection.
4. In **Dataset**, enter the **BigQuery dbt dataset** from your assigned 1Password lab item, such as `dbt_bqp042`. Use your participant-specific dbt dataset, not the Fivetran source dataset or another attendee's dataset. Leave **Target name** and the other settings unchanged.
5. Click **Test connection**. Wait for setup and connection testing to finish, and confirm that **Complete** is green. Then click **Save** and wait for the green **Saved** confirmation.
6. Return to **Studio**. Its status may show **Parsing** and **Checking connection** while it starts. Wait until the status is green and **Connected** before continuing. If testing fails or Studio remains disconnected after saving, ask the instructor to troubleshoot the provisioned connection. Attendees do not need BigQuery Studio, Google Cloud Console or IAM access.
7. Create your personal Git branch `<firstname>_<lastname>_dbt_addon`. Work on that branch throughout the exercise.
8. Open **dbt version → Personal version override**, select the instructor-approved **v2 Stable** runtime, and save/restart. Do not edit the shared Development environment. Confirm the actual runtime in command logs; UI wording can change.

> **Checkpoint:** Studio connects to your assigned BigQuery target, your personal branch is selected and the runtime is confirmed.

## Part 2: Configure and inspect the source

1. Record these identifiers. Copy the actual names from your credentials and Fivetran schema configuration; do not infer them from another attendee's lab.

| Placeholder | Your value |
|---|---|
| `<SOURCE_DATABASE>` | The BigQuery Project ID containing the synced source dataset |
| `<SOURCE_SCHEMA>` | The exact Fivetran dataset, including assigned prefix, your name and source-schema suffix |
| `<DEV_DATABASE>` | The BigQuery project configured in dbt |
| `<DEV_SCHEMA>` | Your assigned dataset, for example `dbt_bqp042` |
| `<DEPLOY_SCHEMA>` | Your assigned separate deployment dataset, for example `dbt_bqp042_prod` |
| Location | Same as the source and output datasets; use the provisioned connection |

2. Inspect the template's staging SQL and source YAML. This exercise uses `stg_financial_services__loan`. The source definition reads `source_database` and `source_schema` from project variables. Set both explicitly; do not rely on a destination-specific default.
3. Update `dbt_project.yml` on your own branch: keep the project name `financial_services_lab`, profile and paths, set both source variables, add `review_threshold: 15000`, retain the staging view configuration and add marts as views. Replace the placeholders with your actual identifiers. Preserve any additional instructor-provided configuration.

4. Keep the existing staging models and source definitions. They preserve the template’s source-row behavior and rename `_fivetran_synced` to `synced_at`; this exercise does not introduce a soft-delete policy.

The full minimal `dbt_project.yml` is in the answer appendix. Try the edits first, then compare.

5. Build only the staging inputs needed for this exercise:

```bash
dbt build --select stg_financial_services__loan
```

6. Resolve failures before continuing. Confirm the built staging relations are in your development target plus `_staging`. The source can be empty; a successful zero-row build is not evidence of a populated pipeline. Check the Fivetran sync status and discuss empty inputs with the instructor.

## Part 3: Write the marts and tests yourself

**Business question:** Loan amount review. Loan amounts are not outstanding balances. No lending eligibility or credit-risk decision is made.

Use these files:

| File | Purpose |
|---|---|
| `models/marts/fct_financial_services_observations.sql` | Row-level fact view |
| `models/marts/mart_financial_services_review_summary.sql` | Business-facing mart view |
| `models/marts/_financial_services__addon_models.yml` | Model descriptions and generic data tests |
| `tests/assert_financial_services_addon.sql` | Singular reconciliation test; returns rows only when an invariant fails |

**Model requirements**

- Build `fct_financial_services_observations` at one row per `loan.id`. Use `ref('stg_financial_services__loan')` and select the source key as `observation_id`.
- Derive `group_key` from `grade`: use `missing:` for null and `value:` plus its string value otherwise. Keep `group_name` as the display string, using `Unknown` for null. Distinguish a real value named Unknown from a missing value through the key.
- Calculate `metric_value` from `loan_amnt` using safe numeric conversion. Convert arithmetic inputs before calculating. Division by zero must produce null. This measures the original loan amount.
- Preserve `synced_at`. Set `requires_review` to `metric_value >= review_threshold`, initially 15000. Missing metrics and their flags stay null; never silently turn them into zero or false.
- Build `mart_financial_services_review_summary` from the fact at one row per `group_key`. Include group name, total observations, measured observations, missing metric count, review count, unweighted mean metric and review share. Divide review count by measured observations with a zero-safe denominator and decimal arithmetic.
- Test unique/non-null observation IDs and group keys, observation-to-staging key relationships and non-null counts. Add a singular test for source/fact row preservation, summary row-count reconciliation, measured + missing = total and review count no greater than measured count. Legitimately missing metrics must remain visible, not fail a blanket non-null test.

1. Create `models/marts` if needed. Write the fact SQL first: start with the staging input(s), give the output columns explicit names and implement the model requirements above. Use CTEs to separate normalization, joins or ranking from derived business flags.
2. Add `{{ config(materialized='view') }}` to the fact. Use `ref()` for staging dependencies; never hard-code your development schema in SQL.
3. Write the second mart from `ref()` of your fact. Keep its documented grain. Review null handling and arithmetic before moving on.
4. Create the YAML file with `version: 2`, a `models` list, descriptions, column definitions and `data_tests`. Add `unique`/`not_null` only where the contract requires them. For relationships, use `arguments: {to: ref('model_name'), field: key_column}` with the actual model and field. Do not duplicate the existing staging model declarations.
5. Write the singular test as one or more SELECTs joined with `union all`. Each SELECT returns a failure label only when an invariant is violated. A passing singular test returns zero rows.
6. Inspect **Lineage**. Try your own implementation before comparing it with the **Full answer** at the end. The appendix includes every new file and the complete minimal project configuration.

> **BigQuery syntax:** A reconciliation SELECT that uses `WHERE` must include a `FROM` clause. For a scalar comparison with no input relation, use `from (select 1) as singleton` before `where`. Keep existing model-backed `FROM` clauses.

## Part 4: Build and validate

1. Inspect the resolved targets before executing:

```bash
dbt ls --select +mart_financial_services_review_summary --resource-type model --output json --output-keys name database schema alias
```

2. Confirm every selected model targets `<DEV_DATABASE>`. The staging views use `<DEV_SCHEMA>_staging` and the two marts use `<DEV_SCHEMA>`. Correct your own credentials if the target differs.
3. Build the mart, its ancestors and associated tests:

```bash
dbt build --select +mart_financial_services_review_summary
```

4. Require a successful final result. Read failing test output and repair the model or assumptions; do not remove tests merely to get a green run. Verify the singular reconciliation test was selected. If it was not, run it explicitly:

```bash
dbt test --select assert_financial_services_addon
```

5. Run a bounded aggregate check in dbt Studio and record the result as your baseline:

```bash
dbt show --inline "select coalesce(sum(observation_count), 0) as observations, coalesce(sum(measured_count), 0) as measured, coalesce(sum(missing_metric_count), 0) as missing, coalesce(sum(review_count), 0) as flagged from {{ ref('mart_financial_services_review_summary') }}" --limit 1
```

These are aggregate validation results through the dbt connection, not a request to open the destination console. If a row preview is useful, the instructor demonstrates it using their own access. Do not assume fixed row counts: each lab's source/filter/sync state can differ.

> **Checkpoint:** The selected models and tests pass, the grain/reconciliation checks hold, and you have recorded baseline aggregates.

## Part 5: Compare dbt State decisions

1. Under **Your profile → Credentials → this project → User development settings**, enable **dbt State**, save and restart Studio. Preserve your target and credentials.
2. Run the exact build command from Part 4. Read the execution logs to confirm the runtime and that State is enabled. Record models built/reused and tests executed/reused separately.
3. Without changing code, variables or source data, run the same command again. Compare State's decisions; elapsed time alone is not proof of reuse.
4. Temporarily change `review_threshold` in your project from `15000` to `25000`. This changes the illustrative review rule without updating source data.
5. Run the same focused build and aggregate query again. Record the changed flag count and State decisions. Row counts must remain unchanged; a flag count can stay the same if no values cross the threshold.
6. Restore `review_threshold: 15000`, rebuild and rerun the aggregate check. Confirm the baseline business results return. Verify the temporary value is absent from the diff before committing.

| Run | Models built | Models reused | Tests executed | Tests reused | Aggregate result |
|---|---|---|---|---|---|
| Baseline with State | | | | | |
| Unchanged rerun | | | | | |
| Changed threshold | | | | | |
| Restored threshold | | | | | |

Views can expose changed upstream values without rebuilding their definitions. Reused test results were not freshly executed. This exercise changes code on a stable source; it does not prove live freshness detection. If a source sync changes the input during comparison, establish a new baseline and repeat with a stable source.

## Part 6: Create an isolated deployment environment and job

### Prepare the branch and target

1. Create `macros/log_financial_services_review_totals.sql` using the helper in the **Full answer**. Run `dbt run-operation log_financial_services_review_totals` in Studio and confirm its totals match your restored baseline. This helper logs the totals and effective target through the current connection.
2. Review `dbt_project.yml`, the four model/test files and the new macro. Commit and push only those changes to your personal branch. Exclude unrelated changes and all credentials.
3. Use **Go to repository** to verify the remote branch has the final SQL, YAML, test and macro files and the restored threshold. Record the exact branch and commit SHA as `<DEPLOY_BRANCH>` and `<DEPLOY_COMMIT>`.
4. Confirm `<DEPLOY_SCHEMA>` is separate from your Studio target and other attendees' targets. Ask the instructor to provide a dedicated deployment connection profile that can read the source and create the deployment marts and staging views. Personal Studio credentials do not automatically configure deployment credentials.

### Configure the environment

1. Select **Orchestration → Environments → Create environment**. Name it `financial_services_deploy_<firstname>_<lastname>`, select **Deployment** and explicitly choose **General** as the deployment type. Do not change shared Staging/Production designations.
2. Select **v2 Stable**, enable **dbt State**, and choose **Only run on a custom branch** with `<DEPLOY_BRANCH>`.
3. Assign the instructor-approved connection profile. Verify its effective project, dataset and location match your recorded deployment target. Use the provisioned BigQuery runner connection; do not replace its service-account key. If the profile or permissions are unavailable, have the instructor resolve them before proceeding.
4. Save, reopen the environment and confirm General type, branch, runtime, State and profile. A unique name alone does not isolate database writes; the profile target must also be yours.

### Create and run the job

1. From your environment, choose **Create job → Deploy job**, name it `financial_services_build_<firstname>_<lastname>` and confirm the environment is yours.
2. Replace the unrestricted default build command with:

```bash
dbt build --select +mart_financial_services_review_summary
```

3. Add `dbt run-operation log_financial_services_review_totals` as the second job command. The deployment job form rejected the inline `dbt show` command during the live BigQuery walkthrough; this macro logs the same aggregate check through the job's own target and credentials.
4. Inherit the runtime and State settings from the environment. Keep **Compare changes against** at **No deferral** for this exercise. Keep source-freshness execution off. Turn off scheduled and job-completion triggers. Save and recheck the job settings.
5. Click **Run now** once and wait for the final result. Confirm the Git commit is `<DEPLOY_COMMIT>`, the logs show the expected runtime/State setting, the source variables resolve correctly and output relations use your deployment schema/dataset (plus `_staging` for staging).
6. Compare the aggregate output with the restored Studio baseline. Record the run URL, final status, executed/reused counts and comparison result. If results differ, investigate source changes, target/profile or remote commit before rerunning.
7. Leave automatic triggers off. Do not merge your branch or alter other learners' resources as part of this exercise.

> **Checkpoint:** A manually triggered job ran your verified Git commit in your isolated deployment target, and its tests and aggregate output support the result.

## What you did

You configured a personal dbt Platform target, turned source observations into tested marts, compared State decisions under a controlled change, restored the baseline and verified a deployment job. You can explain the grain, null handling and limits of the business rule.

## References

- [Inspected industry template](https://github.com/dbt-Hands-on-Labs/dbt-financial-services-lab/tree/f4f159d67675b4b1485369dcc4f5c21af766a18f)
- [Reference Platform workflow, Parts 5–7](../../transformations/fivetran_dbt_snowflake_hands_on_lab.md#part-5-accessing-dbt-platform)
- [dbt State](https://docs.getdbt.com/docs/deploy/dbt-state-about)
- [Deployment environments](https://docs.getdbt.com/docs/deploy/deploy-environments)

## Full answer

Compare this implementation with yours after attempting the exercise. Substitute your source identifiers in the project file. These are complete minimal files for the standard template; preserve extra instructor-provided project settings. Existing staging and source files remain in place. Different SQL can be correct if it meets the same grain, null-handling and reconciliation contracts.

### `dbt_project.yml`

```yaml
name: financial_services_lab
version: '1.0.0'
profile: default
require-dbt-version: [">=1.9.0", "<3.0.0"]
model-paths: [models]
test-paths: [tests]
macro-paths: [macros]
vars:
  source_database: '<SOURCE_DATABASE>'
  source_schema: '<SOURCE_SCHEMA>'
  review_threshold: 15000
models:
  financial_services_lab:
    staging:
      +materialized: view
      +schema: staging
    marts:
      +materialized: view
```

### `models/marts/fct_financial_services_observations.sql`

```sql
{{ config(materialized='view') }}
with normalized as (
    select
        id as observation_id,
        case when grade is null then 'missing:'
             else concat('value:', cast(grade as string)) end as group_key,
        coalesce(cast(grade as string), 'Unknown') as group_name,
        safe_cast(loan_amnt as float64) as metric_value,
        synced_at
    from {{ ref('stg_financial_services__loan') }}
)
select *, metric_value >= {{ var('review_threshold', 15000) }} as requires_review
from normalized
```

### `models/marts/mart_financial_services_review_summary.sql`

```sql
{{ config(materialized='view') }}
select
    group_key,
    min(group_name) as group_name,
    count(*) as observation_count,
    count(metric_value) as measured_count,
    sum(case when metric_value is null then 1 else 0 end) as missing_metric_count,
    sum(case when requires_review then 1 else 0 end) as review_count,
    avg(metric_value) as mean_metric_value,
    1.0 * sum(case when requires_review then 1 else 0 end)
        / nullif(count(metric_value), 0) as review_share
from {{ ref('fct_financial_services_observations') }}
group by group_key
```

### `models/marts/_financial_services__addon_models.yml`

```yaml
version: 2
models:
  - name: fct_financial_services_observations
    description: "One row per loan.id. Metric: original loan amount. Missing metrics and unknown review flags are retained."
    columns:
      - name: observation_id
        data_tests:
          - unique
          - not_null
          - relationships:
              arguments:
                to: ref('stg_financial_services__loan')
                field: id
      - name: group_key
        data_tests:
          - not_null
  - name: mart_financial_services_review_summary
    description: "One row per group_key derived from grade. Counts are observations; review_share excludes missing metrics."
    columns:
      - name: group_key
        data_tests:
          - unique
          - not_null
      - name: observation_count
        data_tests:
          - not_null
      - name: measured_count
        data_tests:
          - not_null
      - name: missing_metric_count
        data_tests:
          - not_null
      - name: review_count
        data_tests:
          - not_null
```

### `tests/assert_financial_services_addon.sql`

```sql
-- Fail if rows disappear, multiply, or are counted incorrectly in the summary.
select 'source_grain' as failure
from (select 1) as singleton
where (select count(*) from {{ ref('fct_financial_services_observations') }}) <>
      (select count(*) from {{ ref('stg_financial_services__loan') }})
union all
select 'summary_reconciliation' as failure
from (select 1) as singleton
where (select count(*) from {{ ref('fct_financial_services_observations') }}) <>
      (select coalesce(sum(observation_count), 0) from {{ ref('mart_financial_services_review_summary') }})
union all
select 'metric_reconciliation' as failure
from {{ ref('mart_financial_services_review_summary') }}
where measured_count + missing_metric_count <> observation_count
   or review_count > measured_count
   or review_count < 0
```

### `macros/log_financial_services_review_totals.sql`

```sql
{% macro log_financial_services_review_totals() %}
  {% set sql %}
    select coalesce(sum(observation_count), 0) as observations, coalesce(sum(measured_count), 0) as measured, coalesce(sum(missing_metric_count), 0) as missing, coalesce(sum(review_count), 0) as flagged from {{ ref('mart_financial_services_review_summary') }}
  {% endset %}
  {% if execute %}
    {% set results = run_query(sql) %}
    {% for row in results.rows %}
      {{ log('Financial Services totals: observations=' ~ row[0] ~ ', measured=' ~ row[1] ~ ', missing=' ~ row[2] ~ ', flagged=' ~ row[3], info=true) }}
    {% endfor %}
    {{ log('Deployment target: ' ~ target.database ~ '.' ~ target.schema, info=true) }}
  {% endif %}
{% endmacro %}
```
