# dbt v1 to v2 migration lab: Utilities — BigQuery

## Objective

Create a meter consumption review project that builds on dbt v1 but contains legacy configuration that dbt v2 rejects. Diagnose the upgrade with **dbt Wizard**, preserve the business results, observe **dbt State**, and finish with a manual job in your own **Production** environment.

The migration is a configuration and test-syntax exercise, not a SQL dialect conversion. You will keep the same BigQuery destination, source data, model grain and business rule throughout.

## Requirements

- A web browser and access to your assigned dbt Platform project.
- Your lab credentials from the instructor's 1Password link.
- Complete the source-data sync before starting. Your synced data should include `utl_records`.

LabStudios provides your dbt project, staging models, Development and Production environments, and their databases or datasets. dbt Wizard, dbt v2 and dbt State are already available. You do not need to create these resources.

Allow about 90–120 minutes. Keep credentials in the credential forms, never in project files or Wizard prompts.

## Lab checkpoints

| Stage | Evidence |
|---|---|
| v1 | Legacy starter builds, tests pass, baseline totals recorded |
| v2 before repair | Actual compatibility diagnostic recorded from the same files |
| Wizard migration | Reviewed diff explains each fix; SQL and business rule preserved |
| v2 after repair | Build/tests pass and totals match v1 on unchanged input |
| State | Unchanged, changed-threshold and restored runs compared |
| Production | Correct remote commit, runtime, destination, successful tests and totals |

## Part 1: Access dbt Platform and isolate your work

1. Accept your dbt Platform lab invitation, complete registration and select the project named `LAB_DBT...` assigned by your instructor. Do not create a trial or use another lab's project.
2. Open **Your profile → Credentials → your assigned project**. Alternatively, from Studio's **Status: Disconnected** menu, open **Edit environment**, find **Connection → User credentials**, and follow the **profile page** link. Use that link only to reach your personal credentials; do not edit the shared environment or connection. Personal credentials apply to your dbt login, so each participant must use their own login.
3. Under **Credentials for project**, click **Edit**. Verify that the **Project ID** is `fivetran-hands-on-lab` and **Service Account JSON** is selected. LabStudios configures the runner service account on the existing BigQuery connection; do not upload your Fivetran destination key. If the project or authentication method differs, ask the instructor to check the provisioned connection.
4. In **Dataset**, enter the **BigQuery dbt dataset** from your assigned 1Password lab item, such as `dbt_bqp042`. Use your participant-specific dbt dataset, not the Fivetran source dataset or another attendee's dataset. Leave **Target name** and the other settings unchanged.
5. Click **Test connection**. Wait for setup and connection testing to finish, and confirm that **Complete** is green. Then click **Save** and wait for the green **Saved** confirmation.
6. Return to **Studio**. Its status may show **Parsing** and **Checking connection** while it starts. Wait until the status is green and **Connected** before continuing. If testing fails or Studio remains disconnected after saving, ask the instructor to troubleshoot the provisioned connection. Attendees do not need BigQuery Studio, Google Cloud Console or IAM access.
7. Select the **`lab-work`** branch (**Change branch → `lab-work`**) and work on it throughout the exercise. If your instructor gives you a different branch, such as a personal branch `<firstname>_<lastname>_v2_migration`, use that one instead.
8. Open **dbt version → Personal version override → Set override**, select **v1 Latest**, and save/restart. Do not edit the shared Development environment. Confirm the actual runtime in command logs; UI wording can change.

> **Checkpoint:** Studio connects to your assigned BigQuery target, your working branch (`lab-work` unless your instructor said otherwise) is selected and the runtime is confirmed.

Under **Your profile → Credentials → this project → User development settings**, set your personal **dbt State** override to **Disabled** for the baseline. Save and restart Studio if prompted. Do not change the account-wide setting. Record the engine version from the first command log; a release-track label alone is not an exact version.

## Part 2: Configure and inspect the source

