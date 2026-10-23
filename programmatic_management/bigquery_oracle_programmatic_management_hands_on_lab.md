# Hands-on Lab: Programmatic Management (Oracle) (BigQuery)

*BigQuery edition — October 2026. Adapted from the corresponding lab worksheet; validate with the provisioned lab resources before delivery.*

Thank you for registering for our hands-on lab. This worksheet has everything you need to build an
end-to-end Fivetran pipeline **programmatically** using the REST API and Postman. Please read through
the requirements first. If you can't meet them, let us know and we'll rebook you for another workshop.

---

## Objective

Use the Fivetran REST API (via Postman) to authorize, then create a **Group**, a **Destination**, and
a **Connection**, start an initial sync, and retrieve the sync status and connection details — mapping
the object model from Module 1 onto real API calls.

## Requirements

- A web browser (Chrome or Firefox)
- Postman (desktop app or web)
- Python 3 for the local service-account JSON encoding helper in Part 5
- A Fivetran account

## Housekeeping

- You'll be invited to the **PROG_MANAGEMENT_HANDS_ON_LAB** Fivetran account.
   - **Do not** set up a new account or trial — you'll receive an invite to a dedicated account before the lab.
- Confirm you received the invite. It's sent to the email you registered with and comes from
  `notifications@fivetran.com`.
- The lab uses a **BigQuery** data warehouse (destination) and an **Oracle** database (source).

## What you'll do in this lab

- Access the provided resources by logging in to the dedicated Fivetran account.
- Access and sign up for Postman.
- Configure Postman by importing the Fivetran REST API collection and setting up authorization.
- Create a group with the REST API.
- Create a BigQuery destination with the REST API.
- Create an Oracle connection and trigger an initial sync with the REST API.
- Retrieve the sync status and connection details with the REST API.

## Part 0: Credentials

**Google BigQuery**

- Provided by your instructor via a 1Password link. The link includes the following fields:
   - **BigQuery Project ID**: the GCP project that hosts the lab's BigQuery datasets.
   - **BigQuery Service Account**: the email of the service account Fivetran uses to connect.
   - **BigQuery Service Account JSON**: the service account's private key. Copy the entire value, including the opening `{` and closing `}`.
   - **BigQuery Dataset Prefix**: a prefix that is unique to your lab (for example, `bqp001_`). Prepend it to every destination dataset name in this worksheet.
   - **BigQuery Location**: the data location of the lab's datasets (for example, `US`).
   - When you paste these credentials into the destination setup form, make sure you select **SaaS** as the deployment model.

- **Oracle**
   - Provided by your instructor via a 1Password link.

