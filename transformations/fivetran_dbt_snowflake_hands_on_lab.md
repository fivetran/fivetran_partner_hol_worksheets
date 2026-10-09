# Hands-on Lab: Fivetran + dbt with Snowflake Cortex

*Updated October 2026. Verified against the current Fivetran, dbtLabs, and Snowflake documentation.*

Thank you for registering for our hands-on lab. This worksheet provides everything you need to prepare and to work through the lab. Please read through the requirements first. If you can't meet them, let us know and we'll rebook you on another workshop. 

> **Completing this lab self-paced?** This worksheet is written for our instructor-led sessions, which use a provisioned Fivetran account, a provisioned dbt Platform account, a PostgreSQL source, and a Snowflake destination with instructor-provided credentials. If you're working through the lab on your own, you don't need any of those: use your own Fivetran partner account and any [Fivetran-supported source](https://fivetran.com/docs/connectors) and [destination](https://fivetran.com/docs/destinations) you have access to. Skip the Housekeeping section and Parts 0–1, and use the remaining parts as an example of what each step looks like, adapting setup details, credentials, and naming to your own resources. If you can't provision your own resources, contact Fivetran to schedule an instructor-led session.

---

## Objective

In this workshop, you'll move data from a PostgreSQL source database into Snowflake using Fivetran, transform it into trusted, production-ready data products using dbt, and see how modern data capabilities like the dbt v2, dbt Wizard, and dbt State make that process faster, smarter, and more efficient.

This is a **full-stack, end-to-end walkthrough of how modern data teams go from raw data to AI-powered outcomes** — no prior experience with any of these tools is required.

## Requirements

- **Web browser**
  - Chrome or Firefox (latest version recommended)

---

## Housekeeping

**Invitation to the `TRANSFORMATIONS_HANDS_ON_LAB` Fivetran account**

- Do **not** set up a new account or trial. You will receive an invite to a dedicated Fivetran account prior to the lab.
- Confirm that you have received the invite to the Fivetran account.
- The invite is sent to the email address you used to sign up for this lab and originates from `notifications@fivetran.com`.

**Invitation to the `Partner Hands-on Labs` dbt Platform account** 
- Do **not** set up a new account or trial. You will receive an invite to a dedicated dbt Platform account prior to the lab.
- Confirm that you have received the invite to the dbt Platform account.
- The invite is sent to the email address you used to sign up for this lab and originates from `support@getdbt.com`.

**Snowflake data warehouse**

- Credentials provided by your instructor (see Part 0).

**PostgreSQL database**

- Credentials provided by your instructor (see Part 0).

---

## What you'll do in this lab

1. Create a destination in Fivetran
2. Create and sync a connection
3. Create transformations using dbt

---

## Part 0: Credentials

### Snowflake

- Provided by your instructor via a 1Password link. The Fivetran destination uses **key pair authentication**, so the link includes a private key rather than a password.
   - When you paste these credentials into the destination setup form, make sure you select **SaaS** as the deployment model.

### PostgreSQL

- Provided by your instructor via a 1Password link.

---

## Part 1: Accessing the provided resources

### Access Fivetran (new users)

1. Log in to Fivetran at [https://fivetran.com/login](https://fivetran.com/login).
2. If you just signed up for the first time, use the email and password you used at sign-up.
3. As a new user, you'll go through a short new-user onboarding flow. The exact prompts change from time to time — you can **Skip** the "tell us about yourself" style questions, and where you're asked what software you use, pick any option (for example, **Salesforce**) or click **Skip**. These selections are cosmetic and don't affect the lab.
4. When prompted to set up your first connection, click through the welcome prompt (for example, **Let's go**) to reach your dashboard.
5. Close any informational pop-ups (such as pricing or product announcements).

### Access Fivetran (existing users)

1. If you already have a Fivetran account, use your existing Fivetran credentials to log in.
2. If you forgot your password, click the **Forgot your password?** link.
3. Switch to the account you've been added to using the account drop-down menu in the top-left of the dashboard.

---

## Part 2: Creating a destination

1. In Fivetran, navigate to the **Destinations** tab and click **Add destination**.
2. Search for **Snowflake** and select it.
3. Give the destination the name `<firstname>_<lastname>_snowflake`, replacing `<firstname>` and `<lastname>` with your actual first and last names. Click **Add**.
4. Enter the provided Snowflake credentials.
5. For the authentication method, select **Key Pair** and paste in the provided private key.
6. In the **Deployment model** section, select **SaaS**.
7. For **Table Type** select **Snowflake Native Tables**
8. Leave **Connection Method** and **Storage for Unstructured Files** as their defaults.
9. Click **Load Virtual Warehouses** and select the warehouse provided in your credentials, which should be `TECH_FOUNDATIONS_HOL_WAREHOUSE`

   > **Note:** Do not select the `SNOWFLAKE_LEARNING_WH` as it will not work for this lab.

10. Leave the remaining fields at their default values. Do not make any changes.
11. Click **Save & Test**.
12. Wait for the setup tests to complete. When they pass, you'll see all connection tests marked successful.
13. Click **View Destination** (or **Continue**) to proceed.

> **Note:** Every destination you create automatically includes the free **Fivetran Platform Connection** (schema `fivetran_metadata`), which loads metadata about your account, connections, and usage. You'll use it in Part 4.

---

## Part 3: Creating a connection

> In current Fivetran terminology, a **connector** is the reusable source type (for example, PostgreSQL), and a **connection** is the configured instance you set up from it. You can create many connections from the same connector.

1. Navigate to the **Connections** tab and, in the top-right corner, click **Add connection**.
2. Search for **PostgreSQL**, hover over the connector tile, and click **Set up**.
3. Select the destination you created in Part 2 (make sure you don't use someone else's destination), then click **Select**.
4. The value you enter in the **Destination schema prefix** field becomes the name of your connection. Use the format `<firstname>_<lastname>_postgres`, replacing `<firstname>` and `<lastname>` with your actual first and last names.
5. For **Destination names** select **Fivetran naming**
6. Populate the setup form with the provided PostgreSQL credentials.
7. Leave **Connection Method** at its default value, **Connect directly**.
8. For **Update Method**, select **Query-Based**.
   - *Context:* PostgreSQL's older XMIN and Fivetran Teleport Sync methods have been sunset and replaced by **Query-Based** change data capture. The other available method is **Logical replication** (using the `pgoutput` plugin). Existing Teleport/XMIN connections keep working.
9. Click **Save & Test**.
10. When prompted, confirm the TLS certificate by selecting the certificate and clicking **Confirm**.
11. Wait for the setup tests to complete. When they pass, you'll see all connection tests marked successful.
12. Click **Continue**.
13. Fivetran now fetches all tables, schemas, and columns for the database.
14. The database user you have been provided has access to the `retail` schema. Make sure the `retail` schema and all of its tables are selected, then click **Save & Continue** to proceed.
15. For handling schema changes, select **Allow all**, then click **Continue** (or **Save**).
16. Click **Start Initial Sync**. Wait for the initial (historical) sync to finish, then let at least one **incremental** sync complete. At least one incremental sync must complete for this lab.
17. The sync will finish in about 1–2 minutes. A successful sync shows the connection status as **Active** with the synced tables and row counts.

> **Checkpoint — sync schedule and sync modes:**
> - By default, connections sync on a **fixed interval of every 6 hours**. You can change this on the connection's **Settings** tab (options range from 1 minute up to 24 hours; a **Cron** schedule is also available on Enterprise/Business Critical plans).
> - Fivetran supports two **sync modes**: **Soft delete** (the default — deleted source rows are marked with `_fivetran_deleted = TRUE` rather than removed) and **History mode** (SCD Type 2, which tracks every version of a row using `_fivetran_start`, `_fivetran_end`, and `_fivetran_active`).

---

## Part 4: Verify the Data in Snowflake
1. Access the Snowflake UI login via the URL provided in the 1Password link
2. Enter in the provided Username and Password provided via the 1Password link
3. You'll need to configure MFA, we recommend Passkey Authentication since it's only a few clicks to setup, but other methods can be selected if Passkey is not available
4. Once you're logged in, click **Catalog** in the left navigation bar
5. Find your schema (e.g., `{firstname}_{lastname}_postgres_retail`)
6. Click **Tables**
7. Click on any of the tables such as `avocado_prices`
8. Click **Data Preview** tab to view the Fivetran synced data and scroll all the way to the right to see `_fivetran_synced`


## Part 5: Accessing dbt Platform

1. Accept the invite from `support@getdbt.com` found in your email by clicking the button `Join Partner Hands-on Labs`
   - The email is titled `A colleague invites you to collaborate with them in dbt Cloud`
2. Fill in the form:
   - **First Name**: Your first name
   - **Last Name**: Your last name
   - **Password**: Set a password
   - **Confirm Password**: Confirm your password
3. Select **I aggree to dbt's Terms of Service**
4. Click **Get started**
5. Record these credentials — you may need them to log back in during the lab.
6. Once you're logged in click on **Project** in the left navigation bar and select the project
   - There should only be one visible project starting with `LAB_DBT`
7. Click on `Studio` in the left navigation bar
8. To configure your personal developer credentials do the following
   - Click on **Status: Disconnected** to open up the **Server status**. You'll notice that it's missing credentails
   - Click on **View Credentials** to open up the pane to enter your development credentials 
      - In **Connection details**, set **Database** to the development database provided for your lab (the name ends in `_DEV`). Do not leave this field blank or use the shared `LABSTUDIOS` database.
      - For **Auth method** select **Key pair**
      - Enter the **Username** provided to you via the 1Password link
      - Enter the **Private key** provided to you via the 1Password link
      - DO NOT enter a **Private key passphrase**. Leave it blank.
      - Change **Schema** to `dbt_<firstname>_<lastname>` where `<firstname>` and `<lastname>` are replaced with your actual first and last names.  
      - Leave **Target name** (`default`) and **Threads** at their default values. **Target name** is a dbt profile label; the **Database** field above controls which Snowflake database receives your models.
      - Click **Save** in the top right
9. Reload the page you should now see that the **Server status** says `Connected`
10. On the left hand navigation bar click on **Create branch** and use the naming convention `<firstname>_<lastname>_branch` where `<firstname>` and `<lastname>` are replaced with your actual first and last names.  
11. Click **Create** branch

> **Checkpoint — dbt Platform Setup**
> - You have now setup your development environment in dbt Platform by connecting to Snowflake with your credentials. You have also created an isolated branch in the connected GitHub repostitory that you can work in independently.

---

## Part 6: Build a Retail Data Product with dbt Wizard

You will turn weekly inventory observations into a tested data product that answers: **Which product/store combinations need replenishment?** The lab uses historical retail data. A replenishment flag is a simple business rule, not a demand forecast.

### 6.1 Confirm your development context

1. In **dbt Studio**, confirm your personal development branch is selected and **Status** is **Parsed**.
2. If you just changed your connection settings, open **More IDE options** at the bottom right, select **Restart Studio**, and click **Restart**. Your project files are preserved.
3. Record these names from your own lab environment. Replace the placeholders in every command and YAML example below before running it.

| Placeholder | Where to find the value |
|---|---|
| `<LANDING_DATABASE>` | Your provisioned Snowflake database ending in `_LANDING` |
| `<RETAIL_SCHEMA>` | The destination schema containing your synced retail tables; Fivetran prefixes the PostgreSQL schema, for example `JANE_DOE_POSTGRES_RETAIL` |
| `<DEV_DATABASE>` | Your personal connection's **Database**, ending in `_DEV` |
| `<DBT_SCHEMA>` | Your personal dbt schema, for example `DBT_FIRSTNAME_LASTNAME` |
| `<LAB_WAREHOUSE>` | Your instructor-provided Snowflake warehouse |

4. Open `dbt_project.yml` and the files under `models/staging/retail`. The template already provides source definitions and staging models. We will preserve these and pass the landing database/schema as command variables.
5. Locate the three staging models used in this exercise:

| Landing table | Existing dbt model | Purpose |
|---|---|---|
| `INV_WEEKLY_DATA` | `stg_retail__inv_weekly_data` | Weekly sales, stock, days of supply, date and anomaly fields |
| `INV_PRODUCTS` | `stg_retail__inv_products` | Product details and replenishment lead time |
| `INV_STORES` | `stg_retail__inv_stores` | Store details and region |

> **Before continuing:** The Fivetran initial sync must be complete. The provisioned source has nine retail tables, but this exercise uses only these three. Self-paced learners need equivalent retail tables and the matching staging template to reproduce these steps. The template's `source_schema: 'retail'` may not match the destination schema. Always supply your actual values through `--vars` below.

### 6.2 Ask Wizard to create the marts

[dbt Wizard](https://docs.getdbt.com/docs/platform/wizard-platform) is available in the **dbt Wizard** panel within Studio. This exercise uses the browser experience; no terminal installation is required. Wizard availability and usage depend on your provisioned account. Ask your instructor if the panel is unavailable.

1. Open the **dbt Wizard** tab beside **Commands** and **Lineage**. Keep **Ask for approval** selected.
2. Paste the following prompt. Wizard should inspect the project and propose a small change before writing files.

```text
Create a retail inventory exercise using the existing staging models:
stg_retail__inv_weekly_data, stg_retail__inv_products and
stg_retail__inv_stores. Inspect their columns and existing project conventions first.

Create only these three new files:
models/marts/fct_retail_inventory_weekly.sql
models/marts/mart_retail_inventory_priorities.sql
models/marts/_retail_inventory__models.yml

Use view materializations and ref() dependencies. Preserve all existing files,
including staging models, source definitions, dbt_project.yml and packages.

The weekly fact must preserve one row per inventory row_id. Left join products
on product_id and stores on store_id. Include row_id, product_id, product_name,
category, subcategory, brand, vendor_name, store_id, store_name, region,
store_type, weekly_sales_volume, current_stock_level, days_of_supply,
restock_lead_time_days, anomaly_type and synced_at. Convert the text date with
TRY_TO_DATE(date, 'YYYY-MM-DD') as inventory_date and the text anomaly flag with
TRY_TO_BOOLEAN(is_anomaly) as is_anomaly.

The priorities mart must reference the weekly fact and select one latest row
per product_id/store_id using inventory_date descending nulls last, with row_id
ascending as a deterministic tie-breaker. Include a stable product_store_key
using a null-aware, unambiguous encoding of both IDs before hashing. Carry the
weekly fact columns through and add:
days_of_supply < restock_lead_time_days as needs_replenishment.

Document both grains and columns. Add unique/not_null tests for the weekly
row_id and latest product_store_key/row_id; not_null and relationships tests
for product/store IDs; not_null tests for dates; and not_null plus boolean
accepted_values tests for is_anomaly and needs_replenishment. Test the latest
row_id relationship to the weekly fact. Use built-in tests, no new packages.

Show the proposed files and explain the joins and grain. Do not run commands,
change settings, commit or push yet.
```

3. Review the proposal, then ask Wizard to create **only those three files**. Approve the file edits when the proposed paths and contents match the exercise.
4. Review the diffs under **Version control**. The only changes should be the three added marts files.
5. Open each model and inspect **Lineage**. Trace the three staging inputs into the weekly fact, then into the latest priorities mart.

> **Why two models?** `fct_retail_inventory_weekly` preserves weekly history. `mart_retail_inventory_priorities` has one latest observation per product/store, so its stock and replenishment counts do not multiply across weeks. The date and boolean tests catch failed text conversions instead of silently accepting nulls.

### 6.3 Verify the target and build

Use the **Commands** tab in Studio, or ask Wizard to run these exact commands and approve each command once. Replace both placeholders inside `--vars`. Run each command as one line.

1. Resolve the selected models before building:

```bash
dbt ls --select +mart_retail_inventory_priorities --resource-type model --output json --output-keys unique_id name database schema alias resource_type --vars '{"source_database":"<LANDING_DATABASE>","source_schema":"<RETAIL_SCHEMA>"}'
```

2. Check that all five models target **your `<DEV_DATABASE>`**. The two marts use `<DBT_SCHEMA>`; the template appends `_STAGING` for the three staging models. If a target points elsewhere, correct the personal connection in Part 5 and restart Studio before proceeding.
3. Build the selected mart, its ancestors, and associated tests:

```bash
dbt build --select +mart_retail_inventory_priorities --vars '{"source_database":"<LANDING_DATABASE>","source_schema":"<RETAIL_SCHEMA>"}'
```

4. Wait for the command's final result. Expand the run output and investigate any failures before continuing. The verified lab implementation selected **5 views and 29 tests**, all passing. Generated test counts may differ if your reviewed YAML differs.
5. Run a bounded aggregate check:

```bash
dbt show --inline "select (select count(*) from {{ ref('stg_retail__inv_weekly_data') }}) as staging_rows, (select count(*) from {{ ref('fct_retail_inventory_weekly') }}) as weekly_rows, count(*) as latest_rows, count(distinct product_store_key) as distinct_pairs, count_if(needs_replenishment) as replenishment_count, min(inventory_date) as first_observation, max(inventory_date) as last_observation from {{ ref('mart_retail_inventory_priorities') }}" --limit 1 --vars '{"source_database":"<LANDING_DATABASE>","source_schema":"<RETAIL_SCHEMA>"}'
```

6. Ask Wizard to perform bounded, read-only checks that each latest row has the maximum date for its product/store, and that the weekly joins have no unmatched product/store IDs. Keep the same source variables and development relations.

The October 2026 verification of the provisioned dataset returned:

| Check | Result |
|---|---:|
| Staging / weekly fact rows | 28,236 / 28,236 |
| Latest rows / distinct product-store pairs | 543 / 543 |
| Incorrect latest dates | 0 |
| Unmatched product / store rows | 0 / 0 |
| Product/store combinations needing replenishment | 116 |
| Latest observation date | 2023-12-24 |

Your counts can differ if the source dataset changes. Validate the relationships and grain rather than editing data to match these numbers. The existing staging models determine Fivetran deletion handling; this exercise does not add a soft-delete policy.

> **Checkpoint — Tested Retail Marts**
> - Your development views build successfully and their tests pass.
> - You can explain the weekly and latest-observation grains.
> - Your reviewed changes are still on your development branch. Keep them uncommitted until the deployment preparation in Part 7.4.

---

## Part 7: Observe dbt State

[dbt State](https://docs.getdbt.com/docs/deploy/dbt-state-about) decides whether selected resources need work using information such as code changes, dependencies and data state. This is different from a manual `state:modified` selector. We will keep the build command identical and change one upstream expression to observe its decisions.

### 7.1 Enable dbt v2 and State for your user in Studio

**Prerequisite:** Your instructor or account admin must have enabled account access to dbt v2 and made dbt State available for the lab. If either option is missing, ask the instructor to check availability. Attendees should not activate a new account trial or change shared environments/jobs.

**Select the personal dbt v2 runtime:**

1. In your lab project's **Studio**, click **dbt version** in the bottom status bar.
2. In the **Development environment** panel, click **Edit** beside **Personal version override**. Use this personal override, not **Edit environment**, which opens the shared project setting.
3. In **User development settings**, open the **dbt version** dropdown and select **v2 Stable**. The verified UI also labeled this option **Preview** and **formerly Fusion Stable**. This override affects only your user for this project.
4. Click **Save and Restart**. Wait for Studio to finish restarting and show **Status: Compiled**; confirm the status bar displays **dbt version: v2 Stable**. If your user already has this selection, leave it in place.
5. Preserve your development database, schema, credentials and other settings. The release track receives updates; do not pin a particular patch version. The lab validation used **dbt 2.0.8**. See [personal version overrides](https://docs.getdbt.com/docs/dbt-versions/upgrade-dbt-platform-version#override-dbt-version).

**Enable State for the same user and project:**

1. Open your profile menu in dbt and go to **Settings**. Under **Your profile**, select **Credentials**.
2. In **Search by project name**, find and select your lab project, then click **Edit** in the project credentials drawer.
3. Scroll to **User development settings**.
4. Set **dbt State** to **Enabled** if it is currently **Disabled (inherited from development environment)**. Leave it enabled if already selected. This override affects only your user for this project.
5. Preserve the **v2 Stable** selection, database, schema, credentials and other settings; click **Save**.
6. Return to **Studio**. Open **More IDE options → Restart Studio → Restart** and wait for **Status: Compiled**.
7. Run the focused build from Part 6.3 and wait for its final result. In **Commands**, expand that run's **System logs**. Verify both the actual **2.x runtime** and **dbt State is enabled**. The validated logs contained `Running Fusion version: 2.0.8`, `dbt 2.0.8` and `Info dbt State is enabled`; the older Fusion label can still appear in v2 logs. Record the version actually reported by your run, rather than relying only on the status bar.

If v2 is unavailable, or the logs still report v1 or disabled State, ask the instructor to check account availability and your personal overrides before comparing State results. Do not describe a normal dbt run as State reuse.

### 7.2 Compare the baseline and unchanged rerun

1. Save the final result of the State-enabled build from Part 7.1 as your comparison baseline. State may already have reusable results from an earlier build; the first run you record does not necessarily execute every model and test.
2. Without editing files, syncing source data or changing variables, run the **same** focused build again:

```bash
dbt build --select +mart_retail_inventory_priorities --vars '{"source_database":"<LANDING_DATABASE>","source_schema":"<RETAIL_SCHEMA>"}'
```

3. Expand both command outputs. Record the actual built/reused/test results and compare them. Use State's decision messages rather than elapsed time alone as evidence.

The dbt v2 validation used an already-warm State history. With the same selector and variables, it produced:

| Run | Models built | Tests executed | Nodes reused | Result |
|---|---:|---:|---:|---|
| State-enabled comparison baseline | 0 | 0 | 34 (5 models + 29 tests) | No errors or warnings |
| Unchanged rerun | 0 | 0 | 34 (5 models + 29 tests) | No errors or warnings |

The baseline took 5.70 seconds and the unchanged rerun took 8.55 seconds. These are observed timings, not performance guarantees; reuse does not guarantee that every rerun is faster. A cold State history may execute the selected nodes to establish reusable results. Record your actual outcome without clearing State or changing data to reproduce this table. Reused tests were not re-executed in either of these runs.

### 7.3 Simulate a supplier delay, then restore it

1. Open `models/marts/fct_retail_inventory_weekly.sql` and save the original expression:

```sql
products.restock_lead_time_days,
```

2. Temporarily replace only that expression with:

```sql
products.restock_lead_time_days + 7 as restock_lead_time_days,
```

3. Save the file. This simulates a seven-day delay in transformation logic; it does not change PostgreSQL or the Fivetran landing data.
4. Run the same focused build and inspect State's decisions for the changed fact and downstream mart. Use **Lineage** to explain why the mart depends on the changed expression.
5. Run this bounded aggregate check with your source variables. Record the new replenishment count and verify the row counts are unchanged:

```bash
dbt show --inline "select (select count(*) from {{ ref('fct_retail_inventory_weekly') }}) as weekly_count, count(*) as latest_count, count_if(needs_replenishment) as needs_replenishment_count from {{ ref('mart_retail_inventory_priorities') }}" --limit 1 --vars '{"source_database":"<LANDING_DATABASE>","source_schema":"<RETAIL_SCHEMA>"}'
```
6. Restore the **exact original expression**, save, and run the same build again. Repeat the aggregate check to confirm the baseline business result is restored.
7. Review the final file diff. The temporary `+ 7` must be gone before moving on to Cortex.

In the dbt v2 validation, the temporary change increased replenishment combinations from **116 to 198** in the verified dataset; weekly/latest row counts remained **28,236 / 543**. State rebuilt only `fct_retail_inventory_weekly`, reused all four other view definitions, executed 23 tests and reused six tests. The unchanged priorities view nevertheless reflected the new upstream lead-time values.

After restoring the original expression, the verified build rebuilt the weekly fact and reused the other four models plus all 29 tests (**33 reused nodes**), with no warnings or errors. The independent aggregate query returned **28,236 / 543 / 116**, confirming that the baseline business result was restored. Reused test results are distinct from the fresh aggregate query performed after restoration.

> **Interpret the evidence carefully:** Views expose upstream data dynamically. A downstream view can show changed values even if State reuses its unchanged definition. Reuse decisions for tests and models can differ; do not assume every node skips, every downstream view rebuilds, or warehouse/model usage becomes free. This exercise demonstrates a code change on a static dataset, not live source-freshness detection. See [State examples](https://docs.getdbt.com/docs/deploy/dbt-state-examples) and [Studio enablement](https://docs.getdbt.com/docs/deploy/dbt-state-enable-studio).

> **Checkpoint — State and Restoration**
> - Your personal runtime is dbt v2, State is enabled, and the execution logs confirm both.
> - You recorded baseline, unchanged, changed and restored outcomes.
> - The original lead-time expression and baseline replenishment result are restored.

### 7.4 Create your deployment environment and run a job

Create a **General deployment environment** and run the retail build manually from your own Git branch. A project supports one Development environment, one Staging deployment environment and one Production deployment environment, plus multiple General deployment environments. Use **General** for this exercise. A unique name identifies your environment; it does not make the environment or its credentials private. See [environment types](https://docs.getdbt.com/docs/dbt-platform-environments).

> **Instructor prerequisites:** Learners need a Developer seat, permission to create deployment environments and create/run jobs, and Git permission to push their personal branch. On Enterprise permission sets, **Developer** alone has read-only environment access; **Job creator** can create jobs but cannot create environments. In the lab, adding **Job Admin scoped to the learner’s dbt project**, alongside existing Studio access, made **Create environment** available. The instructor must arrange that project-scoped access and an approved deployment connection profile; no account-wide administrator role is needed for the environment exercise. If **Create Environment**, **Edit** or **Create job** is missing, ask the instructor to resolve access before proceeding. See [permissions](https://docs.getdbt.com/docs/platform/manage-access/enterprise-permissions).

**Prepare your branch and deployment target**

1. Complete Part 7.3: restore the original expression and verify the baseline result. In Studio, confirm you are on your personal branch.
2. Under **Version control**, review the three retail marts files. Commit and push the reviewed lab changes to your personal remote branch using Studio's Git controls. **Commit and sync** may include all changed files: inspect the list first. If unrelated edits are present, ask dbt Wizard to commit only `models/marts/fct_retail_inventory_weekly.sql`, `models/marts/mart_retail_inventory_priorities.sql` and `models/marts/_retail_inventory__models.yml`, leaving every other file unchanged and uncommitted. Review the proposed file list before approving. In the verified lab, this selective commit was already published remotely; do not issue a separate push merely because the tool response says ‘Committed’. Verify the remote branch in the next step. This is the point in the lab where committing and syncing is required. Do not merge into or push directly to a protected/shared default branch.
3. Open **Go to repository**, sign in to GitHub with repository access if needed, and verify that your remote branch contains both SQL models and their YAML file, with no temporary `+ 7`. Record the exact branch name and commit SHA as `<DEPLOY_BRANCH>` and `<DEPLOY_COMMIT>`. A deployment job checks out remote Git code; it cannot use uncommitted Studio files. If repository policy prevents pushing your branch, resolve that with the instructor rather than changing branch protections.
4. Choose a unique `<LEARNER_ID>` using letters, digits and underscores, for example `JANE_DOE_01`. Record these values:

| Setting | Lab value |
|---|---|
| Environment name | `retail_deploy_<LEARNER_ID>` |
| Job name | `retail_build_<LEARNER_ID>` |
| Deployment database | Your existing `<DEV_DATABASE>` from Part 6.1 |
| `<DEPLOY_SCHEMA>` | A separate schema such as `<DBT_SCHEMA>_DEPLOY`; different from your Studio schema and other learners' schemas |
| Warehouse | `<LAB_WAREHOUSE>` |
| Source database / schema | `<LANDING_DATABASE>` / `<RETAIL_SCHEMA>` |
| Git branch | `<DEPLOY_BRANCH>` containing the pushed retail models |

5. Have the instructor provide or approve a **deployment connection profile** with the lab Snowflake role, authentication and target settings. It must read the three landing tables, use the lab warehouse, and create views in `<DEV_DATABASE>.<DEPLOY_SCHEMA>` and `<DEV_DATABASE>.<DEPLOY_SCHEMA>_STAGING`. The instructor must provision those schemas or the required schema-creation privileges. Do not reuse another learner's target, change a shared profile, or add grants yourself. Your personal Studio credentials are not automatically deployment credentials. Profiles are project-scoped and can be reused by other environments, so use an instructor-approved profile dedicated to your target. See [connection profiles](https://docs.getdbt.com/docs/platform/about-profiles).

**Create the General deployment environment**

1. In your lab project, select **Orchestration → Environments → Create environment**.
2. Under **Environment settings**, enter `retail_deploy_<LEARNER_ID>` as **Environment name**, select **Deployment** as **Environment type**, and explicitly choose **General** under **Set deployment type**. The inspected creation form defaulted to **Production**, so verify this choice before saving.
3. Confirm **dbt version** is **v2 Stable** (already selected in the inspected creation form). This environment setting is separate from your personal Studio override; do not assume the override carries over.
4. Select **Only run on a custom branch** and enter the exact `<DEPLOY_BRANCH>`. Otherwise deployment jobs use the repository's default branch, which may not contain your retail models.
5. In **dbt State**, select **Enable dbt State**. Account availability from Part 7.1 is still required; your personal State override does not configure this environment. New jobs can inherit this setting. See [environment State settings](https://docs.getdbt.com/docs/deploy/dbt-state-enable-env-jobs).
6. Under **Connection profiles**, click **Assign profile**, open **Profile**, select the instructor-approved profile, then click **Assign profile** in the drawer. Click its name to inspect **Profile details** before saving. Confirm the effective **Database**, **Schema**, **Warehouse** and **Role** match your recorded target; a profile with a different database or schema is not suitable merely because it is selectable.
7. If a dedicated profile is needed and the instructor has authorized its creation, open the **Profile** dropdown and choose **Add new profile**. Enter a unique **Profile name**, select the existing lab connection under **Connection**, and choose **Key pair** as the authentication method. Open the **1Password link in your lab welcome email** to retrieve your assigned key-pair credentials. Enter the username and private key directly in the dbt form (and its passphrase if required); do not share the link or credentials in chat or Git. Under **Deployment credentials**, set **Schema** to `<DEPLOY_SCHEMA>`; under **Connection overrides**, set **Database** to `<DEV_DATABASE>`, **Warehouse** to `<LAB_WAREHOUSE>` and the instructor-approved **Role**. Click **Test connection** and resolve any errors, then click **Create profile**; a successful test alone does not save the profile. Select the new profile in the assignment drawer and click **Assign profile**. Do not put secrets in **Extended attributes**, chat or Git. If credentials or profile-creation access are unavailable, ask the instructor to supply the profile. Do not modify a shared profile or account-wide connection.
8. Click **Save**. Open the new environment's **Settings** and verify **General**, **v2 Stable**, the custom branch, enabled State and assigned profile/target. Save any required profile assignment before creating the job. See [deployment environment setup](https://docs.getdbt.com/docs/deploy/deploy-environments).

**Create a manually triggered job**

1. From your new environment page, select **Create job → Deploy job**.
2. Set **Job name** to `retail_build_<LEARNER_ID>` and confirm **Environment** is your General environment.
3. Under **Execution settings → Commands**, replace the default unrestricted `dbt build` with the focused command below. Substitute your source values and enter it on one line:

```bash
dbt build --select +mart_retail_inventory_priorities --vars '{"source_database":"<LANDING_DATABASE>","source_schema":"<RETAIL_SCHEMA>"}'
```

4. Set **dbt State** to **Inherited from environment**. In **Advanced settings**, keep **dbt version** inherited from the environment and **Compare changes against** at **No deferral**. Keep **Run source freshness** off for this exercise. Set **Run timeout** to `300` seconds to bound the lab run.
5. Under **Triggers**, turn **Run on schedule** off and leave **Run when another job finishes** off. Do not add other automatic triggers. Save the job and recheck these settings. See [deploy job settings](https://docs.getdbt.com/docs/deploy/deploy-jobs).

**Run once and verify the deployment**

1. On the saved job's page, click **Run now** to launch one manual run ([manual runs](https://docs.getdbt.com/docs/deploy/job-scheduler)). Wait for the final run status; creating an environment or saving a job does not build models.
2. Open the run details. Confirm the environment is yours and the run's Git commit matches `<DEPLOY_COMMIT>`. Open the build step logs and verify a **2.x runtime**, **dbt State is enabled**, the focused selector and the expected source variables. Investigate failed steps before continuing.
3. Confirm the built relations target `<DEV_DATABASE>.<DEPLOY_SCHEMA>` for the marts and `<DEV_DATABASE>.<DEPLOY_SCHEMA>_STAGING` for staging. The job's **Target name** is a profile label, not a database selector. Record models/tests executed and reused separately; a successful reused test was not re-executed.
4. In a Snowsight worksheet using your lab role and warehouse, substitute your identifiers and run this bounded check against the deployment views:

```sql
select
  (select count(*) from <DEV_DATABASE>.<DEPLOY_SCHEMA>.FCT_RETAIL_INVENTORY_WEEKLY) as weekly_count,
  count(*) as latest_count,
  count(distinct product_store_key) as distinct_pairs,
  count_if(needs_replenishment) as replenishment_count
from <DEV_DATABASE>.<DEPLOY_SCHEMA>.MART_RETAIL_INVENTORY_PRIORITIES;
```

5. Compare with your restored Studio baseline: the verified dataset had **28,236 / 543 / 543 / 116**. If a model is missing, check the pushed branch/commit and target profile before rerunning. Confirm the job has no automatic triggers. Record its run URL and result.
6. Continue to Part 8 using the original Studio marts in `<DEV_DATABASE>.<DBT_SCHEMA>`. The deployment schema is a separate copy for this exercise.

> **Checkpoint — Your Deployment Job**
> - Your General environment uses your pushed branch, v2 Stable, enabled State and an isolated Snowflake target.
> - Your manually triggered job completed successfully, and its logs and aggregate check support the result.
> - Automatic triggers remain off; other learners' environments and the project's Staging/Production designations are unchanged.

> **Validation scope:** On October 9, 2026, the dedicated Key pair profile, General environment and manual deployment job were tested end to end. GitHub confirmed commit `908722ba679f715455c5210caffa90a70b12916f` on `angel_hernandez_dev`, containing only the three marts files; the unrelated `.gitignore` remained uncommitted. [Run 70403228201848](https://qn679.us1.dbt.com/deploy/70403103966603/projects/70403104003567/runs/70403228201848) succeeded in 21 seconds on dbt **2.0.8**, with State enabled: **5 models built, 0 tests executed, 29 test results reused, 0 failures and 0 warnings**. The marts were verified in `LAB_DB_TEST_X8622LFR_RV121YML_DEV.DBT_ANGEL_HERNANDEZ_DEPLOY_20261009`, with staging in the same database under the `_STAGING` schema suffix. A separate read-only aggregate check through dbt Wizard returned **28,236 / 543 / 543 / 116**. Exactly one manual deployment run was triggered; both automatic triggers remained off. Test reuse may vary with available State; it does not mean the tests ran again.

---

## Part 8: Ask Questions with Snowflake Cortex

You will create a **native Snowflake semantic view** over the tested latest-inventory mart and connect it to a **Cortex Agent** using a **Cortex Analyst** tool. The semantic view defines the meaning of the data; Analyst generates SQL; the agent explains the results in its built-in chat.

A semantic **fact** is a row-level value such as days of supply. A semantic **metric** applies an aggregation, such as a count or average. These are separate from the dbt SQL model named `fct_retail_inventory_weekly`.

### 8.1 Check the Snowflake prerequisites

1. Open Snowsight using your provisioned lab user and role.
2. Confirm the two marts exist in `<DEV_DATABASE>.<DBT_SCHEMA>` and the original baseline is restored.
3. Use the existing lab warehouse. Your user needs a usable default warehouse and role for the agent preview; the verified lab default was `TECH_FOUNDATIONS_HOL_WAREHOUSE`.
4. Confirm your lab role can create semantic views and agents in the provisioned `<DEV_DATABASE>.APPS` schema, query the mart and use Cortex (the verified lab role had `SNOWFLAKE.CORTEX_USER`). The instructor should provide these privileges; do not add grants during the exercise.

> **Usage:** dbt builds, SQL validation and agent chat can consume warehouse compute. Wizard and Cortex model calls can incur usage charges. Use the provisioned lab resources, the bounded examples below and the instructor's usage allowance.

### 8.2 Create the semantic view in Workspaces

1. In Snowsight, select **Projects → Workspaces**.
2. From the workspace home or **Add new** menu, choose **Semantic view**, then **Start blank**.
3. Select the **YAML** editor. Replace its contents with the definition below, substituting your `<DEV_DATABASE>` and `<DBT_SCHEMA>`.

```yaml
name: RETAIL_INVENTORY_ASSISTANT
description: Latest available retail inventory observations, one row per product and store. Supports replenishment prioritization from dbt-tested data. Historical lab data, not live inventory or a forecast.
tables:
  - name: inventory
    description: One latest observation per product/store, selected by inventory_date with row_id tie-breaking. Replenishment means days_of_supply is less than restock_lead_time_days.
    base_table:
      database: <DEV_DATABASE>
      schema: <DBT_SCHEMA>
      table: MART_RETAIL_INVENTORY_PRIORITIES
    primary_key:
      columns: [product_store_key]
    dimensions:
      - name: product_store_key
        expr: PRODUCT_STORE_KEY
        data_type: VARCHAR
      - name: product_id
        expr: PRODUCT_ID
        data_type: VARCHAR
      - name: product_name
        expr: PRODUCT_NAME
        data_type: VARCHAR
      - name: store_id
        expr: STORE_ID
        data_type: VARCHAR
      - name: store_name
        expr: STORE_NAME
        data_type: VARCHAR
      - name: category
        expr: CATEGORY
        data_type: VARCHAR
      - name: region
        expr: REGION
        data_type: VARCHAR
      - name: needs_replenishment
        description: True when days of supply is below restock lead time. A simple prioritization rule, not a forecast.
        expr: NEEDS_REPLENISHMENT
        data_type: BOOLEAN
      - name: is_anomaly
        expr: IS_ANOMALY
        data_type: BOOLEAN
      - name: anomaly_type
        expr: ANOMALY_TYPE
        data_type: VARCHAR
    time_dimensions:
      - name: inventory_date
        description: Date of the latest observation available for this product/store. Never assume it is today's date.
        expr: INVENTORY_DATE
        data_type: DATE
    facts:
      - name: days_of_supply
        expr: DAYS_OF_SUPPLY
        data_type: NUMBER
      - name: restock_lead_time_days
        expr: RESTOCK_LEAD_TIME_DAYS
        data_type: NUMBER
      - name: lead_time_gap_days
        description: Restock lead time minus days of supply; larger positive values indicate a larger replenishment gap.
        expr: RESTOCK_LEAD_TIME_DAYS - DAYS_OF_SUPPLY
        data_type: NUMBER
      - name: current_stock_level
        expr: CURRENT_STOCK_LEVEL
        data_type: NUMBER
      - name: weekly_sales_volume
        description: Units sold in the selected observation week, not revenue.
        expr: WEEKLY_SALES_VOLUME
        data_type: NUMBER
    metrics:
      - name: product_store_count
        expr: COUNT(*)
      - name: replenishment_count
        expr: COUNT_IF(needs_replenishment)
      - name: total_stock_units
        expr: SUM(current_stock_level)
      - name: selected_week_sales_units
        expr: SUM(weekly_sales_volume)
      - name: average_days_of_supply
        expr: AVG(days_of_supply)
module_custom_instructions:
  sql_generation: Include inventory dates when listing priorities. Count product/store combinations, not distinct products, for replenishment counts. Rank priorities by lead_time_gap_days descending with product_id and store_id as tie-breakers. Never use CURRENT_DATE to filter this historical dataset. Do not infer revenue or forecasts. This view contains only latest observations and cannot answer historical trend questions.
verified_queries:
  - name: replenishment_count
    question: How many product/store combinations need replenishment?
    sql: SELECT COUNT(*) AS replenishment_count FROM __inventory WHERE needs_replenishment = TRUE
  - name: highest_priority_three
    question: Which three product/store combinations have the largest replenishment gap?
    sql: SELECT product_id, product_name, store_id, store_name, inventory_date, days_of_supply, restock_lead_time_days, lead_time_gap_days FROM __inventory WHERE needs_replenishment = TRUE ORDER BY lead_time_gap_days DESC, product_id, store_id LIMIT 3
```

4. Click **Publish**. Enter `dev` as the **Target name**. This is a workspace publishing target label.
5. Choose **your development database → APPS**, verify the destination, then click **Publish**. The resulting object is `<DEV_DATABASE>.APPS.RETAIL_INVENTORY_ASSISTANT`.
6. Confirm **Changes have been published** and **Valid semantic view** with no errors. Resolve any missing relation or column errors before creating the agent. Snowflake may normalize the YAML names and data types after publishing.

The view deliberately exposes only the latest observation per product/store. It cannot answer historical trend questions. The verified SQL examples use the logical table name `__inventory`; Analyst resolves it to the physical mart.

### 8.3 Create and configure a draft agent

1. Select **AI & ML → Agent Studio → Create agent**.
2. Select `<DEV_DATABASE>.APPS`, set **Agent object name** to `RETAIL_INVENTORY_AGENT`, and **Display name** to `Retail Inventory Assistant`. Click **Create agent**.
3. Open **Configuration → General**. Set the description to:

```text
Explore latest available retail inventory observations and replenishment
priorities from dbt-tested models. Historical lab data; not live inventory
or a forecast.
```

4. Open **Configuration → Tools**. Turn off **Web search** and **Code Execution tool** if enabled; leave **Analytical search** off. This exercise needs only the inventory Analyst tool.
5. Under **Query structured data**, choose **Add semantic view → Add semantic view**.
6. In **Add tool: Cortex Analyst**, select your development database **and APPS schema**, then `RETAIL_INVENTORY_ASSISTANT`. Selecting only the database may leave the semantic-view selector disabled.
7. Set **Name** to `retail_inventory` and **Description** to:

```text
Query latest retail product/store observations, replenishment counts and
priorities, inventory units and selected-week sales units. Cannot answer
historical trends, revenue or forecasts.
```

8. Use **User's default** warehouse after verifying it is the lab warehouse, or choose **Custom** and select the instructor-provided warehouse. Set **Query timeout** to `60` seconds and click **Add**.
9. Open **Configuration → Instructions**. Leave **Model** as `auto`. Set **Orchestration instructions** to:

```text
Use RETAIL_INVENTORY for every quantitative inventory answer. Query current
results instead of relying on conversation memory. The mart has one latest
observation per product/store, not historical time series. Replenishment is
days_of_supply < restock_lead_time_days. Rank by lead_time_gap_days descending,
then product_id and store_id. Do not infer revenue, future demand or real-time
status.
```

10. Set **Response instructions** to:

```text
Give concise answers grounded in tool results. Include inventory dates with
priority lists and explain that this is historical lab data. State limitations
when a question cannot be answered by the semantic view.
```

11. For the small lab questions, set **Time Limit** to `120` seconds and **Token Limit** to `8000`. The UI may recommend a longer time limit; these bounds were sufficient for the verified examples, but latency can vary.
12. Click **Save**. Keep the agent as a **draft** and use its **Preview** tab. Publishing an agent version, granting access and adding it to Snowflake CoWork are not required for this exercise.

### 8.4 Test the chat against SQL

1. Open **Preview** and ask:

```text
How many product/store combinations need replenishment? Use the inventory
tool and show the observation date range.
```

2. Wait for the response. On the verified restored dataset, the answer was **116**, with an observation date range of **2023-12-24 to 2023-12-24**.
3. Click **Show Traces**, select **SQL Execution**, and inspect **Final SQL**, **SQL Results** and **Status**. Verify that the query reads your DEV mart, filters `needs_replenishment = TRUE`, and counts product/store rows rather than distinct products. A plausible chat answer alone is not validation.
4. Hide the traces, then ask:

```text
Which three product/store combinations have the largest replenishment gap?
Include product and store IDs, inventory date, days of supply, restock lead
time and gap.
```

5. Compare the answer with this bounded query in a Snowflake SQL file. Replace both placeholders and use the lab warehouse:

```sql
select
    product_id, product_name, store_id, store_name, inventory_date,
    days_of_supply, restock_lead_time_days,
    restock_lead_time_days - days_of_supply as lead_time_gap_days
from <DEV_DATABASE>.<DBT_SCHEMA>.MART_RETAIL_INVENTORY_PRIORITIES
where needs_replenishment = true
order by lead_time_gap_days desc, product_id, store_id
limit 3;
```

The verified SQL and chat agreed:

| Product | Store | Date | Days of supply | Lead time / gap |
|---|---|---|---:|---:|
| `PROD-0003` — TechPro TVs Premium | `STORE-C003` — Urban Center Store | 2023-12-24 | 0 | 29 / 29 |
| `PROD-0056` — GlowUp Fragrance Deluxe | `STORE-0014` — West Mall Outlet | 2023-12-24 | 0 | 27 / 27 |
| `PROD-C005` — Organic Produce Bundle | `STORE-0019` — Southeast Marketplace Outlet | 2023-12-24 | 0 | 26 / 26 |

6. Discuss a limitation: this semantic view cannot report a month-by-month trend or revenue because it contains only latest inventory observations and unit sales. Extending the use case would require a suitable model and semantic definition.

> **Checkpoint — Transformation to Cortex**
> - Your draft agent answers from a tested dbt mart through a native semantic view.
> - You inspected generated SQL and compared the results with a direct query.
> - The answer identifies the historical observation date and the limits of the replenishment rule.

### References

- [dbt Wizard in the platform](https://docs.getdbt.com/docs/platform/wizard-platform)
- [dbt State overview](https://docs.getdbt.com/docs/deploy/dbt-state-about) and [enable State in Studio](https://docs.getdbt.com/docs/deploy/dbt-state-enable-studio)
- [Snowflake semantic view YAML specification](https://docs.snowflake.com/en/user-guide/views-semantic/semantic-view-yaml-spec)
- [Create semantic views in Semantic Studio](https://docs.snowflake.com/en/user-guide/views-semantic/semantic-studio)
- [Manage Cortex Agents](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents-manage) and [agent setup prerequisites](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agents-setup)
