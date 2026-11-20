# Lake Compute on a workstation: Hospitality

*November 2026. Fivetran Managed Data Lake Service on Amazon S3, with dbt Fusion running your
models on Fivetran Lake Compute from a provided Linux workstation. No warehouse is involved.*

Thank you for registering for our hands-on lab. This worksheet provides all the information
you need. Please read through the requirements first. If you can't meet them, let us know
and we'll rebook you for another workshop.

---

## Objective

Land the Hospitality data in a Managed Data Lake on Amazon S3, then build **Guest transport review**
on Fivetran Lake Compute with dbt Fusion: two tested marts, written back to your lake as
Iceberg tables. Finish by querying the results with the DuckDB CLI, straight from the lake.

## Requirements

- Web browser (Chrome)
- No local software installation is required.
   - Fivetran runs in your browser. dbt Fusion, Git and DuckDB are pre-installed on the
     Linux workstation provided for this lab.

## Housekeeping

- You will receive an invitation to the **MDLS_HANDS_ON_LAB** Fivetran account.
- **Do not set up a new account or trial** — you will receive an invite to a dedicated
  Fivetran account prior to the lab.
- Confirm that you have received the invite. It is sent to the email address you used
  to sign up for this lab and originates from `notifications@fivetran.com`.
- You'll be provided with a **Linux workstation** accessible in your web browser. The
  **Gateway URL**, **Guacamole Username** and **Guacamole Password** are in your 1Password link.

## What you'll do in this lab

1. Set up a Managed Data Lake destination on Amazon S3
2. Set up a connector to send the Hospitality data to the data lake
3. Turn on Lake Compute and create your Lake Compute token
4. Log in to your workstation, clone your dbt project and configure dbt Fusion
5. Build the staging models and two tested marts on Lake Compute
6. Query the results with the DuckDB CLI

---

## Part 0: Credentials

All of these are provided by your instructor via a 1Password link.

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

**Checkpoint:** You are signed in to the `MDLS_HANDS_ON_LAB` Fivetran account.

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
8. Leave **Update AWS Glue Catalog** and **Update Databricks Unity Catalog** off.
9. Turn on **Enable Lake Compute** (Beta). This lets you run dbt projects on the Iceberg data
   in this lake and write your models back to it. You'll create your token after the
   destination is saved, in Part 4.
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
Data Lake destination is provisioned
with the **Fivetran Catalog** automatically. Lake Compute and DuckDB both use it.

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

## Part 4: Create your Lake Compute token

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
   Part 6. Don't paste it into a file in your repository, a chat or a shared document.
6. Open the **Catalog Integration** tab and stay on **Base configuration**. Copy the
   **Catalog** value. This is your destination's **group ID**: dbt uses it as the
   `database` in your profile, and DuckDB uses it as the catalog name in Part 13.

**Checkpoint:** Your token appears in the **Compute Token** list, and you have the token
and your group ID.

---

## Part 5: Log in to your workstation

1. Open the **Gateway URL** from your 1Password link and sign in with your
   **Guacamole Username** and **Guacamole Password**.
2. Once you have logged in, click the **Terminal** icon on the bottom menu.

   > **Tips:** In the workstation terminal, paste with **CTRL + SHIFT + V**. If text copied
   > in your own browser pastes as nothing, press **CTRL + ALT + SHIFT** to open the
   > Guacamole clipboard panel, paste the text into its box, close the panel and paste
   > again in the terminal.

3. Confirm the tools are there:

   ```bash
   dbtf --version
   duckdb --version
   ```

   - `dbtf` is **dbt Fusion** (dbt v2), the only dbt that can run Lake Compute. Plain `dbt`
     on this workstation is dbt Core 1.x and cannot. Every dbt command in this lab uses `dbtf`.
   - If dbt prints **New version available**, ignore it. Don't run `dbt system update`; the
     workstation's version is the one this lab was tested with.

---

## Part 6: Set up your environment variables

dbt Fusion needs three settings to run Lake Compute, and your profile reads your token from
an environment variable so it is never written to a file.