> Terminology note: Fivetran renamed "connector" to **connection** across the UI, docs, and REST API.
> In Postman the folder may appear as **Connection Management** (older exports say "Connector
> Management"); the request names and the `/v1/connections` paths are what matter. The `/connectors`
> paths still work for backward compatibility.

---

## Part 1: Accessing the provided resources

### Access Fivetran (new users)

1. Log in to Fivetran at `https://fivetran.com/login`. If you just signed up, use the username and
   password from your sign-up.
2. You'll go through a new-user flow. When prompted to **Tell us about yourself**, click **Other**.
3. When asked **What software do you currently use**, click **Skip**.
4. When prompted **Get ready to set up your first connection**, click **Let's go**.
5. Close the **You're now on updated pricing** pop-up if it appears.

### Access Fivetran (existing users)

1. Log in with your existing Fivetran credentials.
   - If you forgot your password, use the **Forgot your
   password?** link.
2. Switch to the `PROG_MANAGEMENT_HANDS_ON_LAB` account via the drop-down menu in the top left.

---

## Part 2: Accessing and signing up for Postman

1. Go to `https://postman.com`.
2. Click **Sign Up for Free** (top right).
3. Fill in your email, a username, and a password, then click **Create Free Account**. Adjust the
   username if it's already taken.
4. Verify your account with the code sent to your email, and paste it into the **Verification code**
   field.
5. Fill out your name and role (the values don't matter for this lab).
6. When your workspace is ready, you'll land on the Postman home screen.

---

## Part 3: Configuring Postman

1. Import the collection: click the **...** (more actions) menu, then **Import**.
2. Paste this URL into the Import screen:

   ```
   https://fivetran.com/assets-docs/postman/rest_api_collection.json
   ```

3. The import completes in a few seconds. The **Fivetran REST API** collection appears in the top-left
   of your workspace.
4. Set up authorization using **collection variables** (the recommended method):
   - In Fivetran, go to **Your Username > API Key** and click **Generate new API key**. If a key
     already exists, confirm — this invalidates the old key and generates a new pair.
   - **Copy the API Key and API Secret immediately** (or keep the window open). They can't be retrieved
     after you leave the page.
   - In Postman, select the **Fivetran REST API** collection and open the **Variables** tab.
   - Paste the API Key onto the `api_key` line, and the API Secret onto the `api_secret` line — into
     **both** the *Initial Value* and *Current Value* columns.
   - Click **Save**. Postman handles the Base64 encoding and Basic auth header for you.

> **Checkpoint:** The collection is authorized. You should now be able to run any request. (If you see
> `401 Unauthorized`, re-check that the key and secret are saved in both value columns.)

---

## Part 4: Creating a group

1. In the **Fivetran REST API** collection, open the **Group Management** folder.
2. Select **Create a Group**. It opens on the **Params** tab by default.
3. Click the **Body** tab.
4. Supply a name in the JSON payload. Replace the `<string>` value with `firstname_lastname_bigquery`
   (use your actual first and last names):

   ```json
   {
     "name": "firstname_lastname_bigquery"
   }
   ```

5. Click **Send** to POST to the Groups endpoint. A successful response returns the created group with
   its `id`.
6. **Copy the `id`** and save it to a notepad — you'll need it for the destination and connection.
   (Each attendee's `id` is unique.)
7. Optional: in the Fivetran dashboard, open the **Destinations** tab to see your new group. It shows
   as incomplete until you add a destination.

> **Checkpoint:** You created a group with one POST request instead of clicking through the UI.

---

## Part 5: Creating a destination

1. In **Destination Management**, select **Create a Destination** (`POST /v1/destinations`).
2. In **Body**, replace the example with the JSON below. Use the group ID from Part 4 and the instructor's project ID and location. `region` is the Fivetran processing region; it is separate from BigQuery's dataset location. Use the instructor-approved processing region if it differs from this example.
3. Replace `SERVICE_ACCOUNT_JSON_AS_STRING` with the **entire** service-account JSON encoded as a JSON string, not just its PEM `private_key` field. Keep this value local to your request; do not publish the populated request or credentials.

   ```json
   {
     "group_id": "<your_group_id>",
     "service": "big_query",
     "region": "GCP_US_WEST1",
     "time_zone_offset": "-6",
     "run_setup_tests": true,
     "daylight_saving_time_enabled": true,
     "config": {
       "project_id": "BIGQUERY_PROJECT_ID",
       "data_set_location": "BIGQUERY_LOCATION",
       "secret_key": "SERVICE_ACCOUNT_JSON_AS_STRING"
     }
   }
   ```

   To encode the key without manually escaping it, save the complete provided JSON outside any repository as `bigquery-service-account.json`. Run this in a local Python interpreter (or a local script) and paste the resulting quoted string in place of `"SERVICE_ACCOUNT_JSON_AS_STRING"`:

   ```python
   import json
   from pathlib import Path
   key = json.loads(Path("bigquery-service-account.json").read_text())
   print(json.dumps(json.dumps(key)))
   ```

   This preserves the escaped newlines inside the private key. The printed result is a secret; do not share terminal output. Remove the temporary credential file after the exercise.

4. Click **Send**. Check the response and setup-test results, then confirm your destination is **Connected** in the dashboard. Omitting `hybrid_deployment_agent_id` uses SaaS deployment for this lab.
5. Follow the dataset-naming and automatic-connection guidance below before continuing.

> **Dataset naming:** BigQuery uses `project.dataset.table`; Fivetran destination schemas become datasets. Replace `<dataset_prefix>` with the **BigQuery Dataset Prefix** from your credentials, including its trailing underscore (for example, `bqp001_`). Use your own name as well, such as `bqp001_jane_doe_postgres`. Keep the exact dataset names, including case, throughout the lab.

> **Automatic Platform Connection:** If Fivetran creates a connection named `fivetran_metadata` for your new destination, open **Connections**, verify it belongs to **your** destination, then use **Setup → Delete connection** before creating any other connections. The shared project needs uniquely named datasets. This removes the connection; do not delete shared datasets in BigQuery.

> **Checkpoint:** Your group owns a BigQuery destination. Its `group_id` links it to the group created in Part 4.

---

## Part 6: Creating and syncing a connection

1. Open the **Connection Management** folder and select **Create a Connection**.
2. Click the **Body** tab and delete the existing JSON payload.
3. Paste in the payload below. (Generated from the REST API reference for *Create a Connection* with
   `oracle` as the type, minimized for this lab.)
   - Replace `<your_group_id>` with the `id` from Part 4, Step 6.
   - In `schema_prefix`, replace `<dataset_prefix>` with your BigQuery Dataset Prefix and `firstname_lastname` with your actual first and last names.
   - Replace `ORACLE_PASSWORD` with the password from the 1Password link.
   - Replace `ORACLE_HOSTNAME` with the hostname from the 1Password link.
   - Replace `ORACLE_DATABASE` with the service name or SID from the 1Password link.

   ```json
   {
     "group_id": "<your_group_id>",
     "service": "oracle",
     "trust_certificates": true,
     "trust_fingerprints": true,
     "run_setup_tests": true,
     "paused": false,
     "pause_after_trial": false,
     "sync_frequency": 1440,
     "data_delay_sensitivity": "NORMAL",
     "data_delay_threshold": 0,
     "daily_sync_time": "14:00",
     "schedule_type": "auto",
     "networking_method": "Directly",
     "destination_schema_names": "FIVETRAN_NAMING",
     "config": {
       "connection_type": "Directly",
       "update_method": "TELEPORT",
       "auth_method": "PASSWORD",
       "database": "ORACLE_DATABASE",
       "password": "ORACLE_PASSWORD",
       "port": 2484,
       "host": "ORACLE_HOSTNAME",
       "user": "the fivetran database user provided to you via 1Password",
       "schema_prefix": "<dataset_prefix>firstname_lastname_oracle"
     }
   }
   ```

4. Click **Send** to POST to the Connections endpoint. A success returns the new connection.
5. In the response, find the **`id`** (near the top of the payload) — this uniquely identifies your
   connection. **Copy and save it**; you'll need it to sync and to retrieve details.
6. In the dashboard, open the **Connections** tab — your connection shows as **Connected**.

### Force an immediate sync

7. The connection won't sync until its scheduled time, but you can force one now. Select **Sync
   Connection Data** (under Connection Management).
8. On the **Params** tab, under **Path Variables**, set `connectionId` to the `id` you copied in Step 5
   (replace `<string>` in the Value column).
9. On the **Body** tab, replace `<boolean>` with `true`:

   ```json
   {
     "force": true
   }
   ```

10. Click **Send**. A success confirms the sync was triggered.
11. In Fivetran, open the connection you created. You'll see the sync has started; it should complete
    in about a minute.
    - To refresh the page sooner, do a hard refresh: **Cmd + Shift + R** (macOS) / **Ctrl + Shift + R**
      (Windows/Linux).

> **Checkpoint:** You triggered a sync programmatically — this is the exact `POST
> /v1/connections/{id}/sync` call the Airflow operator makes under the hood.

---

## Part 7: Retrieving sync status and connection details

1. Under **Connection Management**, select **Retrieve Connection Details**.
2. On the **Params** tab, under **Path Variables**, set `connectionId` to the `id` from Part 6, Step 5.
3. Click **Send** to GET the connection details. A success returns the full connection object.
4. Find `sync_state` under the `status` object. Depending on timing, you'll see one of:
   - `scheduled` — sync is waiting to run
   - `syncing` — sync is currently running
   - `paused` — sync is paused
   - `rescheduled` — waiting for more source API calls to become available
5. The `succeeded_at` field shows the last successful sync time. Together these confirm your sync
   succeeded.

The Retrieve Connection Details response also returns everything you need to know about the connection,
including:

- Who created it and when
- Sync frequency and schedule
- Networking information
- Connection details such as hostname, username, and update method

> Sensitive values like passwords or tokens are **never** exposed in this response.

> **Checkpoint (lab complete):** You built and verified an end-to-end pipeline — group → destination →
> connection → sync → status — entirely through the REST API. Everything you just did by hand in
> Postman is exactly what Terraform, Airflow, and the MCP server automate.

---

## What you did

| Object | Endpoint used |
|---|---|
| Group | `POST /v1/groups` |
| Destination | `POST /v1/destinations` |
| Connection | `POST /v1/connections` |
| Sync (operation) | `POST /v1/connections/{connectionId}/sync` |
| Status & details | `GET /v1/connections/{connectionId}` |

- [BigQuery API configuration](https://fivetran.com/docs/destinations/bigquery/api-configuration)

## BigQuery references

- [Fivetran BigQuery setup](https://fivetran.com/docs/destinations/bigquery/setup-guide)
