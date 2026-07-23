# Hands-on Lab: Managed Data Lake Service

*Updated July 2026. Verified against the current Fivetran Managed Data Lake Service
documentation and the "Query Fivetran-managed Apache Iceberg tables from Snowflake"
tutorial.*

Thank you for registering for our hands-on lab. This worksheet provides all the
information you need to prepare. Please read through the requirements first. If you can't
meet them, let us know and we'll rebook you on another workshop.

---

## Objective

Stand up a Managed Data Lake Service destination on Google Cloud Storage, sync a table
into it as Apache Iceberg, and then query that table from Snowflake through a
catalog-linked database — the recommended, documented path.

## Requirements

- Web browser (Chrome)
- No local software installation is required. 
   - Everything in this lab runs in the Fivetran and Snowflake web UIs.

## Housekeeping

- You will receive an invitation to the **MDLS_HANDS_ON_LAB** Fivetran account.
- **Do not set up a new account or trial** — you will receive an invite to a dedicated
  Fivetran account prior to the lab.
- Confirm that you have received the invite. It is sent to the email address you used
  to sign up for this lab and originates from `notifications@fivetran.com`.

## What you'll do in this lab

1. Set up a Managed Data Lake destination on Google Cloud Storage
2. Set up a connector to send data to the data lake
3. Query the landed Iceberg data from Snowflake

---

## Part 0: Credentials

### Snowflake

- Provided by your instructor via a 1Password link.
   - When entering these credentials into any setup form, make sure you select **SaaS** as the deployment model.

### Postgres

- Provided by your instructor via a 1Password link.

---

## Part 1: Accessing the provided resources

1. Log into Fivetran at `https://fivetran.com/login`.
   - If you just signed up for the first time, use the username and password you used
     for your sign-up.
   - If you already have a Fivetran account, use those credentials.
2. Switch to the `MDLS_HANDS_ON_LAB` account via the drop-down menu in the top left.
3. Log into Snowflake.
   - Click the link provided in **Part 0: Credentials**.
   - Enter the provided credentials and click **Sign in**.

**Checkpoint:** you are signed into the `MDLS_HANDS_ON_LAB` Fivetran account and into
the Snowflake account, each in its own browser tab. Keep both open — you will move
between them in Part 4.

---

## Part 2: Setting up the data lake destination

We'll begin by setting up a Managed Data Lake destination.

1. Navigate to the **Destinations** tab in the left-hand navigation.
2. Click **Add Destination**.
3. Three Fivetran Managed Data Lake Service options appear immediately. Select the
   **Google Cloud Storage** option by clicking its **Set up** button.
4. For the **Destination name**, enter `<firstname>_<lastname>_gcs`, replacing
   `<firstname>` and `<lastname>` with your actual first and last name. Click **Add**.
5. Enter the **Bucket name** provided via the 1Password credentials link.
6. For the **GCS Prefix Path**, enter `<firstname>_<lastname>_mdls`, again replacing the
   placeholders with your name.

   > **Note — this field is immutable.** The prefix path cannot be changed after the
   > destination is created. The same is true of the bucket and the region. Getting it
   > wrong means recreating the destination, so check your spelling now. This is one of
   > the delivery lessons from Module 02.

7. Leave the **Update BigLake Metastore** option untoggled.

   > **Note — catalog integrations are final.** Catalog integration settings cannot be
   > edited after the first successful sync into the data lake. In a real engagement you
   > would decide these *before* creating the destination, based on which engines the
   > customer will use. For this lab we're leaving BigLake off because we're querying
   > from Snowflake, which reads the Fivetran Catalog directly.

8. Leave the remaining options at their defaults. Note that **Snapshot Retention
   Period** defaults to one week — unlike the catalog toggles, this one *can* be changed
   at any time later.

9. To access cloud storage locations we need to grant access to Fivetran to the cloud storage
   environment. Pause here, so your instructor can allowlist your destinations. 

10. Click **Save & Test**.

   > If you get an error connecting to the Google account, confirm with your instructor
   > that the service accounts have been correctly added.

11. Let the setup tests run. For GCS these are **Input Validation** and **GCS Read and
    Write Access**. (A **BigLake Metastore Access** test also runs, but only if the
    Update BigLake Metastore toggle is on — which we left off in step 7, so you will not
    see it.)