1. Add the three settings to your shell, so every new terminal has them:

   ```bash
   cat >> ~/.bashrc <<'EOF'
   export DBT_ENGINE_EXPERIMENTAL_MULTI_ADAPTER=true
   export DBT_ALLOW_EXPERIMENTAL_ADAPTERS=true
   export DBT_SEND_ANONYMOUS_USAGE_STATS=false
   EOF
   source ~/.bashrc
   ```

   - `DBT_ENGINE_EXPERIMENTAL_MULTI_ADAPTER` and `DBT_ALLOW_EXPERIMENTAL_ADAPTERS` turn on
     Lake Compute, which is an experimental adapter in dbt Fusion.
   - `DBT_SEND_ANONYMOUS_USAGE_STATS=false` avoids a startup error when Lake Compute is the
     only adapter in a project.

2. Set your token for this terminal. The command prompts for it and doesn't echo it or save
   it in your shell history. Paste the token from Part 4 and press **Enter**:

   ```bash
   read -rsp "Lake Compute token: " LAKE_COMPUTE_TOKEN && export LAKE_COMPUTE_TOKEN && echo
   ```

3. Check it is set, without printing it:

   ```bash
   [ -n "$LAKE_COMPUTE_TOKEN" ] && echo "token is set"
   ```

   > The token lives only in this terminal. If you open a new terminal, run step 2 again.

---

## Part 7: Clone your dbt project

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

   - `models/staging` holds one staging model per Hospitality table. This lab uses
     `stg_hospitality__hpt_records`.
   - Lake Compute can't build **views**. Your copy of the project was generated with every
     `+materialized: view` changed to `+materialized: table`, so the staging models build
     as tables. Keep it that way, and build your marts as tables too.

---

## Part 8: Configure `profiles.yml`

Your workstation already has a dbt profile at `~/.dbt/profiles.yml`, written when the lab
was launched, with your destination's group ID already filled in.

1. Look at it:

   ```bash
   cat ~/.dbt/profiles.yml
   ```

   It looks like this, with your own group ID and schema:

   ```yaml
   default:
     target: default
     outputs:
       default:
         - name: lakecompute
           type: lakecompute
           method: fivetran
           fivetran_credential: "{{ env_var('LAKE_COMPUTE_TOKEN') }}"
           base_url: "https://api.lake-compute.fivetran.com/"
           database: "<your group ID>"
           schema: "dbt_<your suffix>"
   ```

   - `type: lakecompute` sends every model to Lake Compute. The output is a **list** because
     dbt Fusion supports several adapters in one profile; here there is only one.
   - `fivetran_credential` reads your token from the environment variable you set in
     Part 6. The token itself is never written to this file.
   - `database` is your destination's group ID. `schema` is where your models land: marts
     in `dbt_<your suffix>`, staging models in `dbt_<your suffix>_staging`.

2. Check that `database` matches the group ID you copied in Part 4:

   ```bash
   grep database ~/.dbt/profiles.yml
   ```

   If it still shows `REPLACE_WITH_YOUR_DESTINATION_GROUP_ID`, replace it with your group ID:

   ```bash
   sed -i 's/REPLACE_WITH_YOUR_DESTINATION_GROUP_ID/<your_group_id>/' ~/.dbt/profiles.yml
   ```

3. Write down your schema; this worksheet calls it `<DBT_SCHEMA>`:

   ```bash
   grep schema ~/.dbt/profiles.yml
   ```

---

## Part 9: Configure `dbt_project.yml`

The project reads the Hospitality tables from the schema your connector created. Tell it where.

| Placeholder | Your value |
|---|---|
| `<GROUP_ID>` | Your destination's group ID from Part 4 |
| `<SOURCE_SCHEMA>` | `<firstname>_<lastname>_pg_hospitality`: your connector's prefix plus `_hospitality` |
| `<DBT_SCHEMA>` | The `schema` in your profile, for example `dbt_ab12cd34` |

1. Open the project in an editor (`nano dbt_project.yml`, or `code .` for VS Code if
   available on your workstation).
2. Under `vars`, set `source_database` to `<GROUP_ID>` and `source_schema` to
   `<SOURCE_SCHEMA>`, and add `review_threshold: 30`, which your marts will use.
3. Under `models: hospitality_lab:`, keep `staging` as `+materialized: table` with
   `+schema: staging`, and add `marts` with `+materialized: table`.

The result should look like this, with your values in place of the placeholders:

```yaml
name: hospitality_lab
version: '1.0.0'
config-version: 2
profile: 'default'
require-dbt-version: [">=1.9.0", "<3.0.0"]
model-paths: ['models']
test-paths: ['tests']
macro-paths: ['macros']
vars:
  source_database: '<GROUP_ID>'
  source_schema: '<SOURCE_SCHEMA>'
  review_threshold: 30
models:
  hospitality_lab:
    staging:
      +materialized: table
      +schema: staging
    marts:
      +materialized: table
```

