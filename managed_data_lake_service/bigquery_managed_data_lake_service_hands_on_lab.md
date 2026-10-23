# Hands-on Lab: Managed Data Lake Service (BigQuery)

*Updated October 2026. Reviewed against the Fivetran Managed Data Lake Service setup
and metadata catalog documentation. End-to-end validation with provisioned lab
resources is still required before delivery.*

Thank you for registering for our hands-on lab. This worksheet provides all the
information you need to prepare. Please read through the requirements first. If you can't
meet them, let us know and we'll rebook you for another workshop.

> **Completing this lab self-paced?** Use your own Fivetran account, Google Cloud project, GCS bucket, PostgreSQL source, and BigQuery access. Skip Housekeeping and Parts 0–1, and follow the [GCS setup guide](https://fivetran.com/docs/managed-data-lake-service/setup-guide) to grant the storage and BigQuery permissions described below. Replace the supplied names and credentials with your own, use your bucket’s actual region, and perform the instructor demo yourself. Other MDLS-supported clouds and query engines require different setup instructions.

---

## Objective

Stand up a Managed Data Lake Service destination on Google Cloud Storage, sync the
PostgreSQL `agriculture` tables into it as Apache Iceberg, and observe an instructor-led
BigQuery query of `coffee_prices`. BigLake Metastore (classic) registers the tables
as BigQuery external Iceberg tables; the underlying data stays in GCS.

## Requirements

- Web browser (Chrome)
- No local software installation is required.
   - Attendees work in the Fivetran web UI. The instructor uses the Google Cloud console
     and BigQuery Studio for provisioning and demonstrations.
- Attendees do not need or receive BigQuery Studio access.

## Housekeeping

- You will receive an invitation to the **MDLS_HANDS_ON_LAB** Fivetran account.
- **Do not set up a new account or trial** — you will receive an invite to a dedicated
  Fivetran account prior to the lab.
- Confirm that you have received the invite. It is sent to the email address you used
  to sign up for this lab and originates from `notifications@fivetran.com`.

## What you'll do in this lab

1. Set up a Managed Data Lake destination on Google Cloud Storage
2. Set up a connector to send data to the data lake
3. Observe the instructor query the landed Iceberg data from BigQuery

---

## Part 0: Credentials

### Google Cloud lab details

Provided by your instructor via a 1Password link:

- **Bucket name**: the provisioned GCS bucket.
- **Bucket Region**: `us-east4` for the instructor-led lab.
- **BigQuery Project ID**: the project in which Fivetran registers external tables.
- **BigQuery Dataset Prefix**: the lab-specific prefix, including its trailing underscore
  (for example, `bqp001_`). Add your participant name as instructed in Part 3.

Use lowercase letters, digits, and underscores for name components. Use the same
`<firstname>_<lastname>` throughout the worksheet (for example, `jane_doe`). If two
participants have the same name, the instructor assigns a unique suffix, such as
`jane_doe_2`, to use everywhere that participant name appears.

### PostgreSQL

- Connection credentials are provided by your instructor via a 1Password link.

### Instructor prerequisites

- Provision the GCS bucket in `us-east4` and the Google Cloud project for table registration.
- For each destination, collect the **Fivetran service account** shown in its setup form.
  Grant it **Storage Object Admin** on the lab bucket and the required BigQuery permissions
  in the registration project. The [GCS setup guide](https://fivetran.com/docs/managed-data-lake-service/setup-guide)
  documents **BigQuery Data Editor** or a custom role with the required permissions.
  Any dataset-prefix restrictions must cover the names used in this lab and Fivetran's
  automatically created metadata datasets.
- Ensure the instructor's own Google Cloud identity can inspect the bucket, view the
  registered tables, and run BigQuery queries against their underlying GCS data.
  Validate this access before delivery; granting the Fivetran service account access
  does not grant the instructor access.
- Attendees use the Fivetran-managed service account displayed by the MDLS destination;
  they do not paste a BigQuery service-account JSON key into this setup.

---

## Part 1: Accessing the provided resources

1. Log in to Fivetran at [the Fivetran login page](https://fivetran.com/login).
   - If you just signed up for the first time, use the username and password you used
     for your sign-up.
   - If you already have a Fivetran account, use those credentials.
2. Switch to the `MDLS_HANDS_ON_LAB` account via the drop-down menu in the top left.

**Checkpoint:** You are signed in to the `MDLS_HANDS_ON_LAB` Fivetran account.

---

## Part 2: Setting up the data lake destination

We'll begin by setting up a Managed Data Lake destination.

1. Navigate to the **Destinations** tab in the left-hand navigation.
2. Click **Add Destination**.
3. Three Fivetran Managed Data Lake Service options appear immediately. Select the
   **Google Cloud Storage** option by clicking its **Set up** button.
4. For the **Destination name**, enter `<firstname>_<lastname>_gcs`, replacing
   `<firstname>` and `<lastname>` with your actual first and last names. Click **Add**.
5. Enter the **Bucket name** provided via the 1Password credentials link.
6. For the **GCS Prefix Path**, enter `<firstname>_<lastname>_mdls`, again replacing the
   placeholders with your name.

   > **Note — this field is immutable.** The prefix path cannot be changed after the
   > destination is created. The same is true of the bucket and the region. Getting it
   > wrong means recreating the destination, so check your spelling now.

7. For the **Bucket Region**, choose **us-east4**, matching the provisioned bucket.

8. Enable **Update BigLake Metastore** and enter the **BigQuery Project ID** from
   your credentials in the **Google Cloud Project ID** field.

   > **Note — catalog integrations are final.** Catalog integration settings cannot be
   > edited after the first successful sync into the data lake. In a real engagement you
   > would decide these *before* creating the destination, based on which engines the
   > customer will use. For this lab, keep BigLake enabled so Fivetran registers the
   > tables as BigQuery external Iceberg tables for the instructor's query demo.

9. Set **Write Delta Format** to **ON** for demonstration purposes. Iceberg is the
   default format; Delta metadata is optional and must be explicitly enabled.
   This toggle adds a Delta transaction log alongside the Iceberg metadata over the
   same Parquet files. BigQuery continues to query **Iceberg** in this lab.
   Set **Snapshot Retention Period** to one week and leave the remaining options
   at their defaults.

10. Share the **Fivetran service account** displayed in the setup form with your
    instructor. Pause until the instructor confirms both bucket access and BigQuery
    permissions have been granted, as described in the instructor prerequisites.

11. Click **Save & Test**.

   > If you get an error connecting to the Google account, confirm with your instructor
   > that the service accounts have been correctly added.

12. Let the setup tests run. For GCS these are **Input Validation** and **GCS Read and
    Write Access**. A **BigLake Metastore Access** test also runs since we enabled it.

13. Once the tests complete successfully, click **View Destination**.

**Checkpoint:** All setup tests show green, and your destination appears in the
Destinations list with your name. Every Managed Data Lake destination is provisioned
with the **Fivetran Catalog** automatically. The additional BigLake integration
registers external Iceberg tables in BigQuery for the instructor demo in Part 4.

---

## Part 3: Creating a connector

Now we'll create a connector to sync data into our managed data lake in GCS.

1. Navigate to the **Connections** tab.
2. Click **Add Connection**.
3. Search for **PostgreSQL** and click **Set up**.
4. Select the destination you created.
   - **Make sure you don't use someone else's destination** — check that the name matches yours.
5. Select **Fivetran naming**. The name you provide in the **Destination schema prefix** field becomes the name of
   your connector. Use `<dataset_prefix><firstname>_<lastname>_pg`, where
   `<dataset_prefix>` is the **BigQuery Dataset Prefix**, including its trailing underscore.
   For example, `bqp001_` and `jane_doe` produce `bqp001_jane_doe_pg`. Each participant
   must use their own name so their tables register in a separate BigQuery dataset.

   > Write this value down. You'll need it in Part 4, because Fivetran combines it with
   > the source schema name to form the catalog namespace and BigQuery dataset:
   > `bqp001_jane_doe_pg_agriculture` in this example.

6. Populate the setup form with the provided PostgreSQL credentials.
7. For **Authentication Method**, select **Connect with username and password**.
8. For **Connection method**, make sure **Connect directly** is selected.
9. For **Update Method**, select **Query-Based**.
   - *Context:* PostgreSQL's older XMIN and Fivetran Teleport Sync methods have been sunset and replaced by **Query-Based** change data capture. The other available method is **Logical replication** (using the `pgoutput` plugin).
10. Click **Save & Test**.
11. Confirm the TLS certificate when prompted.
12. Wait for the setup tests to complete, then click **Continue**.
13. Fivetran now fetches all tables, schemas, and columns for the provided database.
14. The Fivetran database user provided to you via 1Password has access to the `agriculture` schema, which
    includes the `coffee_prices` table the instructor will query in Part 4.
15. Make sure the `agriculture` schema and all of its tables are selected.
16. Click **Save & Continue**.
17. For handling schema changes, select **Allow all**.
18. Click **Start Initial Sync**.
19. Wait for the sync to complete.

**Checkpoint:** The connection shows a completed initial sync of the `agriculture` schema.
Fivetran has written each table's data to
`gs://<your_provisioned_bucket>/<firstname>_<lastname>_mdls/<dataset_prefix><firstname>_<lastname>_pg_agriculture/<table>/`
(for example `.../coffee_prices/`) as Parquet files with Iceberg metadata. Because you
enabled **Write Delta Format** in Part 2, a Delta transaction log is also present.


> **Instructor demo:** If you look at the GCS bucket, you can see the folder structure
> Fivetran created: a `data/` folder holding the shared Parquet files, a `metadata/`
> folder for the Iceberg snapshots and schemas, and a `_delta_log/` folder for the Delta
> transaction log. The Delta metadata is present because we explicitly enabled it
> for this demonstration. Both sets of metadata reference the same Parquet data;
> Iceberg is written by default, while Delta is optional.
>
> Since **BigLake Metastore** was enabled, Fivetran registers external Iceberg tables
> in BigQuery. The instructor verifies registration and query access in Part 4.

---

## Part 4: Instructor demo — Querying the data from BigQuery

**Instructor-led only:** Attendees follow the demonstration; no attendee BigQuery
Studio login is required. Attendees confirm their own completed sync in Fivetran.

1. In BigQuery Studio, select the **BigQuery Project ID** configured in Part 2.
2. In Explorer, locate a participant's dataset:
   `<dataset_prefix><firstname>_<lastname>_pg_agriculture`
   (for example, `bqp001_jane_doe_pg_agriculture`). Confirm it belongs to that participant.
3. Open `coffee_prices` and inspect its external-table details and Iceberg metadata
   location in the participant's GCS path. Explain that BigQuery queries the data
   stored in GCS through the external table registered by Fivetran.
4. Open a SQL query tab. Replace every placeholder below with the selected participant's
   values, keeping the backticks around the fully qualified table name. Set the query
   processing location to match the dataset location (`us-east4` for the provisioned lab).

   ```sql
   SELECT *
   FROM `<project_id>.<dataset_prefix><firstname>_<lastname>_pg_agriculture.coffee_prices`
   LIMIT 10;
   ```

5. Run the query and show the returned rows. For example, the table identifier for
   Jane Doe is `your-project-id.bqp001_jane_doe_pg_agriculture.coffee_prices`;
   replace `your-project-id` with the actual lab project ID.

**Checkpoint:** The instructor shows rows from the selected participant's
`coffee_prices` external Iceberg table. Attendees can connect their Fivetran sync,
participant-specific GCS path, and registered BigQuery dataset to the query result.

If the dataset or table is missing, verify the project ID, participant's schema prefix,
BigLake toggle, completed sync, and Fivetran service-account permissions. If the table
is visible but the query fails, check the instructor's query/data access and the query
processing location before retrying.

---

## What you did

You configured a Managed Data Lake Service destination on Google Cloud Storage and
synced the PostgreSQL `agriculture` tables as Apache Iceberg, with optional Delta
metadata explicitly enabled for the storage demo. You then observed the instructor
query `coffee_prices` from a participant-specific BigQuery dataset. BigLake Metastore
(classic) registers these external Iceberg tables while the underlying data remains
in GCS; the default Fivetran Catalog also remains available.

## Further reading

- [Managed Data Lake Service](https://fivetran.com/docs/managed-data-lake-service)
- [Setup guide — see the GCS section](https://fivetran.com/docs/managed-data-lake-service/setup-guide)
- [Metadata catalogs — including BigLake metastore](https://fivetran.com/docs/managed-data-lake-service/metadata-catalogs)
- [Managed Data Lake Service tutorials](https://fivetran.com/docs/managed-data-lake-service/tutorials)