12. Once the tests complete successfully, click **View Destination**.

**Checkpoint:** all setup tests show green, and your destination appears in the
Destinations list with your name. Every Managed Data Lake destination is provisioned
with the **Fivetran Catalog** automatically — you did not have to configure it. You'll
use it in Part 4.

---

## Part 3: Creating a connector

Now we'll create a connector to sync data into our managed data lake in GCS.

1. Navigate to the **Connections** tab.
2. Click **Add Connection**.
3. Search for **Google Cloud SQL for PostgreSQL** and click **Set up**.
4. Select the destination you created. 
   - **Make sure you don't use someone else's destination** — check that the name matches yours.
5. The name you provide in the **Destination schema prefix** field becomes the name of
   your connector. Use the format `<firstname>_<lastname>_pg`.

   > Write this value down. You'll need it in Part 4, because Fivetran combines it with
   > the source schema name to form the catalog namespace.

6. Populate the setup form with the provided PostgreSQL credentials.
7. For **Authentication Method**, select **Connect with username and password**.
8. For **Connection method**, make sure **Connect directly** is selected.
9. For **Update Method**, select **Detect Changes via XMIN**.
10. Click **Save & Test**.
11. Confirm the TLS certificate when prompted.
12. Wait for the setup tests to complete, then click **Continue**.
13. Fivetran now fetches all tables, schemas, and columns for the provided database.
14. To keep this lab short, deselect all schemas and tables. We will only sync the
    `coffee_prices` table from the `agriculture` schema.
15. Enable the `agriculture` schema and select the `coffee_prices` table, if it isn't
    already selected.
16. Click **Save & Continue**.
17. For handling schema changes, select **Allow all**.
18. Click **Start Initial Sync**.
19. Wait for the sync to complete.

**Checkpoint:** the connection shows a completed initial sync with one table synced.
Fivetran has written the data to
`gs://mdls-gcs-hands-on-lab/<firstname>_<lastname>_mdls/<firstname>_<lastname>_pg_agriculture/coffee_prices/`
as Parquet files, with both Iceberg metadata and a Delta transaction log. 
   - Both formats are always written — there is no format selector.

> **Instructor demo:** if you look at the GCS bucket, you can see the folder structure
> Fivetran created: a `data/` folder holding the shared Parquet files, a `metadata/`
> folder for the Iceberg snapshots and schemas, and a `_delta_log/` folder for the Delta
> transaction log. One copy of the data, two sets of metadata pointing at it.

---

## Part 4: Querying the data from Snowflake

We're now going to read the Iceberg tables Fivetran wrote to GCS, using Snowflake.

Snowflake reads the Fivetran Catalog — a managed Apache Polaris implementation speaking
the Iceberg REST protocol — through a **catalog-linked database**. This is the
recommended approach in Fivetran's official tutorial, and the simplest: it uses
credentials vended by the catalog, so no Snowflake external volume is required.

### 4.1 Collect the catalog credentials from Fivetran

1. In Fivetran, click the **Destinations** tab and open the destination with your name.
2. Click the **Catalog integration** tab, then the **Snowflake** tab.

   > **Use the Snowflake tab, not Base configuration.** Base configuration holds the
   > values for DuckDB, Trino, and Starburst. The Snowflake tab generates a complete
   > `CREATE CATALOG INTEGRATION` statement for you, pre-filled with your values.

3. Copy these five values from the generated SQL:
   - `CATALOG_URI`
   - `WAREHOUSE` — this is your Fivetran **group ID**, not a Snowflake virtual
     warehouse. It surprises most people the first time.
   - `OAUTH_TOKEN_URI`
   - `OAUTH_CLIENT_ID`
   - `OAUTH_CLIENT_SECRET`

   > **The client secret is displayed once.** Copy it somewhere safe now. Regenerating
   > it immediately invalidates the old one and breaks any existing Snowflake catalog
   > integration until you update it.

### 4.2 Open a Snowflake worksheet

1. Navigate to the provided Snowflake account.
2. Click **Create** and select **SQL Worksheet**.
3. Rename the worksheet so it's easy to find, using the format
   `<firstname>_<lastname>_mdls_hol`.

