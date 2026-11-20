# Lake Compute sidecar + dbt Wizard CLI on a workstation: Hospitality

*November 2026. Fivetran Managed Data Lake Service on Amazon S3 and Snowflake, with dbt Fusion
and the dbt Wizard CLI on a provided Linux workstation running your models on Snowflake and
on Fivetran Lake Compute, its second adapter.*

Thank you for registering for our hands-on lab. This worksheet provides all the information
you need. Please read through the requirements first. If you can't meet them, let us know
and we'll rebook you for another workshop.

---

## Objective

Land the Hospitality data in a Managed Data Lake on Amazon S3, then build
**Guest transport review** with dbt Fusion on a Linux workstation, with two adapters in one
project: Snowflake stages the data as Iceberg tables, and Fivetran Lake Compute builds two
tested marts, generated with the dbt Wizard CLI, next to your lake. Finish by
querying every layer from Snowflake.

## Requirements

- Web browser (Chrome)
- No local software installation is required.
   - Fivetran and Snowflake run in your browser. dbt Fusion, the dbt Wizard CLI and Git are
     pre-installed on the Linux workstation provided for this lab. Wizard is already
     configured to use the lab's AI model; you don't sign in to it or provide a key.

## Housekeeping

- You will receive an invitation to the **MDLS_HANDS_ON_LAB** Fivetran account.
- **Do not set up a new account or trial** — you will receive an invite to a dedicated
  Fivetran account prior to the lab.
- Confirm that you have received the invite. It is sent to the email address you used
  to sign up for this lab and originates from `notifications@fivetran.com`.
- You'll be provided with a **Linux workstation** accessible in your web browser. The
  **Gateway URL**, **Guacamole Username** and **Guacamole Password** are in your 1Password link.

## What you'll do in this lab

1. Set up a Managed Data Lake destination on Amazon S3, with Lake Compute turned on
2. Set up a connector to send the Hospitality data to the data lake
3. Connect Snowflake to your lake through a catalog-linked database
4. Create your Lake Compute token
5. Log in to your workstation, add your Snowflake key and your token, and clone your dbt project
6. Stage the data on Snowflake as Iceberg tables
7. Generate two tested marts with the dbt Wizard CLI and build them on Lake Compute
8. Query the staging tables and the marts from Snowflake

---

## Part 0: Credentials

All of these are provided by your instructor via a 1Password link.

### Snowflake

- **Snowflake Hostname**, **Snowflake User** and **Snowflake Password**, for signing in to
  the Snowflake web UI.
- **Snowflake Private Key**: dbt signs in to Snowflake with this key pair, not with your
  password. You save it on your workstation in Part 7.
- **Snowflake Role** and **Snowflake Warehouse**.
- **Snowflake Dev Database**: where your dbt models are built. If your link also lists a
  **Snowflake Prod Database**, this lab doesn't use it.

### Postgres

- The source database credentials.

### Amazon S3

- **S3 Bucket**, **S3 Region**, **S3 Role ARN** and **S3 Prefix Path**.
   - The role and the prefix are yours alone: the role can only read and write under your
     prefix in the lab bucket.

### Workstation and dbt project

- **Gateway URL**, **Guacamole Username** and **Guacamole Password** for your workstation.
- **Git Clone Command** for your own copy of the Hospitality dbt project, a private GitHub
  repository named `lab-dbt-...` that only your workstation can reach. The command
  clones it into `~/lab-dbt`.

### Fivetran destination

- **Fivetran Destination**: the name of the destination created for you when the lab
  was launched. It starts with `lbs_mdls_`. You open it in Part 2, and it is the only
  destination you set up in this lab.

---

## Part 1: Accessing the provided resources

1. Log in to Fivetran at `https://fivetran.com/login`.
   - If you just signed up for the first time, use the username and password you used
     for your sign-up.
   - If you already have a Fivetran account, use those credentials.