1. Record these identifiers. Copy the actual names from your credentials and Fivetran schema configuration; do not infer them from another attendee's lab.

| Placeholder | Your value |
|---|---|
| `<SOURCE_DATABASE>` | The BigQuery Project ID containing the synced source dataset |
| `<SOURCE_SCHEMA>` | The exact Fivetran dataset, including assigned prefix, your name and source-schema suffix |
| `<DEV_DATABASE>` | The BigQuery project configured in dbt |
| `<DEV_SCHEMA>` | Your assigned dataset, for example `dbt_bqp042` |
| `<PROD_SCHEMA>` | Your assigned BigQuery dbt Prod Dataset, for example `dbt_bqp042_prod` |
| Location | Same as the source and output datasets; use the provisioned connection |

2. In Studio, inspect the existing staging SQL and source YAML. This exercise uses `stg_utilities__utl_records`. The source definition reads `source_database` and `source_schema` from project variables. You will set both variables in Part 3.
3. Keep the existing staging models and source definitions. They preserve the template’s source-row behavior and rename `_fivetran_synced` to `synced_at`; this exercise does not introduce a soft-delete policy.

## Part 3: Create the legacy v1 project

**Business question:** Meter consumption review. Count meter observations, not unique customers; no anomaly or demand forecast is inferred.

Follow these steps in order on your working branch. Keep the following model contracts as you build and migrate the project:

- Build `fct_utilities_observations` at one row per `utl_records.record_id`. Use `ref('stg_utilities__utl_records')` and select the source key as `observation_id`.
- Derive `group_key` from `service_territory`: use `missing:` for null and `value:` plus its string value otherwise. Keep `group_name` as the display string, using `Unknown` for null. Distinguish a real value named Unknown from a missing value through the key.
- Calculate `metric_value` from `energy_consumption_kwh` using safe numeric conversion. Convert arithmetic inputs before calculating. Division by zero must produce null. This measures observed energy consumption in kWh.
- Preserve `synced_at`. Set `requires_review` to `metric_value >= review_threshold`, initially 50. Missing metrics and their flags stay null; never silently turn them into zero or false.
- Build `mart_utilities_review_summary` from the fact at one row per `group_key`. Include group name, total observations, measured observations, missing metric count, review count, unweighted mean metric and review share. Divide review count by measured observations with a zero-safe denominator and decimal arithmetic.
- Test unique/non-null observation IDs and group keys, observation-to-staging key relationships and non-null counts. Add a singular test for source/fact row preservation, summary row-count reconciliation, measured + missing = total and review count no greater than measured count. Legitimately missing metrics must remain visible, not fail a blanket non-null test.

### 3.1 Update the project configuration

Open `dbt_project.yml`. Replace the source placeholders with the values recorded in Part 2. Preserve any additional settings already in your project, and save. The project `version` and model YAML `version: 2` do not select the dbt engine.

**File:** `dbt_project.yml`

```yaml
name: utilities_lab
version: '1.0.0'
profile: default
require-dbt-version: [">=1.10.5", "<3.0.0"]
model-paths: [models]
test-paths: [tests]
macro-paths: [macros]
vars:
  source_database: '<SOURCE_DATABASE>'
  source_schema: '<SOURCE_SCHEMA>'
  review_threshold: 50
models:
  utilities_lab:
    staging:
      +materialized: view
      +schema: staging
    marts:
      +materialized: view
```

### 3.2 Create the fact model

Create `models/marts/fct_utilities_observations.sql` (and its folder if needed), paste the code below, and save the file.

**File:** `models/marts/fct_utilities_observations.sql`

```sql
{{ config(materialized='view') }}
with normalized as (
    select
        record_id as observation_id,
        case when service_territory is null then 'missing:'
             else concat('value:', cast(service_territory as string)) end as group_key,
        coalesce(cast(service_territory as string), 'Unknown') as group_name,
        safe_cast(energy_consumption_kwh as float64) as metric_value,
        synced_at
    from {{ ref('stg_utilities__utl_records') }}
)
select *, metric_value >= {{ var('review_threshold', 50) }} as requires_review
from normalized
```