---

## Part 10: Running dbt on Lake Compute

Run every command from inside your project directory, in a terminal where your token is set
(Part 6).

1. Install any packages and parse the project:

   ```bash
   dbtf deps
   dbtf parse
   ```

2. Test the connection:

   ```bash
   dbtf debug
   ```

   - The **connection test** must be **OK**.
   - On a lake where you haven't built anything yet, the write test fails with
     `Schema "dbt_..." does not exist`. That is expected: `dbtf debug` doesn't create
     schemas, and your first build does.

3. Build the staging models this exercise uses:

   ```bash
   dbtf build --select stg_hospitality__hpt_records
   ```

4. The build must finish successfully. A zero-row build is not evidence of a populated
   pipeline: if a model is empty, check the Fivetran sync status from Part 3.

> **How dbt runs on Lake Compute.** `dbtf` compiles your models on the workstation and sends
> the SQL to Lake Compute, which runs it next to your data lake and writes each model back
> as an Iceberg table. Lake Compute speaks **DuckDB SQL**. It supports `table`,
> `incremental` and `ephemeral` models and data tests; it does not support views, pre- or
> post-hooks, grants or unit tests.

---

## Part 11: Build the marts

**Business question:** Guest transport review. These are guest recommendation observations, not actual trip durations.

Use these files:

| File | Purpose |
|---|---|
| `models/marts/fct_hospitality_observations.sql` | Row-level fact table |
| `models/marts/mart_hospitality_review_summary.sql` | Business-facing mart table |
| `models/marts/_hospitality__addon_models.yml` | Model descriptions and generic data tests |
| `tests/assert_hospitality_addon.sql` | Singular reconciliation test; returns rows only when an invariant fails |

**Model requirements**

- Build `fct_hospitality_observations` at one row per `hpt_records.record_id`. Use `ref('stg_hospitality__hpt_records')` and select the source key as `observation_id`.
- Derive `group_key` from `event_category`: use `missing:` for null and `value:` plus its string value otherwise. Keep `group_name` as the display string, using `Unknown` for null. Distinguish a real value named Unknown from a missing value through the key.
- Calculate `metric_value` from `transportation_eta_minutes` using safe numeric conversion. Convert arithmetic inputs before calculating. Division by zero must produce null. This measures estimated transport time in minutes.
- Preserve `synced_at`. Set `requires_review` to `metric_value >= review_threshold`, initially 30. Missing metrics and their flags stay null; never silently turn them into zero or false.
- Build `mart_hospitality_review_summary` from the fact at one row per `group_key`. Include group name, total observations, measured observations, missing metric count, review count, unweighted mean metric and review share. Divide review count by measured observations with a zero-safe denominator and decimal arithmetic.
- Test unique/non-null observation IDs and group keys, observation-to-staging key relationships and non-null counts. Add a singular test for source/fact row preservation, summary row-count reconciliation, measured + missing = total and review count no greater than measured count. Legitimately missing metrics must remain visible, not fail a blanket non-null test.
- Use these exact output column names on `mart_hospitality_review_summary`, because the aggregate checks later in this worksheet query them: `observation_count`, `measured_count`, `missing_metric_count`, `review_count`.

1. Create `models/marts` and `tests` if needed. Write the fact SQL first: start with the
   staging input(s), give the output columns explicit names and implement the model
   requirements above. Use CTEs to separate normalization, joins or ranking from derived
   business flags.
2. Add `{{ config(materialized='table') }}` to each mart. Use `ref()` for staging
   dependencies; never hard-code your schema in SQL.
3. Write in **DuckDB SQL**. Use `try_cast(x as double)` for safe numeric conversion and
   `try_cast(x as date)` or `try_strptime(...)` for dates. Numeric source columns can
   arrive in the lake as text, so convert them before doing arithmetic or comparisons.
4. Write the second mart from `ref()` of your fact. Keep its documented grain. Review null
   handling and arithmetic before moving on.
5. Create the YAML file with `version: 2`, a `models` list, descriptions, column
   definitions and `data_tests`. Add `unique`/`not_null` only where the contract requires
   them. For relationships, use `arguments: {to: ref('model_name'), field: key_column}`
   with the actual model and field. Do not duplicate the existing staging model declarations.