2. Switch to the `MDLS_HANDS_ON_LAB` account via the drop-down menu in the top left.
3. Log in to Snowflake.
   - Open the **Snowflake Hostname** from your 1Password link.
   - Enter your **Snowflake User** and **Snowflake Password** and click **Sign in**.

   > **MFA enrollment:** Snowflake requires multi-factor authentication for password
   > sign-ins. The first time you sign in, Snowflake will prompt you to enroll and
   > offer several options. We recommend choosing **Passkey**. Complete the enrollment
   > before continuing.

**Checkpoint:** You are signed in to Fivetran and Snowflake, each in its own browser tab.
Keep both open — you will move between them and your workstation.

---

## Part 2: Setting up the data lake destination

We'll begin by setting up a Managed Data Lake destination.

1. Navigate to the **Destinations** tab in the left-hand navigation.
2. Find the destination named in the **Fivetran Destination** field of your 1Password
   link (it starts with `lbs_mdls_`). It was created for you when the lab was launched
   and shows as **Incomplete** until you finish setting it up. Click it.

   > **Make sure it's yours.** Your destination is the only one that has been given
   > access to your S3 role and prefix; anyone else's will fail its setup tests.

3. Three Fivetran Managed Data Lake Service options appear. Select the
   **S3 Data Lake** option by clicking its **Set up** button.
4. For **Bucket name**, enter the **S3 Bucket** value from your 1Password link.
5. For **Fivetran Role ARN**, enter the **S3 Role ARN** value from your 1Password link.
6. For **S3 Prefix Path**, enter the **S3 Prefix Path** value from your 1Password link.
   Use it exactly as given: your role can only write under this prefix, so any other value
   fails the setup tests.

   > **Note — this field is immutable.** The prefix path cannot be changed after the
   > destination is created. The same is true of the bucket and the region. Getting it
   > wrong means recreating the destination, so check your spelling now.

7. In **S3 Bucket Region**, select the **S3 Region** value from your 1Password link
   (usually `us-west-2`). It must match the bucket's actual region. Lake Compute runs in
   `us-east-1` and `us-west-2` only.
8. Leave **Update AWS Glue Catalog** and **Update Databricks Unity Catalog** off. Snowflake
   reads the **Fivetran Catalog** directly.
9. Turn on **Enable Lake Compute** (Beta). This lets dbt run models on the Iceberg data in
   this lake and write them back to it. You'll create your token in Part 5.
10. Leave **Write Delta Format** off and the remaining options at their defaults.
11. Click **Save & Test**.

   > If the AWS access test fails, check that you opened your own destination and that
   > the bucket, role ARN and prefix match your 1Password link, then confirm with your
   > instructor that your destination was given access to your role.

12. Let the setup tests run. For S3 these are **Input Validation**, **AWS Read and
    Write Access**, **Fivetran Catalog Access**, **Bucket Region Test** and optionally
    **S3 Private Link Test**.
13. Once the tests complete successfully, click **View Destination**.

**Checkpoint:** All setup tests show green, and your destination appears in the
Destinations list under the name in your **Fivetran Destination** field. Every Managed
Data Lake destination is provisioned with the **Fivetran Catalog** automatically. Snowflake
and Lake Compute both use it.

---

## Part 3: Creating a connector

Now we'll create a connector to sync the Hospitality data into your managed data lake in S3.

1. Navigate to the **Connections** tab.
2. Click **Add Connection**.
3. Search for **PostgreSQL** and click **Set up**.
4. Select your destination (the one from Part 2).
   - **Make sure you don't use someone else's destination** — check that the name matches your **Fivetran Destination** field.
5. Select **Fivetran naming**. The name you provide in the **Destination schema prefix** field becomes the name of
   your connector. Use the format `<firstname>_<lastname>_pg`.

   > Write this value down. Fivetran combines it with the source schema name to form the
   > schema your dbt project reads: `<firstname>_<lastname>_pg_hospitality`.

6. Populate the setup form with the provided PostgreSQL credentials.
7. For **Authentication Method**, select **Connect with username and password**.
8. For **Connection method**, make sure **Connect directly** is selected.
9. For **Update Method**, select **Query-Based**.
   - *Context:* PostgreSQL's older XMIN and Fivetran Teleport Sync methods have been sunset and replaced by **Query-Based** change data capture. The other available method is **Logical replication** (using the `pgoutput` plugin).
