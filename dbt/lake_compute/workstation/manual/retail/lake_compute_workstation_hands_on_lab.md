# Lake Compute on a workstation: Retail

*November 2026. Fivetran Managed Data Lake Service on Amazon S3, with dbt Fusion running your
models on Fivetran Lake Compute from a provided Linux workstation. No warehouse is involved.*

Thank you for registering for our hands-on lab. This worksheet provides all the information
you need. Please read through the requirements first. If you can't meet them, let us know
and we'll rebook you for another workshop.

---

## Objective

Land the Retail data in a Managed Data Lake on Amazon S3, then build **Retail inventory priorities**
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
2. Set up a connector to send the Retail data to the data lake
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
- **Git Clone Command** for your own copy of the Retail dbt project, a private GitHub
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

Now we'll create a connector to sync the Retail data into your managed data lake in S3.

1. Navigate to the **Connections** tab.
2. Click **Add Connection**.
3. Search for **PostgreSQL** and click **Set up**.
4. Select your destination (the one from Part 2).
   - **Make sure you don't use someone else's destination** — check that the name matches your **Fivetran Destination** field.
5. Select **Fivetran naming**. The name you provide in the **Destination schema prefix** field becomes the name of
   your connector. Use the format `<firstname>_<lastname>_pg`.

   > Write this value down. Fivetran combines it with the source schema name to form the
   > schema your dbt project reads: `<firstname>_<lastname>_pg_retail`.

6. Populate the setup form with the provided PostgreSQL credentials.
7. For **Authentication Method**, select **Connect with username and password**.
8. For **Connection method**, make sure **Connect directly** is selected.
9. For **Update Method**, select **Query-Based**.
   - *Context:* PostgreSQL's older XMIN and Fivetran Teleport Sync methods have been sunset and replaced by **Query-Based** change data capture. The other available method is **Logical replication** (using the `pgoutput` plugin).
10. Click **Save & Test**.
11. Confirm the TLS certificate when prompted.
12. Wait for the setup tests to complete, then click **Continue**.
13. Fivetran now fetches all tables, schemas, and columns for the provided database.
14. Make sure the `retail` schema and all of its tables are selected.
15. Click **Save & Continue**.
16. For handling schema changes, select **Allow all**.
17. Click **Start Initial Sync**.
18. Wait for the sync to complete.

**Checkpoint:** The connection shows a completed initial sync of the `retail` schema.
Fivetran has written each table to
`s3://<S3 Bucket>/<S3 Prefix Path>/<firstname>_<lastname>_pg_retail/<table>/`
(for example `.../inv_weekly_data/`) as Parquet files with Iceberg metadata.

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

LabStudios created a private copy of the Retail dbt project for you, and gave your
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

   - `models/staging` holds one staging model per Retail table. This lab uses
     `stg_retail__inv_weekly_data`, `stg_retail__inv_products`, `stg_retail__inv_stores`.
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

The project reads the Retail tables from the schema your connector created. Tell it where.

| Placeholder | Your value |
|---|---|
| `<GROUP_ID>` | Your destination's group ID from Part 4 |
| `<SOURCE_SCHEMA>` | `<firstname>_<lastname>_pg_retail`: your connector's prefix plus `_retail` |
| `<DBT_SCHEMA>` | The `schema` in your profile, for example `dbt_ab12cd34` |

1. Open the project in an editor (`nano dbt_project.yml`, or `code .` for VS Code if
   available on your workstation).
2. Under `vars`, set `source_database` to `<GROUP_ID>` and `source_schema` to
   `<SOURCE_SCHEMA>`, and add `review_threshold: 0`, which your marts will use.
3. Under `models: retail_lab:`, keep `staging` as `+materialized: table` with
   `+schema: staging`, and add `marts` with `+materialized: table`.

The result should look like this, with your values in place of the placeholders:

```yaml
name: retail_lab
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
  review_threshold: 0
models:
  retail_lab:
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
   dbtf build --select stg_retail__inv_weekly_data stg_retail__inv_products stg_retail__inv_stores
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

**Business question:** Retail inventory priorities. The latest observation per product/store is historical inventory, not a demand forecast.

Use these files:

| File | Purpose |
|---|---|
| `models/marts/fct_retail_inventory_weekly.sql` | Row-level fact table |
| `models/marts/mart_retail_inventory_priorities.sql` | Business-facing mart table |
| `models/marts/_retail__addon_models.yml` | Model descriptions and generic data tests |
| `tests/assert_retail_addon.sql` | Singular reconciliation test; returns rows only when an invariant fails |

**Model requirements**

- Build `fct_retail_inventory_weekly` at one row per `row_id`. Left join products on `product_id` and stores on `store_id`; preserve the weekly input count.
- Include product/store names, category, region, stock, sales, days of supply, synced timestamp, typed inventory date and anomaly flag. Use safe date/boolean conversions for the chosen destination. Add `review_threshold` (initially 0 days) to the product's lead time.
- Build `mart_retail_inventory_priorities` from the fact. Select the latest date per product/store, with `row_id` ascending as a deterministic tie-breaker. Use a null-aware structured encoding of both IDs before hashing the product/store key.
- Carry the relevant fact columns through and calculate `needs_replenishment` as days of supply less than adjusted lead time. Do not use today's date to filter historical data.
- Test unique/non-null weekly row IDs and latest keys, product/store relationships, typed dates/flags, latest-to-weekly row relationships and non-null replenishment inputs. Add a singular test for weekly row preservation, one latest row per pair and correct latest dates.

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
   dbtf ls --select +mart_retail_inventory_priorities --resource-type model --output json --output-keys name database schema alias
   ```

   Every model should target your group ID. The staging models use `<DBT_SCHEMA>_staging`
   and the two marts use `<DBT_SCHEMA>`.