### 3.3 Create the business summary model

Create `models/marts/mart_utilities_review_summary.sql` (and its folder if needed), paste the code below, and save the file.

**File:** `models/marts/mart_utilities_review_summary.sql`

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
from {{ ref('fct_utilities_observations') }}
group by group_key
```

### 3.4 Add descriptions and legacy tests

Create `models/marts/_utilities__addon_models.yml` (and its folder if needed), paste the code below, and save the file. The custom metadata and generic-test inputs intentionally use legacy syntax. Keep them unchanged for the v1 baseline; you will migrate them with Wizard in Part 5. The accepted-values test permits nulls; existing not-null tests remain in place where required by the model contract.

**File:** `models/marts/_utilities__addon_models.yml`

```yaml
version: 2
models:
  - name: fct_utilities_observations
    description: 'One row per utl_records.record_id. Metric: observed energy consumption in kWh. Missing metrics and unknown
      review flags are retained.'
    columns:
      - name: observation_id
        data_tests:
          - unique
          - not_null
          - relationships:
              to: ref('stg_utilities__utl_records')
              field: record_id
      - name: group_key
        data_tests:
          - not_null
      - name: requires_review
        data_tests:
          - accepted_values:
              values:
                - true
                - false
              quote: false
    config:
      migration_owner: utilities_analytics
  - name: mart_utilities_review_summary
    description: One row per group_key derived from service_territory. Counts are observations; review_share excludes missing
      metrics.
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

### 3.5 Add the reconciliation test

Create `tests/assert_utilities_addon.sql` (and its folder if needed), paste the code below, and save the file. This test returns failure rows when a reconciliation check fails. A passing result contains no rows.

**File:** `tests/assert_utilities_addon.sql`

```sql
-- Fail if rows disappear, multiply, or are counted incorrectly in the summary.
select 'source_grain' as failure
from (select 1) as singleton
where (select count(*) from {{ ref('fct_utilities_observations') }}) <>
      (select count(*) from {{ ref('stg_utilities__utl_records') }})
union all
select 'summary_reconciliation' as failure
from (select 1) as singleton
where (select count(*) from {{ ref('fct_utilities_observations') }}) <>
      (select coalesce(sum(observation_count), 0) from {{ ref('mart_utilities_review_summary') }})
union all
select 'metric_reconciliation' as failure
from {{ ref('mart_utilities_review_summary') }}
where measured_count + missing_metric_count <> observation_count
   or review_count > measured_count
   or review_count < 0
```

### 3.6 Add the totals logging macro

Create `macros/log_utilities_review_totals.sql` (and its folder if needed), paste the code below, and save the file. This macro logs the aggregate results and effective target for comparison between development and Production.

**File:** `macros/log_utilities_review_totals.sql`

```sql
{% macro log_utilities_review_totals() %}
  {% set sql %}
    select coalesce(sum(observation_count), 0) as observations, coalesce(sum(measured_count), 0) as measured, coalesce(sum(missing_metric_count), 0) as missing, coalesce(sum(review_count), 0) as flagged from {{ ref('mart_utilities_review_summary') }}
  {% endset %}
  {% if execute %}
    {% set results = run_query(sql) %}
    {% for row in results.rows %}
      {{ log('Utilities totals: observations=' ~ row[0] ~ ', measured=' ~ row[1] ~ ', missing=' ~ row[2] ~ ', flagged=' ~ row[3], info=true) }}
    {% endfor %}
    {{ log('Deployment target: ' ~ target.database ~ '.' ~ target.schema, info=true) }}
  {% endif %}
{% endmacro %}
```

### 3.7 Check your staging input and lineage

Build the staging inputs before the complete baseline:

```bash
dbt build --select stg_utilities__utl_records
```