10. Click **Save & Test**.
11. Confirm the TLS certificate when prompted.
12. Wait for the setup tests to complete, then click **Continue**.
13. Fivetran now fetches all tables, schemas, and columns for the provided database.
14. Make sure the `hospitality` schema and all of its tables are selected.
15. Click **Save & Continue**.
16. For handling schema changes, select **Allow all**.
17. Click **Start Initial Sync**.
18. Wait for the sync to complete.

**Checkpoint:** The connection shows a completed initial sync of the `hospitality` schema.
Fivetran has written each table to
`s3://<S3 Bucket>/<S3 Prefix Path>/<firstname>_<lastname>_pg_hospitality/<table>/`
(for example `.../hpt_records/`) as Parquet files with Iceberg metadata.

---

## Part 4: Connect Snowflake to your lake

Snowflake reads the Fivetran Catalog through a **catalog integration** and a
**catalog-linked database**. Your dbt project reads its sources from that database, and Lake
Compute uses the same catalog integration to make the marts it builds visible in Snowflake.

### 4.1 Collect the catalog values from Fivetran

1. In Fivetran, open **Destinations** and click your destination.
2. Click the **Catalog integration** tab, then the **Snowflake** tab. Fivetran generates a
   complete `CREATE CATALOG INTEGRATION` statement with your values filled in.

   > **The client secret is displayed once.** Copy the whole statement now. Regenerating
   > the secret immediately breaks any catalog integration that uses the old one.

### 4.2 Create the catalog integration and your landing database

1. In Snowflake, click **Create → SQL Worksheet** and name it `<firstname>_<lastname>_lake_compute`.
2. Paste the statement from Fivetran. In it:
   - Replace `{fivetran_delivered_schema_name}` with your connector's schema,
     `<firstname>_<lastname>_pg_hospitality`.
   - Rename the database in the `CREATE DATABASE` statement to your **landing database**:
     your **Snowflake Dev Database** with `_DEV` replaced by `_LANDING`. For example, if your
     Dev database is `LAB_DB_ACME_AB12CD34_JDOE_DEV`, create `LAB_DB_ACME_AB12CD34_JDOE_LANDING`.

   > **Why this name?** The dbt project finds its sources in the database whose name matches
   > its target with `_DEV` swapped for `_LANDING`. Your workstation's profile targets your
   > Dev database, so with that name the project finds your sources without any change.

   The statement looks like this. **Do not paste this example; it will not work.**

   ```sql
   CREATE OR REPLACE CATALOG INTEGRATION fivetran_catalog_mystify_velvet
     CATALOG_SOURCE = POLARIS
     TABLE_FORMAT = ICEBERG
     CATALOG_NAMESPACE = 'jane_doe_pg_hospitality'
     REST_CONFIG = (
       ACCESS_DELEGATION_MODE = VENDED_CREDENTIALS,
       CATALOG_URI = 'https://<your-catalog-host>/api/catalog'
       WAREHOUSE = 'mystify_velvet'
     )
     REST_AUTHENTICATION = (
       TYPE = OAUTH
       OAUTH_TOKEN_URI = 'https://<your-catalog-host>/api/catalog/v1/oauth/tokens'
       OAUTH_CLIENT_ID = 'redacted'
       OAUTH_CLIENT_SECRET = 'redacted'
       OAUTH_ALLOWED_SCOPES = ('PRINCIPAL_ROLE:ALL')
     )
     ENABLED = TRUE,
     REFRESH_INTERVAL_SECONDS = 30;

   CREATE DATABASE LAB_DB_ACME_AB12CD34_JDOE_LANDING
     LINKED_CATALOG = (
       CATALOG = 'fivetran_catalog_mystify_velvet'
     );
   ```

3. Run both statements.
4. Check that Snowflake found your tables:

   ```sql
   SHOW ICEBERG TABLES IN DATABASE <your landing database>;
   ```

**Checkpoint:** Your Hospitality tables, including `hpt_records`, are listed. Keep the
catalog integration: Lake Compute needs it in Part 11.

---

