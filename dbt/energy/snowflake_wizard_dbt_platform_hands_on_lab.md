# dbt Platform add-on: Energy — Snowflake with dbt Wizard

*October 2026. This worksheet starts after the source lab's initial sync and follows the dbt Platform portion of the Fivetran + dbt lab. It ends with a deployment job; no Cortex or destination-console exercise is included.*

## Objective

Build **Crude price monitoring** from the synced Energy data. Configure your personal development target, create two tested marts with dbt Wizard, compare dbt State decisions, then run a job from your own branch in an isolated deployment target.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `commodity_prices` into your own destination. Use **Fivetran → Connection → Schema** to confirm names and selected columns.
- Accept the invitation to the provisioned dbt Platform project, with a Developer seat and Git access to a generated copy of [dbt-energy-lab](https://github.com/dbt-Hands-on-Labs/dbt-energy-lab). The instructor provides project-scoped environment/job creation permissions and an approved deployment connection profile.
- The instructor enables dbt v2 and State for the session and makes dbt Wizard available. If a required feature is unavailable, resolve it with the instructor; do not start a trial or change shared account settings.
- The connection must read the landing data and create models in your development/deployment targets and their `_staging` schemas/datasets. Credentials belong only in the provisioned credential forms, never in SQL, YAML, prompts or Git.
- All destination data previews are instructor-led. No Snowflake console sign-in is required for this add-on. Perform the validation commands below in **dbt Studio**. Local DuckDB previews from the SDK lab remain hands-on.

## Part 1: Access dbt Platform and isolate your work

1. Accept your dbt Platform lab invitation, complete registration and select the project named `LAB_DBT...` assigned by your instructor. Do not create a trial or use another lab's project.
2. Open **Studio**. If disconnected, open **Status → View Credentials**, or **Your profile → Credentials → your project**.
3. In **Connection details**, set **Database** to your provisioned development database ending in `_DEV`. Use **Key pair** authentication and enter your assigned username and private key directly from the 1Password credential link. Leave the passphrase blank unless one was explicitly provided.
4. Set **Schema** to `dbt_<firstname>_<lastname>` and preserve the instructor-provided role and warehouse. Leave the target name at its default. Test and save; do not target the landing database or shared `LABSTUDIOS` database.
5. If permissions or credentials are missing, have the instructor repair your provisioned access before continuing.
6. Restart Studio if needed and confirm it is connected. Create your personal Git branch `<firstname>_<lastname>_dbt_addon`. Work on that branch throughout the exercise.
7. Open **dbt version → Personal version override**, select the instructor-approved **v2 Stable** runtime, and save/restart. Do not edit the shared Development environment. Confirm the actual runtime in command logs; UI wording can change.

> **Checkpoint:** Studio connects to your assigned Snowflake target, your personal branch is selected and the runtime is confirmed.

## Part 2: Configure and inspect the source

1. Record these identifiers. Copy the actual names from your credentials and Fivetran schema configuration; do not infer them from another attendee's lab.

| Placeholder | Your value |
|---|---|
| `<SOURCE_DATABASE>` | Your provisioned database ending in `_LANDING` |
| `<SOURCE_SCHEMA>` | The actual Fivetran schema, including your connection prefix and source-schema suffix |
| `<DEV_DATABASE>` | Your provisioned database ending in `_DEV` |
| `<DEV_SCHEMA>` | Your personal schema, for example `DBT_JANE_DOE` |
| `<DEPLOY_SCHEMA>` | A separate instructor-approved schema, for example `DBT_JANE_DOE_DEPLOY` |
| Warehouse / role | Values from your assigned lab credentials |

2. Inspect the template's staging SQL and source YAML. This exercise uses `stg_energy__commodity_prices`. The source definition reads `source_database` and `source_schema` from project variables. Set both explicitly; do not rely on a destination-specific default.
3. Update `dbt_project.yml` on your own branch: keep the project name `energy_lab`, profile and paths, set both source variables, add `review_threshold: 70`, retain the staging view configuration and add marts as views. Replace the placeholders with your actual identifiers. Preserve any additional instructor-provided configuration.

4. Keep the existing staging models and source definitions. They preserve the template’s source-row behavior and rename `_fivetran_synced` to `synced_at`; this exercise does not introduce a soft-delete policy.

Use this complete minimal configuration as a reference for the standard template; preserve extra settings if your instructor has customized the project:

```yaml
name: energy_lab
version: '1.0.0'
profile: default
require-dbt-version: [">=1.9.0", "<3.0.0"]
model-paths: [models]
test-paths: [tests]
macro-paths: [macros]
vars:
  source_database: '<SOURCE_DATABASE>'
  source_schema: '<SOURCE_SCHEMA>'
  review_threshold: 70
models:
  energy_lab:
    staging:
      +materialized: view
      +schema: staging
    marts:
      +materialized: view
```

5. Build only the staging inputs needed for this exercise:

```bash
dbt build --select stg_energy__commodity_prices
```

6. Resolve failures before continuing. Confirm the built staging relations are in your development target plus `_staging`. The source can be empty; a successful zero-row build is not evidence of a populated pipeline. Check the Fivetran sync status and discuss empty inputs with the instructor.

## Part 3: Build the marts with dbt Wizard

**Business question:** Crude price monitoring. Missing quotes remain unknown. These historical quotes are not a price forecast.

Use these files:

| File | Purpose |
|---|---|
| `models/marts/fct_energy_observations.sql` | Row-level fact view |
| `models/marts/mart_energy_review_summary.sql` | Business-facing mart view |
| `models/marts/_energy__addon_models.yml` | Model descriptions and generic data tests |
| `tests/assert_energy_addon.sql` | Singular reconciliation test; returns rows only when an invariant fails |

**Model requirements**

- Build `fct_energy_observations` at one row per `commodity_prices.id`. Use `ref('stg_energy__commodity_prices')` and select the source key as `observation_id`.
- Derive `group_key` from `'WTI crude'`: use `missing:` for null and `value:` plus its string value otherwise. Keep `group_name` as the display string, using `Unknown` for null. Distinguish a real value named Unknown from a missing value through the key.
- Calculate `metric_value` from `wti_crude` using safe numeric conversion. Convert arithmetic inputs before calculating. Division by zero must produce null. This measures WTI crude quote in source units.
- Preserve `synced_at`. Set `requires_review` to `metric_value >= review_threshold`, initially 70. Missing metrics and their flags stay null; never silently turn them into zero or false.
- Build `mart_energy_review_summary` from the fact at one row per `group_key`. Include group name, total observations, measured observations, missing metric count, review count, unweighted mean metric and review share. Divide review count by measured observations with a zero-safe denominator and decimal arithmetic.
- Test unique/non-null observation IDs and group keys, observation-to-staging key relationships and non-null counts. Add a singular test for source/fact row preservation, summary row-count reconciliation, measured + missing = total and review count no greater than measured count. Legitimately missing metrics must remain visible, not fail a blanket non-null test.

1. Open **dbt Wizard** in Studio and keep **Ask for approval** enabled.
2. Paste the model requirements above followed by this prompt:

```text
Inspect the existing energy staging models and source definitions first.
Create only these four files:
models/marts/fct_energy_observations.sql
models/marts/mart_energy_review_summary.sql
models/marts/_energy__addon_models.yml
tests/assert_energy_addon.sql

Use Snowflake SQL and ref() dependencies, view materializations and built-in dbt tests.
Follow the Model requirements immediately above (paste them with this prompt).
Read review_threshold from dbt_project.yml with var('review_threshold').
Preserve existing staging, sources, project configuration and packages.
Document grains, units, null handling and the limits of the review rule.
Propose the files and explain their logic before editing. Do not run commands,
change settings, commit or push. Do not request credentials or warehouse-console access.
```

3. Review the proposed joins, grain, expressions, tests and file paths. Ask Wizard to correct discrepancies before approving edits.
4. Approve only the four exercise files. Inspect the diff and confirm the earlier project-variable changes are still yours. Keep unrelated files unchanged.
5. Inspect **Lineage** for both marts. Explain the dependencies before running commands.

## Part 4: Build and validate

1. Inspect the resolved targets before executing:

```bash
dbt ls --select +mart_energy_review_summary --resource-type model --output json --output-keys name database schema alias
```

2. Confirm every selected model targets `<DEV_DATABASE>`. The staging views use `<DEV_SCHEMA>_staging` and the two marts use `<DEV_SCHEMA>` (Snowflake may display unquoted names in uppercase). Correct your own credentials if the target differs.
3. Build the mart, its ancestors and associated tests:

```bash
dbt build --select +mart_energy_review_summary
```

4. Require a successful final result. Read failing test output and repair the model or assumptions; do not remove tests merely to get a green run. Verify the singular reconciliation test was selected. If it was not, run it explicitly:

```bash
dbt test --select assert_energy_addon
```

5. Run a bounded aggregate check in dbt Studio and record the result as your baseline:

```bash
dbt show --inline "select coalesce(sum(observation_count), 0) as observations, coalesce(sum(measured_count), 0) as measured, coalesce(sum(missing_metric_count), 0) as missing, coalesce(sum(review_count), 0) as flagged from {{ ref('mart_energy_review_summary') }}" --limit 1
```

These are aggregate validation results through the dbt connection, not a request to open the destination console. If a row preview is useful, the instructor demonstrates it using their own access. Do not assume fixed row counts: each lab's source/filter/sync state can differ.

> **Checkpoint:** The selected models and tests pass, the grain/reconciliation checks hold, and you have recorded baseline aggregates.

## Part 5: Compare dbt State decisions

1. Under **Your profile → Credentials → this project → User development settings**, enable **dbt State**, save and restart Studio. Preserve your target and credentials.
2. Run the exact build command from Part 4. Read the execution logs to confirm the runtime and that State is enabled. Record models built/reused and tests executed/reused separately.
3. Without changing code, variables or source data, run the same command again. Compare State's decisions; elapsed time alone is not proof of reuse.
4. Temporarily change `review_threshold` in your project from `70` to `100`. This changes the illustrative review rule without updating source data.
5. Run the same focused build and aggregate query again. Record the changed flag count and State decisions. Row counts must remain unchanged; a flag count can stay the same if no values cross the threshold.
6. Restore `review_threshold: 70`, rebuild and rerun the aggregate check. Confirm the baseline business results return. Verify the temporary value is absent from the diff before committing.

| Run | Models built | Models reused | Tests executed | Tests reused | Aggregate result |
|---|---|---|---|---|---|
| Baseline with State | | | | | |
| Unchanged rerun | | | | | |
| Changed threshold | | | | | |
| Restored threshold | | | | | |

Views can expose changed upstream values without rebuilding their definitions. Reused test results were not freshly executed. This exercise changes code on a stable source; it does not prove live freshness detection. If a source sync changes the input during comparison, establish a new baseline and repeat with a stable source.

## Part 6: Create an isolated deployment environment and job

### Prepare the branch and target

1. Review `dbt_project.yml` and the four new exercise files. Commit and push only those changes to your personal branch. Exclude unrelated changes and all credentials.
2. Use **Go to repository** to verify the remote branch has the final SQL, YAML and test files and the restored threshold. Record the exact branch and commit SHA as `<DEPLOY_BRANCH>` and `<DEPLOY_COMMIT>`.
3. Confirm `<DEPLOY_SCHEMA>` is separate from your Studio target and other attendees' targets. Ask the instructor to provide a dedicated deployment connection profile that can read the source and create the deployment marts and staging views. Personal Studio credentials do not automatically configure deployment credentials.

### Configure the environment

1. Select **Orchestration → Environments → Create environment**. Name it `energy_deploy_<firstname>_<lastname>`, select **Deployment** and explicitly choose **General** as the deployment type. Do not change shared Staging/Production designations.
2. Select **v2 Stable**, enable **dbt State**, and choose **Only run on a custom branch** with `<DEPLOY_BRANCH>`.
3. Assign the instructor-approved connection profile. Verify its effective database, schema, role and warehouse match your recorded deployment target. If the instructor authorizes a new dedicated profile, select the existing lab connection, use Key pair authentication from your 1Password credentials, set the deployment schema and development database override, test and save it, then assign it to this environment. If the profile or permissions are unavailable, have the instructor resolve them before proceeding.
4. Save, reopen the environment and confirm General type, branch, runtime, State and profile. A unique name alone does not isolate database writes; the profile target must also be yours.

### Create and run the job

1. From your environment choose **Create job → Deploy job**, name it `energy_build_<firstname>_<lastname>` and confirm the environment is yours.
2. Replace the unrestricted default build command with:

```bash
dbt build --select +mart_energy_review_summary
```

3. Add the aggregate command from Part 4 as a second job command. This verifies deployment results through the job's own target and credentials.
4. Inherit the runtime and State settings from the environment. Keep **Compare changes against** at **No deferral** for this exercise. Keep source-freshness execution off. Turn off scheduled and job-completion triggers. Save and recheck the job settings.
5. Click **Run now** once and wait for the final result. Confirm the Git commit is `<DEPLOY_COMMIT>`, the logs show the expected runtime/State setting, the source variables resolve correctly and output relations use your deployment schema/dataset (plus `_staging` for staging).
6. Compare the aggregate output with the restored Studio baseline. Record the run URL, final status, executed/reused counts and comparison result. If results differ, investigate source changes, target/profile or remote commit before rerunning.
7. Leave automatic triggers off. Do not merge your branch or alter other learners' resources as part of this exercise.

> **Checkpoint:** A manually triggered job ran your verified Git commit in your isolated deployment target, and its tests and aggregate output support the result.

## What you did

You configured a personal dbt Platform target, turned source observations into tested marts, compared State decisions under a controlled change, restored the baseline and verified a deployment job. You can explain the grain, null handling and limits of the business rule.

## References

- [Inspected industry template](https://github.com/dbt-Hands-on-Labs/dbt-energy-lab/tree/c2b9a156a383503f41808c97c5b8b36188c101d7)
- [Reference Platform workflow, Parts 5–7](../../transformations/fivetran_dbt_snowflake_hands_on_lab.md#part-5-accessing-dbt-platform)
- [dbt State](https://docs.getdbt.com/docs/deploy/dbt-state-about)
- [Deployment environments](https://docs.getdbt.com/docs/deploy/deploy-environments)
- [dbt Wizard in Studio](https://docs.getdbt.com/docs/platform/wizard-platform)