6. Write the singular test as one or more SELECTs joined with `union all`. Each SELECT
   returns a failure label only when an invariant is violated. A passing singular test
   returns zero rows.

Try your own implementation before comparing it with the **Full answer** at the end.

---

## Part 12: Build and validate

1. Inspect the resolved targets before building:

   ```bash
   dbtf ls --select +mart_hospitality_review_summary --resource-type model --output json --output-keys name database schema alias
   ```

   Every model should target your group ID. The staging models use `<DBT_SCHEMA>_staging`
   and the two marts use `<DBT_SCHEMA>`.

2. Build the mart, its ancestors and their tests:

   ```bash
   dbtf build --select +mart_hospitality_review_summary
   ```

3. Require a successful final result. Read failing test output and repair the model or
   assumptions; don't remove tests just to get a green run. Check the singular
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

## Part 13: Query your lake with the DuckDB CLI

Lake Compute wrote your marts back to the data lake as Iceberg tables, registered in the
Fivetran Catalog. DuckDB can read them directly through that catalog, using short-lived
storage credentials the catalog hands out. No AWS keys are needed.

1. In Fivetran, open your destination's **Catalog Integration** tab and stay on
   **Base configuration**. It lists four values: **Server endpoint**, **Catalog**,
   **Client ID** and **Client secret**.

   > Copy the values when you need them, and don't click **Regenerate client secret**:
   > that invalidates the current secret everywhere it is used.

2. In the workstation terminal, set the **Client secret** the same way as your token, so it
   never appears in a command or in DuckDB's history:

   ```bash
   read -rsp "Catalog client secret: " POLARIS_CLIENT_SECRET && export POLARIS_CLIENT_SECRET && echo
   ```

3. Start DuckDB:

   ```bash
   duckdb
   ```

4. Load the extensions, create a secret for the catalog and attach your lake. Replace
   `<server_endpoint>` with the **Server endpoint** (it ends in `/api/catalog`; leave off
   any trailing slash), `<client_id>` with the **Client ID** and `<catalog>` with the
   **Catalog** value, your group ID:

   ```sql
   LOAD iceberg;
   LOAD httpfs;

   CREATE SECRET polaris (
     TYPE iceberg,
     CLIENT_ID '<client_id>',
     CLIENT_SECRET getenv('POLARIS_CLIENT_SECRET'),
     OAUTH2_SERVER_URI '<server_endpoint>/v1/oauth/tokens',
     OAUTH2_SCOPE 'PRINCIPAL_ROLE:ALL'
   );

   ATTACH '<catalog>' AS lake (TYPE iceberg, ENDPOINT '<server_endpoint>', SECRET polaris);
   ```

   The secret lasts only for this DuckDB session.

5. List what's in your lake:

   ```sql
   SHOW ALL TABLES;
   ```

   You should see your connector's schema, `<firstname>_<lastname>_pg_hospitality`, your
   staging schema `<DBT_SCHEMA>_staging` and your marts schema `<DBT_SCHEMA>`.

6. Query the raw data Fivetran landed:

   ```sql
   SELECT count(*) FROM lake.<firstname>_<lastname>_pg_hospitality.hpt_records;
   ```

7. Query your mart:

   ```sql
   SELECT * FROM lake.<DBT_SCHEMA>.mart_hospitality_review_summary LIMIT 10;
   ```

8. Repeat the aggregate check from Part 12, this time from DuckDB. The result should match
   what `dbtf show` returned:

   ```sql
   select coalesce(sum(observation_count), 0) as observations, coalesce(sum(measured_count), 0) as measured, coalesce(sum(missing_metric_count), 0) as missing, coalesce(sum(review_count), 0) as flagged from lake.<DBT_SCHEMA>.mart_hospitality_review_summary;
   ```

9. Leave DuckDB with `.quit`.