Resolve staging failures before continuing and confirm the source sync is complete. A successful zero-row build does not demonstrate populated input. Inspect **Lineage**: staging → fact → mart. Review the grains and null handling above, save all files, and keep the legacy YAML unchanged until Part 5.

## Part 4: Prove the v1 baseline

1. Inspect the resolved targets before executing:

```bash
dbt ls --select +mart_utilities_review_summary --resource-type model --output json --output-keys name database schema alias
```

2. Confirm every selected model targets `<DEV_DATABASE>`. The staging views use `<DEV_SCHEMA>_staging` and the two marts use `<DEV_SCHEMA>`. Correct your own credentials if the target differs.
3. Build the mart, its ancestors and associated tests:

```bash
dbt build --select +mart_utilities_review_summary
```

4. Require a successful final result. Read failing test output and repair the model or assumptions; do not remove tests merely to get a green run. Verify the singular reconciliation test was selected. If it was not, run it explicitly:

```bash
dbt test --select assert_utilities_addon
```

5. Run a bounded aggregate check in dbt Studio and record the result as your baseline:

```bash
dbt show --inline "select coalesce(sum(observation_count), 0) as observations, coalesce(sum(measured_count), 0) as measured, coalesce(sum(missing_metric_count), 0) as missing, coalesce(sum(review_count), 0) as flagged from {{ ref('mart_utilities_review_summary') }}" --limit 1
```

These are aggregate validation results through the dbt connection, not a request to open the destination console. If a row preview is useful, the instructor demonstrates it using their own access. Do not assume fixed row counts: each lab's source/filter/sync state can differ.

6. Record the v1 runtime, branch, input row count, aggregate result and test outcome. Run the v1-only deprecation inventory:

```bash
dbt parse --no-partial-parse --show-all-deprecations
```

Keep the log. Expect deprecations for legacy test arguments and custom config metadata. These are intentionally retained until Part 5. Warning names and counts depend on the selected v1 build. Do not enable warnings-as-errors for this teaching baseline; do not suppress warnings either. If an account policy already promotes warnings to errors, have the instructor provide a suitable isolated lab project.

> **Checkpoint:** The **legacy files** built successfully on v1, their tests passed, populated source data was confirmed, and the baseline numbers and deprecation log were recorded. Do not switch to v2 before this checkpoint.

## Part 5: Reproduce the failure and migrate with dbt Wizard

1. Save the successful v1 state with **Commit and sync** on your lab branch. Record the baseline commit SHA. This is your comparison and recovery point; keep the baseline out of any scheduled job.
2. Use **dbt version → Personal version override → Set override** to select **v2 Stable**, save and restart Studio. Keep your personal State override **Disabled** so the engine migration is the only change.
3. Run:

```bash
dbt compile --select +mart_utilities_review_summary
```

4. Record the actual diagnostics and runtime. Look for errors about `config.migration_owner` and the top-level test arguments; error codes and wording may differ by runtime. The legacy generic-test arguments and custom config are intentional compatibility problems. A warehouse permission error, unavailable package or missing table is a separate setup problem. If the first error masks another, rerun after each repair. If all files compile unchanged, confirm the runtime and starter YAML; do not invent an error or call the exercise complete.
5. Open **dbt Wizard** and choose **Ask for approval** (the session may default to automatic edits). Paste:

```text
Help migrate this utilities project from dbt v1 to dbt v2. Use the
migrating-dbt-core-to-fusion skill if available. Inspect the actual compile
errors and the project files before proposing changes.

The v1 baseline built successfully. Preserve the documented fact and mart grains, stable
keys, null handling, review_threshold=50, view materializations, source
variables, SQL business logic and all existing test assertions.

Focus on models/marts/_utilities__addon_models.yml. Classify each error,
explain the smallest compatible fix, and show file diffs for review. Preserve
migration_owner metadata in the supported location; do not discard it. Fix
test argument structure without deleting tests or weakening severity.

Ask before each edit and command. Do not change credentials, runtime, State,
branches or environment settings. Do not commit, push or run Production jobs.
Compile after the edits and report any remaining blockers. If unrelated
staging/package issues appear, explain them separately before expanding scope.
```