2. Build the mart, its ancestors and their tests:

   ```bash
   dbtf build --select +mart_retail_inventory_priorities
   ```

3. Require a successful final result. Read failing test output and repair the model or
   assumptions; don't remove tests just to get a green run. Check the singular
   reconciliation test ran. If it didn't, run it explicitly:

   ```bash
   dbtf test --select assert_retail_addon
   ```

4. Run an aggregate check through dbt and record the result:

   ```bash
   dbtf show --inline "select count(*) as latest_rows, count(distinct product_store_key) as distinct_pairs, sum(case when needs_replenishment then 1 else 0 end) as replenishment_count, min(inventory_date) as first_observation, max(inventory_date) as last_observation from {{ ref('mart_retail_inventory_priorities') }}" --limit 1
   ```

   Don't expect fixed numbers: each lab's sync state can differ.

5. Optional: save your work to your repository.

   ```bash
   git add dbt_project.yml models/marts tests
   git commit -m "Add retail marts on Lake Compute"
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

   You should see your connector's schema, `<firstname>_<lastname>_pg_retail`, your
   staging schema `<DBT_SCHEMA>_staging` and your marts schema `<DBT_SCHEMA>`.

6. Query the raw data Fivetran landed:

   ```sql
   SELECT count(*) FROM lake.<firstname>_<lastname>_pg_retail.inv_weekly_data;
   ```

7. Query your mart:

   ```sql
   SELECT * FROM lake.<DBT_SCHEMA>.mart_retail_inventory_priorities LIMIT 10;
   ```

8. Repeat the aggregate check from Part 12, this time from DuckDB. The result should match
   what `dbtf show` returned:

   ```sql
   select count(*) as latest_rows, count(distinct product_store_key) as distinct_pairs, sum(case when needs_replenishment then 1 else 0 end) as replenishment_count, min(inventory_date) as first_observation, max(inventory_date) as last_observation from lake.<DBT_SCHEMA>.mart_retail_inventory_priorities;
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
| A source table is not found | `source_schema` doesn't match your connector's schema, or the sync hasn't finished | Set `source_schema` to `<firstname>_<lastname>_pg_retail` and check the sync in Fivetran |
| Conversion or comparison errors on numeric columns | A numeric source column arrived in the lake as text | Convert it with `try_cast(... as double)` before using it |
| DuckDB returns 401 from the token endpoint | Client ID or secret is wrong or stale | Copy the current **Client ID** and **Client secret** from **Catalog Integration → Base configuration** |

---

## What you did

You configured a Managed Data Lake Service destination on Amazon S3, synced the
Retail data into it as Apache Iceberg, and built two tested marts on Fivetran Lake
Compute with dbt Fusion from a Linux workstation, with no warehouse. Lake Compute wrote
your marts back to the same lake, and you read them with DuckDB through the Fivetran Catalog.

## References

- [Inspected industry template](https://github.com/dbt-Hands-on-Labs/dbt-retail-lab)
- Lake Compute — `docs.getdbt.com/docs/lake-compute`
- Managed Data Lake Service — `fivetran.com/docs/managed-data-lake-service`
- Query Iceberg tables from DuckDB — `fivetran.com/docs/managed-data-lake-service/tutorials`

## Full answer

Compare this implementation with yours after attempting the exercise. The complete
`dbt_project.yml` is in Part 9. Existing staging and source files stay in place. Different
SQL can be correct if it meets the same grain, null-handling and reconciliation contracts.

### `models/marts/fct_retail_inventory_weekly.sql`

```sql
{{ config(materialized='table') }}
select
    w.row_id, w.product_id, w.store_id,
    p.product_name, p.category, s.store_name, s.region,
    cast(try_strptime(cast(w.date as varchar), '%Y-%m-%d') as date) as inventory_date,
    try_cast(cast(w.is_anomaly as varchar) as boolean) as is_anomaly,
    w.current_stock_level, w.weekly_sales_volume,
    try_cast(w.days_of_supply as double) as days_of_supply,
    p.restock_lead_time_days + {{ var('review_threshold', 0) }} as restock_lead_time_days,
    w.synced_at
from {{ ref('stg_retail__inv_weekly_data') }} w
left join {{ ref('stg_retail__inv_products') }} p on w.product_id = p.product_id
left join {{ ref('stg_retail__inv_stores') }} s on w.store_id = s.store_id
```

### `models/marts/mart_retail_inventory_priorities.sql`

```sql
{{ config(materialized='table') }}
with ranked as (
    select *,
        row_number() over (
            partition by product_id, store_id
            order by inventory_date desc nulls last, row_id asc
        ) as observation_rank
    from {{ ref('fct_retail_inventory_weekly') }}
)
select
    sha256(cast(to_json([cast(product_id as varchar), cast(store_id as varchar)]) as varchar)) as product_store_key,
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