## Part 5: Create your Lake Compute token

Lake Compute runs your dbt models next to your data lake and writes the results back to it
as Iceberg tables. dbt authenticates to it with a compute token that belongs to you.

1. In Fivetran, open **Destinations** and click your destination.
2. Open the **Compute** tab. Check that **Enable Lake Compute** is on; turn it on if it isn't.
3. Under **Compute Token**, click **+ Generate Token** (or **Add new compute token** if you
   have none yet).
4. In **Generate new compute token**:
   - **Token name:** `<firstname>_<lastname>_workstation`
   - Keep **User-scoped token** selected. It is tied to your user and stops working if your
     access to this destination is removed, for example when the lab is cleaned up.
   - **Expiration:** the default is **Does not expire**. Change it to an option that ends
     no later than the lab's end date.
5. Click **Generate**, then copy the token right away and keep it somewhere private until
   Part 7. Don't paste it into a file in your repository, a chat or a shared document.

**Checkpoint:** Your token appears in the **Compute Token** list, and you have copied it.

---

## Part 6: Log in to your workstation

1. Open the **Gateway URL** from your 1Password link and sign in with your
   **Guacamole Username** and **Guacamole Password**.
2. Once you have logged in, click the **Terminal** icon on the bottom menu.

   > **Tips:** In the workstation terminal, paste with **CTRL + SHIFT + V**. If text copied
   > in your own browser pastes as nothing, press **CTRL + ALT + SHIFT** to open the
   > Guacamole clipboard panel, paste the text into its box, close the panel and paste
   > again in the terminal.

3. Confirm dbt Fusion and the dbt Wizard CLI are there:

   ```bash
   dbtf --version
   wizard --version
   ```

   - `dbtf` is **dbt Fusion** (dbt v2), the only dbt that can run Lake Compute. Plain `dbt`
     on this workstation is dbt Core 1.x and cannot read this lab's profile. Every dbt
     command in this lab uses `dbtf`.
   - If dbt prints **New version available**, ignore it. Don't run `dbt system update`; the
     workstation's version is the one this lab was tested with.

---

## Part 7: Add your Snowflake key and your token

dbt signs in to Snowflake with your key pair, and Lake Compute writes its results back to
Snowflake as you. LabStudios never puts your private key or your token on the workstation:
you add both yourself.

1. Check that your terminal has the two Lake Compute settings. They come from a block
   LabStudios added to your `~/.bashrc`, so every new terminal has them:

   ```bash
   env | grep -E 'DBT_ENGINE_EXPERIMENTAL_MULTI_ADAPTER|DBT_ALLOW_EXPERIMENTAL_ADAPTERS'
   ```

   Both must show `=true`. They turn on Lake Compute, which is an experimental adapter in
   dbt Fusion. If either is missing, run `source ~/.bashrc` and check again; if they are
   still missing, ask your instructor.

2. Create the key file, readable only by you, and open it in an editor:

   ```bash
   install -m 600 /dev/null ~/.dbt/snowflake_key.p8
   nano ~/.dbt/snowflake_key.p8
   ```

3. Paste the **Snowflake Private Key** from your 1Password link in full, including the
   `-----BEGIN PRIVATE KEY-----` and `-----END PRIVATE KEY-----` lines. Save with
   **CTRL + O** and **Enter**, then leave with **CTRL + X**.
4. Check the file without printing the key:

   ```bash
   stat -c '%a' ~/.dbt/snowflake_key.p8
   head -n 1 ~/.dbt/snowflake_key.p8
   ```

   The first line must print `600`, the second `-----BEGIN PRIVATE KEY-----`.

5. Set your Lake Compute token for this terminal. The command prompts for it and doesn't
   echo it or save it in your shell history. Paste the token from Part 5 and press **Enter**:

   ```bash
   read -rsp "Lake Compute token: " LAKE_COMPUTE_TOKEN && export LAKE_COMPUTE_TOKEN && echo
   ```

6. Check it is set, without printing it:

   ```bash
   [ -n "$LAKE_COMPUTE_TOKEN" ] && echo "token is set"
   ```

   > The token lives only in this terminal. If you open a new terminal, run step 5 again.
   > Never put the token or the key in your repository, a commit or a prompt.

