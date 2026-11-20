# dbt Wizard CLI add-on: Sales — Snowflake

*November 2026. This worksheet starts after the source lab's initial sync. You run dbt Fusion
and the dbt Wizard CLI on a provided Linux workstation, against your own copy of the dbt
project, then look at the results in the Snowflake web UI.*

## Objective

Build **Renewal review** from the synced Sales data. Connect dbt on your workstation to your
personal Snowflake target, create two tested marts with the dbt Wizard CLI, test a
controlled change to the business rule, view the results in Snowflake and push your work to
your repository.

## Requirements and instructor preparation

- Complete the source lab and successfully sync `qbr_data` into your own destination. Use **Fivetran → Connection → Schema** to confirm names and selected columns.
- A Linux workstation, accessed in your browser, with dbt Fusion (`dbtf`), Git and the dbt
  Wizard CLI (`wizard`) pre-installed. Wizard is already configured to use the lab's AI
  model; you don't sign in to it or provide a key.
- Your own private copy of [dbt-sales-lab](https://github.com/dbt-Hands-on-Labs/dbt-sales-lab), a GitHub repository named `lab-dbt-...` that only your
  workstation can reach.
- Credentials belong only in the files this worksheet names under `~/.dbt/`, never in SQL,
  YAML, prompts or Git.

## Part 0: Credentials

All of these are in the 1Password link your instructor provided.

### Workstation and dbt project

- **Gateway URL**, **Guacamole Username** and **Guacamole Password** for your workstation.
- **Git Clone Command** for your copy of the dbt project.

### Snowflake

- **Snowflake Account**, **Snowflake User**, **Snowflake Password**, **Snowflake Private Key**,
  **Snowflake Role**, **Snowflake Warehouse** and your databases. dbt connects with the
  **private key**; the **password** is only for signing in to the Snowflake web UI in Part 8.

## Part 1: Log in to your workstation

1. Open the **Gateway URL** and sign in with your **Guacamole Username** and
   **Guacamole Password**.
2. Click the **Terminal** icon on the bottom menu.

   > **Tips:** In the workstation terminal, paste with **CTRL + SHIFT + V**. If text copied
   > in your own browser pastes as nothing, press **CTRL + ALT + SHIFT** to open the
   > Guacamole clipboard panel, paste the text into its box, close the panel and paste
   > again in the terminal.

3. Confirm the tools are there:

   ```bash
   dbtf --version
   wizard --version
   git --version
   ```

   - `dbtf` is **dbt Fusion** (dbt v2). Every dbt command in this worksheet uses `dbtf`. Plain
     `dbt` on this workstation is dbt Core 1.x; don't mix the two in one project.
   - If dbt prints **New version available**, ignore it. Don't run `dbt system update`.

## Part 2: Clone your project and create a branch

1. Go to your home directory and run the **Git Clone Command** from your 1Password link:

   ```bash
   cd ~
   <paste your Git Clone Command here>
   cd ~/lab-dbt-*
   ```

2. Work on your own branch, so `main` stays as the template left it:

   ```bash
   git switch -c <firstname>_<lastname>_wizard
   ```

3. Look around: `ls models/staging` lists the staging models and `cat dbt_project.yml`
   shows the project configuration.

> **Checkpoint:** You are in your clone, on your own branch.

## Part 3: Connect dbt to Snowflake

Your workstation already has a dbt profile at `~/.dbt/profiles.yml`, written when the lab was
launched. It holds your non-secret connection details and points at a key file that you
create now.

1. Look at the profile:

   ```bash
   cat ~/.dbt/profiles.yml
   ```

   It looks like this, with your own values:

   ```yaml
   default:
     target: dev
     outputs:
       dev:
         type: snowflake
         account: "<your account>"
         user: "<your user>"
         role: "<your role>"
         warehouse: "<lab warehouse>"
         database: "<your database ending in _DEV>"
         schema: "dbt_<your suffix>"
         private_key_path: "/home/ubuntu/.dbt/snowflake_key.p8"
         threads: 4
   ```

2. Create the key file from the **Snowflake Private Key** field in your 1Password link. It starts with `-----BEGIN PRIVATE KEY-----` and ends with `-----END PRIVATE KEY-----`; copy all of it, including those two lines.
   Run this, paste the key, press **Enter**, then press **CTRL + D**:

   ```bash
   cat > ~/.dbt/snowflake_key.p8
   ```

3. Make the key readable only by you:

   ```bash
   chmod 600 ~/.dbt/snowflake_key.p8
   ```

4. Check the connection from inside your project:

   ```bash
   dbtf debug
   ```

   The connection test must be **OK**. If it fails, re-create the key file (an incomplete paste
   is the usual cause) or ask the instructor to check your provisioned access.

> **Checkpoint:** `dbtf debug` connects to your Snowflake target.

## Part 4: Configure and inspect the source

1. Record these identifiers. Copy the actual names from your profile, your credentials and the Fivetran schema configuration; do not infer them from another attendee's lab.

| Placeholder | Your value |
|---|---|
| `<SOURCE_DATABASE>` | Your provisioned database ending in `_LANDING`, where Fivetran landed the source data |
| `<SOURCE_SCHEMA>` | The actual Fivetran schema, including your connection prefix and source-schema suffix |
| `<DEV_DATABASE>` | The `database` in your `~/.dbt/profiles.yml`, ending in `_DEV` |
| `<DEV_SCHEMA>` | The `schema` in your `~/.dbt/profiles.yml`, for example `dbt_ab12cd34` |
| Warehouse / role | The `warehouse` and `role` in your `~/.dbt/profiles.yml` |

2. Inspect the template's staging SQL and source YAML. This exercise uses `stg_sales__qbr_data`. The source definition reads `source_database` and `source_schema` from project variables. Set both explicitly; do not rely on a destination-specific default.
3. Update `dbt_project.yml` in your clone: keep the project name `sales_lab`, profile and paths, set both source variables, add `review_threshold: 60`, retain the staging view configuration and add marts as views. Replace the placeholders with your actual identifiers. Preserve any additional instructor-provided configuration.

4. Keep the existing staging models and source definitions. They preserve the template’s source-row behavior and rename `_fivetran_synced` to `synced_at`; this exercise does not introduce a soft-delete policy.

Use this complete minimal configuration as a reference for the standard template; preserve extra settings if your instructor has customized the project:

```yaml
name: sales_lab
version: '1.0.0'
profile: default
require-dbt-version: [">=1.9.0", "<3.0.0"]
model-paths: [models]
test-paths: [tests]
macro-paths: [macros]
vars:
  source_database: '<SOURCE_DATABASE>'
  source_schema: '<SOURCE_SCHEMA>'
  review_threshold: 60
models:
  sales_lab:
    staging:
      +materialized: view
      +schema: staging
    marts:
      +materialized: view
```

5. Build only the staging inputs needed for this exercise:

```bash
dbtf build --select stg_sales__qbr_data
```

6. Resolve failures before continuing. Confirm the built staging relations are in your development target plus `_staging`. The source can be empty; a successful zero-row build is not evidence of a populated pipeline. Check the Fivetran sync status and discuss empty inputs with the instructor.

   > **Note:** The build can finish with a warning that the `marts` configuration path in
   > `dbt_project.yml` is unused (`UnusedResourceConfigPath`). This is expected: the
   > `models/marts` folder doesn't exist until the Wizard creates it in Part 5. You can ignore it.

## Part 5: Build the marts with the dbt Wizard CLI

**Business question:** Renewal review. Count QBR observations, not distinct accounts; the mean is not weighted by contract value.

Use these files:

| File | Purpose |
|---|---|
| `models/marts/fct_sales_observations.sql` | Row-level fact view |
| `models/marts/mart_sales_review_summary.sql` | Business-facing mart view |
| `models/marts/_sales__addon_models.yml` | Model descriptions and generic data tests |
| `tests/assert_sales_addon.sql` | Singular reconciliation test; returns rows only when an invariant fails |

**Model requirements**

- Build `fct_sales_observations` at one row per `qbr_data.record_id`. Use `ref('stg_sales__qbr_data')` and select the source key as `observation_id`.
- Derive `group_key` from `industry`: use `missing:` for null and `value:` plus its string value otherwise. Keep `group_name` as the display string, using `Unknown` for null. Distinguish a real value named Unknown from a missing value through the key.
- Calculate `metric_value` from `renewal_probability` using safe numeric conversion. Convert arithmetic inputs before calculating. Division by zero must produce null. This measures the record-level renewal probability on a 0–100 scale.
- Preserve `synced_at`. Set `requires_review` to `metric_value < review_threshold`, initially 60. Missing metrics and their flags stay null; never silently turn them into zero or false.
- Build `mart_sales_review_summary` from the fact at one row per `group_key`. Include group name, total observations, measured observations, missing metric count, review count, unweighted mean metric and review share. Divide review count by measured observations with a zero-safe denominator and decimal arithmetic.
- Test unique/non-null observation IDs and group keys, observation-to-staging key relationships and non-null counts. Add a singular test for source/fact row preservation, summary row-count reconciliation, measured + missing = total and review count no greater than measured count. Legitimately missing metrics must remain visible, not fail a blanket non-null test.

1. From your project directory, parse the project. Wizard reads the compiled project in
   `target/`, so it needs this first:

   ```bash
   dbtf parse
   ```

2. Start an interactive Wizard session in the project directory:

   ```bash
   wizard
   ```

3. Paste the model requirements above, followed by this prompt, and press **Enter**:

```text
Inspect the existing sales staging models and source definitions first.
Create only these four files:
models/marts/fct_sales_observations.sql
models/marts/mart_sales_review_summary.sql
models/marts/_sales__addon_models.yml
tests/assert_sales_addon.sql

Use Snowflake SQL and ref() dependencies, view materializations and built-in dbt tests.
Follow the Model requirements immediately above (paste them with this prompt).
Read review_threshold from dbt_project.yml with var('review_threshold').
This project runs on dbt Fusion v2: nest generic test parameters such as `to`
and `field` under `arguments:` in YAML.
Preserve existing staging, sources, project configuration and packages.
Document grains, units, null handling and the limits of the review rule.
Propose the files and explain their logic before editing. Do not run commands,
change settings, commit or push. Do not request credentials or warehouse-console access.
```

4. Review the proposed joins, grain, expressions, tests and file paths. Ask Wizard to correct
   discrepancies before it edits anything.
5. When Wizard asks to create or change files, approve only the four exercise files. Decline
   any other edit, and any command it proposes to run: you run dbt yourself in Part 6.
6. Leave the session (type `/` to see Wizard's commands, or press **CTRL + C**), then review
   what changed:

   ```bash
   git status
   git diff
   ```

   Only `dbt_project.yml` (your changes from Part 4) and the four new files should appear.
   Restore anything else with `git checkout -- <file>`.

7. See how the models depend on each other before building them:

   ```bash
   dbtf ls --select +mart_sales_review_summary --resource-type model
   ```

## Part 6: Build and validate

1. Inspect the resolved targets before executing:

   ```bash
   dbtf ls --select +mart_sales_review_summary --resource-type model --output json --output-keys name database schema alias
   ```

2. Confirm every selected model targets `<DEV_DATABASE>`. The staging views use `<DEV_SCHEMA>_staging` and the two marts use `<DEV_SCHEMA>` (Snowflake may display unquoted names in uppercase). Correct your profile if the target differs.
3. Build the mart, its ancestors and associated tests:

   ```bash
   dbtf build --select +mart_sales_review_summary
   ```

   > If the build stops while parsing with `dbt1159 Deprecated test arguments ... at top-level`,
   > a generic test's `to` and `field` sit directly under the test name, the dbt 1.x layout.
   > Nest them under `arguments:`, as in
   > `relationships: {arguments: {to: ref('model_name'), field: key_column}}`, and build again.

4. Require a successful final result. Read failing test output and repair the model or
   assumptions; do not remove tests merely to get a green run. You can ask Wizard to help
   diagnose a failure. Verify the singular reconciliation test was selected. If it was not,
   run it explicitly:

   ```bash
   dbtf test --select assert_sales_addon
   ```

5. Run a bounded aggregate check and record the result as your baseline:

   ```bash
   dbtf show --inline "select coalesce(sum(observation_count), 0) as observations, coalesce(sum(measured_count), 0) as measured, coalesce(sum(missing_metric_count), 0) as missing, coalesce(sum(review_count), 0) as flagged from {{ ref('mart_sales_review_summary') }}" --limit 1
   ```

   Do not assume fixed row counts: each lab's source/filter/sync state can differ.

> **Checkpoint:** The selected models and tests pass, the grain/reconciliation checks hold,
> and you have recorded baseline aggregates.

## Part 7: Test a controlled change

1. Temporarily change `review_threshold` in `dbt_project.yml` from `60` to `80`.
   This changes the illustrative review rule without updating source data.
2. Run the build and aggregate commands from Part 6 again. Record the changed flag count.
   Row counts must remain unchanged; a flag count can stay the same if no values cross the
   threshold.
3. Restore `review_threshold: 60`, rebuild and rerun the aggregate check. Confirm the
   baseline business results return, and that `git diff dbt_project.yml` no longer shows
   the temporary value.

| Run | Aggregate result |
|---|---|
| Baseline | |
| Changed threshold | |
| Restored threshold | |

## Part 8: View your data in the Snowflake web UI

Look at the tables and views dbt built, in Snowflake's own UI.

1. In your own browser, open the Snowflake sign-in page for the **Snowflake Account** in your
   1Password link (`https://<Snowflake Account>.snowflakecomputing.com`, or the **Snowflake
   Hostname** if one is listed).
2. Sign in with your **Snowflake User** and **Snowflake Password**.

   > **MFA enrollment:** Snowflake requires multi-factor authentication for password
   > sign-ins. The first time you sign in, Snowflake will prompt you to enroll and offer
   > several options. We recommend choosing **Passkey**. Complete the enrollment before
   > continuing.

3. Open **Horizon Catalog → Database Explorer** (or **Data → Databases**), expand
   `<DEV_DATABASE>` and find two schemas: `<DEV_SCHEMA>_STAGING` with the staging views and
   `<DEV_SCHEMA>` with your marts. Snowflake shows unquoted names in uppercase.
4. Click your mart and open **Data Preview** to see its rows.
5. Click **+ Create → SQL Worksheet**. In the worksheet's context selector, choose your
   **Snowflake Role** and **Snowflake Warehouse**, then run:

   ```sql
   select *
   from <DEV_DATABASE>.<DEV_SCHEMA>.mart_sales_review_summary
   limit 100;
   ```

6. Run the aggregate check from Part 6 here too, replacing the `{{ ref(...) }}` with
   `<DEV_DATABASE>.<DEV_SCHEMA>.<model name>`. The result should match what `dbtf show`
   returned.

> You can read your own databases only; your role cannot see other attendees' data.

**Checkpoint:** You can see the marts dbt built, and their rows, in the Snowflake web UI.

## Part 9: Commit and push your work

1. Check that only the exercise files changed and that no key file is in the project:

   ```bash
   git status
   ```

2. Commit and push your branch:

   ```bash
   git add dbt_project.yml models/marts/fct_sales_observations.sql models/marts/mart_sales_review_summary.sql models/marts/_sales__addon_models.yml tests/assert_sales_addon.sql
   git commit -m "Add sales marts with dbt Wizard"
   git push -u origin <firstname>_<lastname>_wizard
   ```

> **Checkpoint:** Your branch, with the four exercise files and the restored threshold, is in
> your repository.

## What you did

You connected dbt Fusion on a workstation to your personal Snowflake target, used the dbt
Wizard CLI to turn source observations into tested marts, checked a controlled change to the
business rule, looked at the results in the Snowflake web UI and pushed your work.
You can explain the grain, null handling and limits of the business rule.

## References

- Inspected industry template: [dbt-sales-lab](https://github.com/dbt-Hands-on-Labs/dbt-sales-lab)
- [Use dbt Wizard locally](https://docs.getdbt.com/docs/dbt-ai/wizard-quickstart)
- [dbt Wizard command reference](https://docs.getdbt.com/docs/dbt-ai/wizard-cli-reference)
- [dbt Platform edition of this worksheet](../../dbt_platform/sales/snowflake_wizard_dbt_platform_hands_on_lab.md)