6. Review and approve the proposed changes. Wizard's named migration skill may be unavailable in a particular account; the same scoped prompt can still guide an ordinary Wizard investigation. If Wizard itself is unavailable, ask the instructor to restore access rather than silently skipping the core exercise.
7. Inspect the diff yourself. The v2 runtime may also add `dbt_internal_packages/` to `.gitignore`; review that generated-directory exclusion separately from Wizard’s YAML repair. Inputs to `relationships` and `accepted_values` belong under `arguments`. Custom metadata belongs under `config.meta`. SQL, thresholds and assertions should not change. Compare against the **Migration answer** at the end only after attempting the migration.
8. Run the compile again. Resolve all remaining compatibility errors. If Wizard cannot resolve one, retain its diagnostic and ask the instructor; a Wizard summary alone does not establish success.

The job-list **Review job → Debug in Studio with dbt Wizard** entry point is an alternative when a successful job already exists. This fresh-project lab starts directly in Studio and does not require a prior Production run.

## Part 6: Prove v2 preserves the result

With State still disabled, run each command separately:

```bash
dbt build --select +mart_utilities_review_summary
dbt test --select assert_utilities_addon
dbt run-operation log_utilities_review_totals
```

Require a successful build and tests. Run the exact aggregate query from Part 4 again and compare every aggregate with the v1 baseline. Counts must match exactly on unchanged source data. Keep the same source variables, target, materializations and `review_threshold: 50`.

Inspect the accepted-values, key and reconciliation test results, not just the model status. A compile proves compatibility; a build plus data tests checks the actual warehouse result. The logging macro also records the effective target and will be reused in Production.

| Evidence | v1 baseline | v2 after migration |
|---|---|---|
| Exact engine version | | |
| Target database/project and schema/dataset | | |
| Fact and mart row counts | | |
| Aggregate results from Part 4 | | |
| Build and test status | | |
| Source changed during comparison? | | |

If totals differ, check input sync timing, variables, target, and the Wizard diff before accepting the migration. Do not edit the expected numbers or remove tests to hide a difference.

## Part 7: Observe dbt State

Use the same model selection for every comparison below. The changed-threshold run adds only a command-level variable override; the saved project remains at 50. This is the dbt State service, not a `state:modified` selector against a saved manifest.

1. Under **Your profile → Credentials → this project → User development settings**, enable **dbt State**, save and restart Studio. Preserve your target and credentials.
2. Run the exact build command from Part 6. Read the execution logs to confirm the runtime and that State is enabled. Record models built/reused and tests executed/reused separately.
3. Without changing code, variables or source data, run the same command again. Compare State's decisions; elapsed time alone is not proof of reuse.
4. Temporarily override the review threshold for one build. This changes the compiled rule without changing source data or your saved project:

```bash
dbt build --select +mart_utilities_review_summary --vars '{"review_threshold": 75}'
```

5. Run the aggregate query from Part 4 (or `dbt run-operation log_utilities_review_totals`) to read the changed views. Record the flag count and State decisions, including which nodes rebuilt and why. Row counts must remain unchanged; a flag count can stay the same if no values cross the threshold. The query reads the built relations; it does not rebuild them or reset the threshold.
6. Restore the baseline by running the build **without** the override, then log the totals:

```bash
dbt build --select +mart_utilities_review_summary
dbt run-operation log_utilities_review_totals
```

Confirm the v2 baseline business results return and the saved project still has `review_threshold: 50`. In Production, the run’s **Explain** tab provides the State decision breakdown.

| Run | Models built | Models reused | Tests executed | Tests reused | Aggregate result |
|---|---|---|---|---|---|
| Baseline with State | | | | | |
| Unchanged rerun | | | | | |
| Changed threshold | | | | | |
| Restored threshold | | | | | |