---

## Part 8: Clone your dbt project

LabStudios created a private copy of the Hospitality dbt project for you, and gave your
workstation a key that can read and write it.

1. In the terminal, go to your home directory and run the **Git Clone Command** from your
   1Password link:

   ```bash
   cd ~
   <paste your Git Clone Command here>
   ```

2. Change into the project:

   ```bash
   cd ~/lab-dbt
   ```

3. Look around:

   ```bash
   ls models/staging
   cat dbt_project.yml
   ```

   `models/staging` holds one staging model per Hospitality table. This lab uses
   `stg_hospitality__hpt_records`.

---

## Part 9: Look at your `profiles.yml`

Your workstation already has a dbt profile at `~/.dbt/profiles.yml`, written when the lab
was launched with your own Snowflake values. Look, but don't change it:

```bash
cat ~/.dbt/profiles.yml
```

It looks like this, with your own values:

```yaml
default:
  target: default
  outputs:
    default:
      - name: snowflake
        type: snowflake
        default: true
        method: keypair
        account: "<your account>"
        user: "<your Snowflake User>"
        role: "<your Snowflake Role>"
        warehouse: "<your Snowflake Warehouse>"
        database: "<your Snowflake Dev Database>"
        schema: "dbt_<your suffix>"
        private_key_path: "/home/<you>/.dbt/snowflake_key.p8"
        threads: 4
      - name: lakecompute
        type: lakecompute
        method: fivetran
        fivetran_credential: "{{ env_var('LAKE_COMPUTE_TOKEN') }}"
        base_url: "https://api.lake-compute.fivetran.com/"
```

- The output is a **list** of two adapters. `snowflake` has `default: true`: every model runs
  on Snowflake unless it says otherwise.
- `method: keypair` and `private_key_path` point dbt at the key file you created in Part 7.
- `lakecompute` is the second adapter. It has no database or schema of its own: it writes to
  the same database and schema as Snowflake.
- `fivetran_credential` reads your token from the environment variable you set in Part 7.
  The token itself is never written to this file.

Write down your schema; this worksheet calls it `<DBT_SCHEMA>`:

```bash
grep schema ~/.dbt/profiles.yml
```

Marts land in `<DBT_SCHEMA>` and staging models in `<DBT_SCHEMA>_staging`, both in your
**Snowflake Dev Database**.

---

## Part 10: Configure the project

In this lab, each adapter does what it is best at:

| Layer | Runs on | Materialized as |
|---|---|---|
| Staging models | Snowflake, the default adapter | Iceberg tables |
| Fact and summary marts | Lake Compute (`adapter='lakecompute'`) | Iceberg tables |

Lake Compute reads and writes Iceberg tables only, so the staging layer it builds on must be
Iceberg, not views. Snowflake writes its Iceberg tables to the lab's storage, which is
already set up on your Dev database.

1. Open the project in an editor (`nano dbt_project.yml`, or `code .` for VS Code if
   available on your workstation).
2. Under `vars`, set `source_schema` to your connector's schema,
   `<firstname>_<lastname>_pg_hospitality`, and add `review_threshold: 30`, which your marts
   will use. Leave `source_database` commented out: the project finds your landing database
   from Part 4 by itself.
3. Under `models: hospitality_lab:`, change `staging` to `+materialized: table`, add
   `+table_format: iceberg`, keep `+schema: staging`, and add `marts` with
   `+materialized: table`.

The result should look like this, with your schema in place of the placeholder:

```yaml
name: 'hospitality_lab'
version: '1.0.0'
config-version: 2
profile: 'default'

model-paths: ['models']
test-paths: ['tests']
macro-paths: ['macros']

vars:
  source_schema: '<firstname>_<lastname>_pg_hospitality'
  review_threshold: 30

models:
  hospitality_lab:
    staging:
      +materialized: table
      +table_format: iceberg
      +schema: staging
    marts:
      +materialized: table
```

---

## Part 11: Check the connection and stage the data on Snowflake

