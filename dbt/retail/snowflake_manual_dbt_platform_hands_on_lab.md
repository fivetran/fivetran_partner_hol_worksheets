# dbt Platform add-on: Retail — Snowflake without dbt Wizard

*October 2026. This worksheet starts after the source lab's initial sync and follows the dbt Platform portion of the Fivetran + dbt lab. It ends with a deployment job; no Cortex or destination-console exercise is included.*

## Objective

Build **Retail inventory priorities** from the synced Retail data. Configure your personal development target, create two tested marts without dbt Wizard, compare dbt State decisions, then run a job from your own branch in an isolated deployment target.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `inv_weekly_data`, `inv_products`, `inv_stores` into your own destination. Use **Fivetran → Connection → Schema** to confirm names and selected columns.
- Accept the invitation to the provisioned dbt Platform project, with a Developer seat and Git access to a generated copy of [dbt-retail-lab](https://github.com/dbt-Hands-on-Labs/dbt-retail-lab). The instructor provides project-scoped environment/job creation permissions and an approved deployment connection profile.
- The instructor enables dbt v2 and State for the session. If a required feature is unavailable, resolve it with the instructor; do not start a trial or change shared account settings.
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

2. Inspect the template's staging SQL and source YAML. This exercise uses `stg_retail__inv_weekly_data`, `stg_retail__inv_products`, `stg_retail__inv_stores`. The source definition reads `source_database` and `source_schema` from project variables. Set both explicitly; do not rely on a destination-specific default.
3. Update `dbt_project.yml` on your own branch: keep the project name `retail_lab`, profile and paths, set both source variables, add `review_threshold: 0`, retain the staging view configuration and add marts as views. Replace the placeholders with your actual identifiers. Preserve any additional instructor-provided configuration.

4. Keep the existing staging models and source definitions. They preserve the template’s source-row behavior and rename `_fivetran_synced` to `synced_at`; this exercise does not introduce a soft-delete policy.

The full minimal `dbt_project.yml` is in the answer appendix. Try the edits first, then compare.

5. Build only the staging inputs needed for this exercise:

```bash
dbt build --select stg_retail__inv_weekly_data stg_retail__inv_products stg_retail__inv_stores
```

6. Resolve failures before continuing. Confirm the built staging relations are in your development target plus `_staging`. The source can be empty; a successful zero-row build is not evidence of a populated pipeline. Check the Fivetran sync status and discuss empty inputs with the instructor.

## Part 3: Write the marts and tests yourself

**Business question:** Retail inventory priorities. The latest observation per product/store is historical inventory, not a demand forecast.

Use these files:

| File | Purpose |
|---|---|
| `models/marts/fct_retail_inventory_weekly.sql` | Row-level fact view |
| `models/marts/mart_retail_inventory_priorities.sql` | Business-facing mart view |
| `models/marts/_retail__addon_models.yml` | Model descriptions and generic data tests |
| `tests/assert_retail_addon.sql` | Singular reconciliation test; returns rows only when an invariant fails |

**Model requirements**

- Build `fct_retail_inventory_weekly` at one row per `row_id`. Left join products on `product_id` and stores on `store_id`; preserve the weekly input count.
- Include product/store names, category, region, stock, sales, days of supply, synced timestamp, typed inventory date and anomaly flag. Use safe date/boolean conversions for the chosen destination. Add `review_threshold` (initially 0 days) to the product's lead time.
- Build `mart_retail_inventory_priorities` from the fact. Select the latest date per product/store, with `row_id` ascending as a deterministic tie-breaker. Use a null-aware structured encoding of both IDs before hashing the product/store key.
- Carry the relevant fact columns through and calculate `needs_replenishment` as days of supply less than adjusted lead time. Do not use today's date to filter historical data.
- Test unique/non-null weekly row IDs and latest keys, product/store relationships, typed dates/flags, latest-to-weekly row relationships and non-null replenishment inputs. Add a singular test for weekly row preservation, one latest row per pair and correct latest dates.

1. Create `models/marts` if needed. Write the fact SQL first: start with the staging input(s), give the output columns explicit names and implement the model requirements above. Use CTEs to separate normalization, joins or ranking from derived business flags.
2. Add `{{ config(materialized='view') }}` to the fact. Use `ref()` for staging dependencies; never hard-code your development schema in SQL.
3. Write the second mart from `ref()` of your fact. Keep its documented grain. Review null handling and arithmetic before moving on.
4. Create the YAML file with `version: 2`, a `models` list, descriptions, column definitions and `data_tests`. Add `unique`/`not_null` only where the contract requires them. For relationships, use `arguments: {to: ref('model_name'), field: key_column}` with the actual model and field. Do not duplicate the existing staging model declarations.
5. Write the singular test as one or more SELECTs joined with `union all`. Each SELECT returns a failure label only when an invariant is violated. A passing singular test returns zero rows.
6. Inspect **Lineage**. Try your own implementation before comparing it with the **Full answer** at the end. The appendix includes every new file and the complete minimal project configuration.

## Part 4: Build and validate

1. Inspect the resolved targets before executing:

```bash
dbt ls --select +mart_retail_inventory_priorities --resource-type model --output json --output-keys name database schema alias
```

2. Confirm every selected model targets `<DEV_DATABASE>`. The staging views use `<DEV_SCHEMA>_staging` and the two marts use `<DEV_SCHEMA>` (Snowflake may display unquoted names in uppercase). Correct your own credentials if the target differs.
3. Build the mart, its ancestors and associated tests:

```bash
dbt build --select +mart_retail_inventory_priorities
```

4. Require a successful final result. Read failing test output and repair the model or assumptions; do not remove tests merely to get a green run. Verify the singular reconciliation test was selected. If it was not, run it explicitly:

```bash
dbt test --select assert_retail_addon
```

5. Run a bounded aggregate check in dbt Studio and record the result as your baseline:

```bash
dbt show --inline "select count(*) as latest_rows, count(distinct product_store_key) as distinct_pairs, sum(case when needs_replenishment then 1 else 0 end) as replenishment_count, min(inventory_date) as first_observation, max(inventory_date) as last_observation from {{ ref('mart_retail_inventory_priorities') }}" --limit 1
```

These are aggregate validation results through the dbt connection, not a request to open the destination console. If a row preview is useful, the instructor demonstrates it using their own access. Do not assume fixed row counts: each lab's source/filter/sync state can differ.

> **Checkpoint:** The selected models and tests pass, the grain/reconciliation checks hold, and you have recorded baseline aggregates.

## Part 5: Compare dbt State decisions

1. Under **Your profile → Credentials → this project → User development settings**, enable **dbt State**, save and restart Studio. Preserve your target and credentials.
2. Run the exact build command from Part 4. Read the execution logs to confirm the runtime and that State is enabled. Record models built/reused and tests executed/reused separately.
3. Without changing code, variables or source data, run the same command again. Compare State's decisions; elapsed time alone is not proof of reuse.
4. Temporarily change `review_threshold` in your project from `0` to `7`. This adds seven days to replenishment lead time in the fact; it does not update source data.
5. Run the same focused build and aggregate query again. Record the changed flag count and State decisions. Row counts must remain unchanged; a flag count can stay the same if no values cross the threshold.
6. Restore `review_threshold: 0`, rebuild and rerun the aggregate check. Confirm the baseline business results return. Verify the temporary value is absent from the diff before committing.

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

1. Select **Orchestration → Environments → Create environment**. Name it `retail_deploy_<firstname>_<lastname>`, select **Deployment** and explicitly choose **General** as the deployment type. Do not change shared Staging/Production designations.
2. Select **v2 Stable**, enable **dbt State**, and choose **Only run on a custom branch** with `<DEPLOY_BRANCH>`.
3. Assign the instructor-approved connection profile. Verify its effective database, schema, role and warehouse match your recorded deployment target. If the instructor authorizes a new dedicated profile, select the existing lab connection, use Key pair authentication from your 1Password credentials, set the deployment schema and development database override, test and save it, then assign it to this environment. If the profile or permissions are unavailable, have the instructor resolve them before proceeding.
4. Save, reopen the environment and confirm General type, branch, runtime, State and profile. A unique name alone does not isolate database writes; the profile target must also be yours.

### Create and run the job

1. From your environment choose **Create job → Deploy job**, name it `retail_build_<firstname>_<lastname>` and confirm the environment is yours.
2. Replace the unrestricted default build command with:

```bash
dbt build --select +mart_retail_inventory_priorities
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

- [Inspected industry template](https://github.com/dbt-Hands-on-Labs/dbt-retail-lab/tree/bfe544e4735f7e896be367c50e4a86fa428d0080)
- [Reference Platform workflow, Parts 5–7](../../transformations/fivetran_dbt_snowflake_hands_on_lab.md#part-5-accessing-dbt-platform)
- [dbt State](https://docs.getdbt.com/docs/deploy/dbt-state-about)
- [Deployment environments](https://docs.getdbt.com/docs/deploy/deploy-environments)

## Full answer

Compare this implementation with yours after attempting the exercise. Substitute your source identifiers in the project file. These are complete minimal files for the standard template; preserve extra instructor-provided project settings. Existing staging and source files remain in place. Different SQL can be correct if it meets the same grain, null-handling and reconciliation contracts.

### `dbt_project.yml`

```yaml
name: retail_lab
version: '1.0.0'
profile: default
require-dbt-version: [">=1.9.0", "<3.0.0"]
model-paths: [models]
test-paths: [tests]
macro-paths: [macros]
vars:
  source_database: '<SOURCE_DATABASE>'
  source_schema: '<SOURCE_SCHEMA>'
  review_threshold: 0
models:
  retail_lab:
    staging:
      +materialized: view
      +schema: staging
    marts:
      +materialized: view
```

### `models/marts/fct_retail_inventory_weekly.sql`

```sql
{{ config(materialized='view') }}
select
    w.row_id, w.product_id, w.store_id,
    p.product_name, p.category, s.store_name, s.region,
    try_to_date(cast(w.date as varchar), 'YYYY-MM-DD') as inventory_date,
    try_to_boolean(cast(w.is_anomaly as varchar)) as is_anomaly,
    w.current_stock_level, w.weekly_sales_volume, w.days_of_supply,
    p.restock_lead_time_days + {{ var('review_threshold', 0) }} as restock_lead_time_days,
    w.synced_at
from {{ ref('stg_retail__inv_weekly_data') }} w
left join {{ ref('stg_retail__inv_products') }} p on w.product_id = p.product_id
left join {{ ref('stg_retail__inv_stores') }} s on w.store_id = s.store_id
```

### `models/marts/mart_retail_inventory_priorities.sql`

```sql
{{ config(materialized='view') }}
with ranked as (
    select *,
        row_number() over (
            partition by product_id, store_id
            order by inventory_date desc nulls last, row_id asc
        ) as observation_rank
    from {{ ref('fct_retail_inventory_weekly') }}
)
select
    sha2(to_json(array_construct(product_id, store_id)), 256) as product_store_key,
    row_id, product_id, product_name, category, store_id, store_name, region,
    inventory_date, is_anomaly, current_stock_level, weekly_sales_volume,
    days_of_supply, restock_lead_time_days,
    days_of_supply < restock_lead_time_days as needs_replenishment,
    synced_at
from ranked
where observation_rank = 1
```

### `models/marts/_retail__addon_models.yml`

```yaml
version: 2
models:
  - name: fct_retail_inventory_weekly
    description: "One row per weekly inventory row_id, with product and store context."
    columns:
      - name: row_id
        data_tests:
          - unique
          - not_null
      - name: product_id
        data_tests:
          - not_null
          - relationships:
              arguments:
                to: ref('stg_retail__inv_products')
                field: product_id
      - name: store_id
        data_tests:
          - not_null
          - relationships:
              arguments:
                to: ref('stg_retail__inv_stores')
                field: store_id
      - name: inventory_date
        data_tests:
          - not_null
      - name: is_anomaly
        data_tests:
          - not_null
      - name: days_of_supply
        data_tests:
          - not_null
      - name: restock_lead_time_days
        data_tests:
          - not_null
  - name: mart_retail_inventory_priorities
    description: "One latest observation per product/store, with deterministic row_id tie-breaking."
    columns:
      - name: product_store_key
        data_tests:
          - unique
          - not_null
      - name: row_id
        data_tests:
          - unique
          - not_null
          - relationships:
              arguments:
                to: ref('fct_retail_inventory_weekly')
                field: row_id
      - name: inventory_date
        data_tests:
          - not_null
      - name: needs_replenishment
        data_tests:
          - not_null
```

### `tests/assert_retail_addon.sql`

```sql
-- Fail for row multiplication, missing observations, or a wrong latest date.
select 'weekly_grain' as failure
where (select count(*) from {{ ref('fct_retail_inventory_weekly') }})
   <> (select count(*) from {{ ref('stg_retail__inv_weekly_data') }})
union all
select 'latest_date' as failure
from {{ ref('mart_retail_inventory_priorities') }} m
where m.inventory_date <> (
    select max(f.inventory_date) from {{ ref('fct_retail_inventory_weekly') }} f
    where f.product_id = m.product_id and f.store_id = m.store_id
)
union all
select 'missing_pair' as failure
where (select count(*) from {{ ref('mart_retail_inventory_priorities') }}) <>
      (select count(*) from (select product_id, store_id from {{ ref('fct_retail_inventory_weekly') }} group by product_id, store_id) pairs)
```