Studio may display successful State reuse as **Skip**. Check the successful overall status and State logs to distinguish reuse from a node skipped after an upstream failure. Views can expose changed upstream values without rebuilding their definitions. Reused test results were not freshly executed. This exercise changes code on a stable source; it does not prove live freshness detection. If a source sync changes the input during comparison, establish a new baseline and repeat with a stable source.

## Part 8: Run the migrated code in Production

1. Restore `review_threshold: 50` and require the restored State run and aggregate check to pass. Review all changed files, including the logging macro. Confirm only the intended compatibility fixes and exercise files remain.
2. **Commit and sync** to your lab branch. Open the remote repository and record the exact branch and commit SHA. GitHub-template labs normally use `lab-work`; dbt-managed repositories may publish to `main`. Use the actual branch the Production environment will run, not an assumed name. Do not merge to an organization's production branch.
3. Open **Orchestration → Environments → Production** in your assigned project. Confirm deployment type **Production**, **v2 Stable**, the remote branch and the provisioned deployment credential. Confirm its output target is your assigned BigQuery project and **BigQuery dbt Prod Dataset** (for example `dbt_bqp042_prod`); staging uses that dataset plus `_staging`. Source variables must still point at landing. Personal Studio credentials do not configure job credentials.
4. Confirm **dbt State** is enabled in the precreated Production environment. Keep its connection profile and deployment credentials unchanged.
5. Choose **Create job → Deploy job**. Name it `utilities_v2_migration_<firstname>_<lastname>`. Replace the default commands with these three, in order:

```bash
dbt build --select +mart_utilities_review_summary
dbt test --select assert_utilities_addon
dbt run-operation log_utilities_review_totals
```

6. Inherit runtime/State from the environment. Keep explicit deferral at **No deferral**; do not add `--state`/`state:modified` as a substitute for dbt State. Disable scheduled and job-completion triggers. Save and recheck the environment and commands.
7. Click **Run now**. Wait for the final status, then verify the remote commit SHA, v2 runtime, State status, effective output locations, and successful tests in the run details. State may reuse results; report executed and reused nodes separately.
8. Compare the logging macro's Production totals with the restored development baseline on the same source data. Investigate differences before calling the lab complete. Record the run URL, commit SHA, target and totals. Leave automatic triggers off.

> **Completion:** Your migrated code runs successfully from the remote Git branch in the dedicated Production environment, and its test results and totals support the migration. A successful Studio run alone does not satisfy this checkpoint.

## References

- [Upgrade to dbt v2](https://docs.getdbt.com/docs/dbt-versions/dbt-upgrade/upgrading-to-v2)
- [Generic-test arguments](https://docs.getdbt.com/docs/build/data-tests)
- [Deprecations and custom config metadata](https://docs.getdbt.com/reference/deprecations)
- [Wizard migration workflow](https://docs.getdbt.com/docs/dbt-ai/wizard-ide#dbt-v2-migration-workflow)
- [Enable State in Studio](https://docs.getdbt.com/docs/deploy/dbt-state-enable-studio)
- [How State makes reuse decisions](https://docs.getdbt.com/docs/deploy/dbt-state-about)

## Appendix: Migration answer — compare after using Wizard

In `models/marts/_utilities__addon_models.yml`, preserve all existing descriptions, columns and assertions. Move the custom metadata into `config.meta`:

```yaml
  - name: fct_utilities_observations
    config:
      meta:
        migration_owner: utilities_analytics
```

For `fct_utilities_observations.observation_id`, replace the existing `relationships` test entry with:

```yaml
          - relationships:
              arguments:
                to: ref('stg_utilities__utl_records')
                field: record_id
```

For `fct_utilities_observations.requires_review`, replace the existing `accepted_values` test entry with:

```yaml
          - accepted_values:
              arguments:
                values:
                  - true
                  - false
                quote: false
```

No SQL change is needed for these intentional defects. Keep every existing test and reconciliation assertion. Preserve `review_threshold: 50` and rerun Part 6 before enabling State.