### 4.3 Create the catalog integration and linked database

The simplest approach is to paste the statement Fivetran generated on the **Catalog
integration → Snowflake** tab directly into your worksheet. It looks like this:
 - **DO NOT paste the SQL below. This is just an example and will not work**

```sql
CREATE OR REPLACE CATALOG INTEGRATION fivetran_catalog_renegade_kindle
  CATALOG_SOURCE = POLARIS
  TABLE_FORMAT = ICEBERG
  CATALOG_NAMESPACE = '{fivetran_delivered_schema_name}'
  REST_CONFIG = (
    ACCESS_DELEGATION_MODE = VENDED_CREDENTIALS,
    CATALOG_URI = 'https://kneeling-pronto.us-east4.gcp.polaris.fivetran.com/api/catalog'
    WAREHOUSE = 'renegade_kindle'
  )
  REST_AUTHENTICATION = (
    TYPE = OAUTH
    OAUTH_TOKEN_URI = 'https://kneeling-pronto.us-east4.gcp.polaris.fivetran.com/api/catalog/v1/oauth/tokens'
    OAUTH_CLIENT_ID = 'redacted'
    OAUTH_CLIENT_SECRET = 'redacted'
    OAUTH_ALLOWED_SCOPES = ('PRINCIPAL_ROLE:ALL')
  )
  ENABLED = TRUE,
  REFRESH_INTERVAL_SECONDS = 1800; -- Configurable: adjust this value to control how often Snowflake refreshes the catalog integration

CREATE DATABASE catalog_db_renegade_kindle
  LINKED_CATALOG = (
    CATALOG = 'fivetran_catalog_renegade_kindle'
  );
```
1. You will need to replace `{fivetran_delivered_schema_name}` with name of your connector followed by `agriculture`, so if you've been following the naming conventions it will look something like `firstname_lastname_pg_agriculture`

> **How the namespace is formed:** Fivetran combines the **destination schema prefix**
> you set in Part 3 with the **source schema name** (`agriculture`). So a connector named
> `angel_hernandez_pg` syncing the `agriculture` schema produces the namespace
> `angel_hernandez_pg_agriculture`. This trips people up — the namespace is not just the
> connector name.

2. Execute the statement. A successful run returns a status message confirming the
integration was created.

> **Two parameters people leave out.** `ACCESS_DELEGATION_MODE = VENDED_CREDENTIALS` is
> what selects the vended-credentials path — without it you are configuring the
> external-volume approach instead, and will then wonder why Snowflake wants a volume
> you never created. `OAUTH_TOKEN_URI` is required alongside the client ID and secret.
> Both are in the generated SQL; copy the whole statement rather than retyping it.

### 4.4 Verify the tables were discovered
1. Replace `your_database_name` with the catalog linked database that was created in 4.3

```sql
SHOW ICEBERG TABLES IN DATABASE your_database_name;
```

**Checkpoint:** `coffee_prices` appears in the results. The catalog-linked database
auto-discovers tables as Fivetran adds them — you never write a `CREATE TABLE`
statement.

### 4.5 Query the data

```sql
SELECT *
FROM your_database_name.<firstname>_<lastname>_pg_agriculture.coffee_prices
LIMIT 100;
```

**Checkpoint:** rows are returned. You are querying Iceberg data sitting in a Google
Cloud Storage bucket, from Snowflake compute, with no copy and no second ETL.

### 4.6 Understand the refresh behavior

Snowflake polls the Fivetran Catalog to detect new snapshots written by Fivetran syncs.

- The default polling interval is **30 seconds**.
- Valid values for `REFRESH_INTERVAL_SECONDS` range from `30` to `86400`.

You may see an older Fivetran integration guide mention 300 seconds. That is the value
baked into Fivetran's generated script, not the Snowflake default — don't conflate the
two in front of a customer.

> **Delivery note.** This refresh lag is why you should not trigger a downstream
> transformation job the instant a Fivetran sync completes when reading through a
> catalog-linked database. The catalog won't have caught up, and the job will "succeed"
> having read stale metadata. Use an interval schedule instead.

---

## Optional: Part 5 — the other two Snowflake approaches