Run every command from inside `~/lab-dbt`, in a terminal where your token is set (Part 7).

1. Install any packages and parse the project:

   ```bash
   dbtf deps
   dbtf parse
   ```

2. Test both adapters:

   ```bash
   dbtf debug
   ```

   Read the checks. All of these must be **OK**:
   - **snowflake connection test**
   - **lakecompute connection test**
   - **MDLS write test** and **MDLS read-back test**: Lake Compute can write to your lake
   - **Snowflake propagation test**: what Lake Compute writes becomes visible in Snowflake,
     through the catalog integration you created in Part 4

   The output ends with **All checks passed!**

3. Build the staging models this exercise uses:

   ```bash
   dbtf build --select stg_hospitality__hpt_records
   ```

4. The build must finish successfully. A zero-row build is not evidence of a populated
   pipeline: if a model is empty, check the Fivetran sync status from Part 3.

**Checkpoint:** `dbtf debug` passes every check, and
`stg_hospitality__hpt_records` is an Iceberg table in your Dev database, in the schema
`<DBT_SCHEMA>_staging`.

---

## Part 12: Build the marts with the dbt Wizard CLI

**Business question:** Guest transport review. These are guest recommendation observations, not actual trip durations.

Use these files:

| File | Purpose |
|---|---|
| `models/marts/fct_hospitality_observations.sql` | Row-level fact table, built on Lake Compute |
| `models/marts/mart_hospitality_review_summary.sql` | Business-facing mart table, built on Lake Compute |
| `models/marts/_hospitality__addon_models.yml` | Model descriptions and generic data tests |
| `tests/assert_hospitality_addon.sql` | Singular reconciliation test; returns rows only when an invariant fails |

**Model requirements**

- Build `fct_hospitality_observations` at one row per `hpt_records.record_id`. Use `ref('stg_hospitality__hpt_records')` and select the source key as `observation_id`.
- Derive `group_key` from `event_category`: use `missing:` for null and `value:` plus its string value otherwise. Keep `group_name` as the display string, using `Unknown` for null. Distinguish a real value named Unknown from a missing value through the key.
- Calculate `metric_value` from `transportation_eta_minutes` using safe numeric conversion. Convert arithmetic inputs before calculating. Division by zero must produce null. This measures estimated transport time in minutes.
- Preserve `synced_at`. Set `requires_review` to `metric_value >= review_threshold`, initially 30. Missing metrics and their flags stay null; never silently turn them into zero or false.
- Build `mart_hospitality_review_summary` from the fact at one row per `group_key`. Include group name, total observations, measured observations, missing metric count, review count, unweighted mean metric and review share. Divide review count by measured observations with a zero-safe denominator and decimal arithmetic.
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
Inspect the existing hospitality staging models and source definitions first.
Create only these four files:
models/marts/fct_hospitality_observations.sql
models/marts/mart_hospitality_review_summary.sql
models/marts/_hospitality__addon_models.yml
tests/assert_hospitality_addon.sql