**Checkpoint:** DuckDB reads the tables Fivetran landed and the marts Lake Compute built,
straight from your data lake.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| **AWS Read and Write Access** test fails | You opened someone else's destination, or the bucket, prefix or role ARN was mistyped | Check that the destination name matches your **Fivetran Destination** field and the values match your 1Password link, then re-run the test; if it still fails, ask the instructor to confirm your destination was given access to your role |
| `dbt` says the profile is misconfigured or doesn't know `lakecompute` | You ran `dbt` (dbt Core 1.x) | Use `dbtf` |
| dbt refuses to run Lake Compute, or crashes at startup | The settings from Part 6 are missing in this terminal | Run `source ~/.bashrc` and set your token again |
| Authentication fails | Token not set in this terminal, or it was copied incompletely | Run Part 6 step 2 again; generate a new token if needed |
| `dbtf debug` write test fails with `Schema ... does not exist` | Expected before your first build | Continue: `dbtf build` creates the schemas |
| `501: Views are not supported on Lake Compute` | A model or folder is configured as a view | Set `+materialized: table` in `dbt_project.yml` or `materialized='table'` in the model |
| A source table is not found | `source_schema` doesn't match your connector's schema, or the sync hasn't finished | Set `source_schema` to `<firstname>_<lastname>_pg_hospitality` and check the sync in Fivetran |
| `dbt1159` `Deprecated test arguments ... at top-level` when dbt parses the project | A generic test's parameters (`to`, `field`) sit directly under the test name, the dbt 1.x layout | Nest them under `arguments:`, as in `relationships: {arguments: {to: ref('model_name'), field: key_column}}`, then build again |
| Conversion or comparison errors on numeric columns | A numeric source column arrived in the lake as text | Convert it with `try_cast(... as double)` before using it |
| DuckDB returns 401 from the token endpoint | Client ID or secret is wrong or stale | Copy the current **Client ID** and **Client secret** from **Catalog Integration → Base configuration** |

---

## What you did

You configured a Managed Data Lake Service destination on Amazon S3, synced the
Hospitality data into it as Apache Iceberg, and built two tested marts on Fivetran Lake
Compute with dbt Fusion from a Linux workstation, with no warehouse. Lake Compute wrote
your marts back to the same lake, and you read them with DuckDB through the Fivetran Catalog.

## References

- [Inspected industry template](https://github.com/dbt-Hands-on-Labs/dbt-hospitality-lab)
- Lake Compute — `docs.getdbt.com/docs/lake-compute`
- Managed Data Lake Service — `fivetran.com/docs/managed-data-lake-service`
- Query Iceberg tables from DuckDB — `fivetran.com/docs/managed-data-lake-service/tutorials`

## Full answer

Compare this implementation with yours after attempting the exercise. The complete
`dbt_project.yml` is in Part 9. Existing staging and source files stay in place. Different
SQL can be correct if it meets the same grain, null-handling and reconciliation contracts.

### `models/marts/fct_hospitality_observations.sql`

```sql
{{ config(materialized='table') }}
with normalized as (
    select
        record_id as observation_id,
        case when event_category is null then 'missing:'
             else concat('value:', cast(event_category as varchar)) end as group_key,
        coalesce(cast(event_category as varchar), 'Unknown') as group_name,
        try_cast(cast(transportation_eta_minutes as varchar) as double) as metric_value,
        synced_at
    from {{ ref('stg_hospitality__hpt_records') }}
)
select *, metric_value >= {{ var('review_threshold', 30) }} as requires_review
from normalized
```

### `models/marts/mart_hospitality_review_summary.sql`

```sql
{{ config(materialized='table') }}
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
from {{ ref('fct_hospitality_observations') }}
group by group_key
```

### `models/marts/_hospitality__addon_models.yml`

```yaml
version: 2
models:
  - name: fct_hospitality_observations
    description: "One row per hpt_records.record_id. Metric: estimated transport time in minutes. Missing metrics and unknown review flags are retained."
    columns:
      - name: observation_id
        data_tests:
          - unique
          - not_null
          - relationships:
              arguments:
                to: ref('stg_hospitality__hpt_records')
                field: record_id
      - name: group_key
        data_tests:
          - not_null
  - name: mart_hospitality_review_summary
    description: "One row per group_key derived from event_category. Counts are observations; review_share excludes missing metrics."
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

### `tests/assert_hospitality_addon.sql`

```sql
-- Fail if rows disappear, multiply, or are counted incorrectly in the summary.
select 'source_grain' as failure
where (select count(*) from {{ ref('fct_hospitality_observations') }}) <>
      (select count(*) from {{ ref('stg_hospitality__hpt_records') }})
union all
select 'summary_reconciliation' as failure
where (select count(*) from {{ ref('fct_hospitality_observations') }}) <>
      (select coalesce(sum(observation_count), 0) from {{ ref('mart_hospitality_review_summary') }})
union all
select 'metric_reconciliation' as failure
from {{ ref('mart_hospitality_review_summary') }}
where measured_count + missing_metric_count <> observation_count
   or review_count > measured_count
   or review_count < 0
```