Skip this unless you finish early. The catalog-linked database with vended credentials
that you built in Part 4 is the recommended path for most cases. This section exists so
you know when the other two documented approaches apply, and why.

Fivetran documents three approaches:

| Approach | External volume | Private networking | Use when |
|---|---|---|---|
| Catalog-linked DB + vended credentials | No | No | Most cases — what you built in Part 4 |
| Catalog-linked DB + external volume | Yes | Yes | **Private networking is required** |
| Manually created Iceberg tables | Yes | Yes | Custom naming or object organization |

**When private networking is required**, use the second approach — a catalog-linked
database backed by a Snowflake external volume you create. Vended credentials always
traverse the public internet, so a customer who needs a private path to storage cannot
use them. This is the one to remember, because it is easy to reach for manual table
registration instead and end up with unnecessary maintenance.

**Manual table registration** is only for custom naming or object organization. It
requires you to first create a Snowflake external volume (`CREATE EXTERNAL VOLUME`),
then create each Iceberg table explicitly:

```sql
CREATE ICEBERG TABLE IF NOT EXISTS <database>.<schema>.coffee_prices
  EXTERNAL_VOLUME = '<external_volume_name>'
  CATALOG = 'fivetran_catalog_int'
  CATALOG_NAMESPACE = '<firstname>_<lastname>_pg_agriculture'
  CATALOG_TABLE_NAME = 'coffee_prices'
  AUTO_REFRESH = TRUE;
```

Note what this costs you: a statement like this for **every** table, repeated every time
Fivetran adds one. The catalog-linked database in Part 4 discovers new tables on its own.
That difference is the whole reason the first approach is recommended.

> Running this section requires an external volume, which is not provisioned in the lab
> environment. Read it rather than executing it unless your instructor says otherwise.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| GCS setup test fails on read/write access | Service accounts not yet added to the bucket | Ask the instructor to confirm the service account has access to the lab GCS bucket |
| Catalog integration creation fails on auth | Client secret mistyped, or values copied from the wrong tab | Re-copy the whole generated statement from **Catalog integration → Snowflake**, not Base configuration; the secret displays once |
| Snowflake asks for an external volume you never created | `ACCESS_DELEGATION_MODE = VENDED_CREDENTIALS` omitted from `REST_CONFIG` | Recreate the catalog integration with that parameter — it is what selects the vended-credentials path |
| `SHOW ICEBERG TABLES` returns nothing | Namespace in `ALLOWED_NAMESPACES` doesn't match | It is connector name **+ source schema**, e.g. `first_last_pg_agriculture` — not just the connector name |
| Tables appear but a query returns no rows | Initial sync hadn't finished when the catalog last refreshed | Wait 30 seconds and re-run; the catalog polls on an interval |
| `WAREHOUSE` parameter rejected | Snowflake warehouse name used instead of the Fivetran group ID | The `WAREHOUSE` field on the catalog integration takes your Fivetran **group ID** |
| You see other participants' tables | `ALLOWED_NAMESPACES` omitted | Recreate the database with `ALLOWED_NAMESPACES` scoped to your namespace |

---

## What you did

You configured a Managed Data Lake Service destination on Google Cloud Storage, synced
a PostgreSQL table into it as Apache Iceberg with a Delta transaction log alongside it,
and queried it from Snowflake through a catalog-linked database reading the Fivetran
Catalog — a managed Apache Polaris implementation speaking the Iceberg REST protocol.

Along the way you saw three things that matter in a real delivery: the prefix path and
catalog integrations are decisions you cannot undo, the catalog namespace is the
connector name plus the source schema, and the catalog refresh interval introduces a lag
that downstream scheduling has to account for.

## Further reading

- Managed Data Lake Service — `fivetran.com/docs/managed-data-lake-service`
- Setup guide — `fivetran.com/docs/managed-data-lake-service/setup-guide`
- Query Iceberg tables from Snowflake —
  `fivetran.com/docs/managed-data-lake-service/tutorials/query-iceberg-tables-from-snowflake`
- Query Iceberg tables from DuckDB and from Python — same `tutorials` path
- Metadata catalogs — `fivetran.com/docs/managed-data-lake-service/metadata-catalogs`