This project uses two adapters. The staging models run on Snowflake as Iceberg
tables. Both marts run on Fivetran Lake Compute: start each mart with
{{ config(adapter='lakecompute', materialized='table') }} and write DuckDB SQL
with ref() dependencies and built-in dbt tests.
Lake Compute does not support views, pre- or post-hooks, grants or unit tests.
Numeric source columns can arrive in the lake as text: convert them with
try_cast(... as double) before arithmetic or comparisons, and use try_cast or
try_strptime for dates and booleans.
Follow the Model requirements immediately above (paste them with this prompt).
Read review_threshold from dbt_project.yml with var('review_threshold').
Preserve existing staging, sources, project configuration and packages.
Document grains, units, null handling and the limits of the review rule.
Propose the files and explain their logic before editing. Do not run commands,
change settings, commit or push. Do not request credentials or tokens.
```

4. Review the proposed joins, grain, expressions, tests and file paths. Check that each mart
   starts with `{{ config(adapter='lakecompute', materialized='table') }}` and that numeric
   text columns are converted before they are compared. Ask Wizard to correct discrepancies
   before it edits anything.
5. When Wizard asks to create or change files, approve only the four exercise files. Decline
   any other edit, and any command it proposes to run: you run dbt yourself in Part 13.
6. Leave the session (type `/` to see Wizard's commands, or press **CTRL + C**), then review
   what changed:

   ```bash
   git status
   git diff
   ```

   Only `dbt_project.yml` (your changes from Part 10) and the four new files should appear.
   Restore anything else with `git checkout -- <file>`.

7. See how the models depend on each other before building them:

   ```bash
   dbtf ls --select +mart_hospitality_review_summary --resource-type model
   ```

> **How the two adapters share data.** Your staging models are Iceberg tables that Snowflake
> wrote to the lake. `dbtf` compiles each mart on the workstation and sends it to Lake
> Compute, which reads the staging tables there, runs your mart SQL next to the data and
> writes each mart back as an Iceberg table. The catalog integration from Part 4 then makes
> each mart visible in Snowflake, in the same database and schema a Snowflake model would
> use. Lake Compute supports `table`, `incremental` and `ephemeral` models and data tests;
> it does not support views, pre- or post-hooks, grants or unit tests.

---

## Part 13: Build and validate

1. Inspect the resolved targets before building:

   ```bash
   dbtf ls --select +mart_hospitality_review_summary --resource-type model --output json --output-keys name database schema alias
   ```

   Every model should target your **Snowflake Dev Database**. The staging models use
   `<DBT_SCHEMA>_staging` and the two marts use `<DBT_SCHEMA>`.

2. Build the mart, its ancestors and their tests:

   ```bash
   dbtf build --select +mart_hospitality_review_summary
   ```

3. Require a successful final result. Read failing test output and repair the model or
   assumptions; don't remove tests just to get a green run. You can ask Wizard to help
   diagnose a failure. Check the singular
   reconciliation test ran. If it didn't, run it explicitly:

   ```bash
   dbtf test --select assert_hospitality_addon
   ```

4. Run an aggregate check through dbt and record the result:

   ```bash
   dbtf show --inline "select coalesce(sum(observation_count), 0) as observations, coalesce(sum(measured_count), 0) as measured, coalesce(sum(missing_metric_count), 0) as missing, coalesce(sum(review_count), 0) as flagged from {{ ref('mart_hospitality_review_summary') }}" --limit 1
   ```

   Don't expect fixed numbers: each lab's sync state can differ.

5. Optional: save your work to your repository.

   ```bash
   git add dbt_project.yml models/marts tests
   git commit -m "Add hospitality marts on Lake Compute"
   git push
   ```

**Checkpoint:** The marts and their tests pass on Lake Compute, and you have your aggregate
result.

---

## Part 14: Query your models from Snowflake

Every model you built is a table in Snowflake, whichever adapter built it.

1. In your Snowflake worksheet, look at the staging table Snowflake built:

   ```sql
   SHOW ICEBERG TABLES IN SCHEMA <your Dev database>.<DBT_SCHEMA>_STAGING;
   ```

2. Query the mart Lake Compute built:

   ```sql
   SELECT * FROM <your Dev database>.<DBT_SCHEMA>.MART_HOSPITALITY_REVIEW_SUMMARY LIMIT 10;
   ```

3. Repeat the aggregate check from Part 13, this time from Snowflake. The result should
   match what `dbtf show` returned:

   ```sql
   select coalesce(sum(observation_count), 0) as observations, coalesce(sum(measured_count), 0) as measured, coalesce(sum(missing_metric_count), 0) as missing, coalesce(sum(review_count), 0) as flagged from <your Dev database>.<DBT_SCHEMA>.MART_HOSPITALITY_REVIEW_SUMMARY;
   ```

**Checkpoint:** Snowflake reads the staging tables it built and the marts Lake Compute
built, in your Dev database.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| **AWS Read and Write Access** test fails | You opened someone else's destination, or the bucket, prefix or role ARN was mistyped | Check that the destination name matches your **Fivetran Destination** field and the values match your 1Password link, then re-run the test; if it still fails, ask the instructor to confirm your destination was given access to your role |
| `SHOW ICEBERG TABLES` returns nothing | The namespace in the catalog integration doesn't match your connector's schema | It is the connector name **+ source schema**: `<firstname>_<lastname>_pg_hospitality` |
| `dbt` says the profile is misconfigured or doesn't know `lakecompute` | You ran `dbt` (dbt Core 1.x) | Use `dbtf` |
| dbt refuses to run Lake Compute | The two settings from Part 7 are missing in this terminal | Run `source ~/.bashrc` and check them again (Part 7, step 1) |
| Snowflake sign-in fails with a key-pair or private key error | The key file is missing, incomplete or not `600` | Recreate `~/.dbt/snowflake_key.p8` from the whole **Snowflake Private Key**, including the `BEGIN` and `END` lines (Part 7) |
| `dbtf debug` fails the lakecompute connection test | The token is not set in this terminal, or it was copied incompletely | Run Part 7, step 5 again; generate a new token if needed |
| `dbtf debug` fails a write test with `Schema ... does not exist` | Nothing has been built in your schema yet | Build the staging models (Part 11, step 3), then run `dbtf debug` again |
| `dbtf debug` fails the Snowflake propagation test | The catalog integration from Part 4 is missing or its secret was regenerated | Recreate it from the current statement in **Catalog integration → Snowflake** |
| A source table is not found | The landing database is not named after your Dev database, or `source_schema` is wrong | Name it like your Dev database with `_DEV` replaced by `_LANDING` (Part 4), and set `source_schema` to `<firstname>_<lastname>_pg_hospitality` |
| `501: Views are not supported on Lake Compute` | A Lake Compute model, or a model it reads, is a view | Make the staging layer an Iceberg table (Part 10) and the marts tables |
| Wizard writes a view, Snowflake SQL, or a mart without the Lake Compute adapter | Wizard used another dialect or materialization | Ask it to start each mart with `{{ config(adapter='lakecompute', materialized='table') }}` and use DuckDB SQL, as the prompt says, then review the diff again |
| `wizard` can't find the project or its models | `dbtf parse` hasn't run, or you started Wizard outside the project | Run `dbtf parse` in `~/lab-dbt`, then start `wizard` there |
| Wizard reports that the profile is invalid or doesn't know `lakecompute` | Wizard ran dbt Core (`dbt`), which cannot read this profile | Decline the command and run dbt yourself with `dbtf`. Wizard is set up to use `dbtf` for `~/lab-dbt` (`~/.dbt/wizard/wizard_config.toml`); if that file doesn't list it, ask your instructor |
| Conversion or comparison errors in a mart | A numeric source column arrived as text | Convert it with `try_cast(... as double)` before using it |

---

## What you did

You configured a Managed Data Lake Service destination on Amazon S3 with Lake Compute
turned on, synced the Hospitality data into it as Apache Iceberg and connected Snowflake to
it through a catalog-linked database. On a Linux workstation, one dbt Fusion project used
two adapters: Snowflake staged the data as Iceberg tables, and the dbt Wizard CLI generated
two tested marts that Fivetran Lake Compute built next to the lake. You read every layer
from Snowflake.

## References

- [Inspected industry template](https://github.com/dbt-Hands-on-Labs/dbt-hospitality-lab)
- Lake Compute — `docs.getdbt.com/docs/lake-compute`
- Managed Data Lake Service — `fivetran.com/docs/managed-data-lake-service`
- Query Iceberg tables from Snowflake — `fivetran.com/docs/managed-data-lake-service/tutorials`
- [Use dbt Wizard locally](https://docs.getdbt.com/docs/dbt-ai/wizard-quickstart)
- [Manual edition of this worksheet](../../manual/hospitality/lake_compute_sidecar_workstation_hands_on_lab.md)
- [Lake Compute sidecar on dbt Platform with dbt Wizard, the same exercise in your browser](../../../dbt_platform/wizard/hospitality/lake_compute_dbt_platform_hands_on_lab.md)
- [Lake Compute on a workstation with the dbt Wizard CLI, the same exercise without a warehouse](../../../workstation/wizard/hospitality/lake_compute_workstation_hands_on_lab.md)
