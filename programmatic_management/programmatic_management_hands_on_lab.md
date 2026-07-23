# Hands-on Lab: Programmatic Management

*Updated July 2026. Verified against the current Fivetran Programmatic Management documentation.*

Thank you for registering for our hands-on lab. This worksheet has everything you need to build an
end-to-end Fivetran pipeline **programmatically** using the REST API and Postman. Please read through
the requirements first. If you can't meet them, let us know and we'll rebook you on another workshop.

---

## Objective

Use the Fivetran REST API (via Postman) to authorize, then create a **Group**, a **Destination**, and
a **Connection**, start an initial sync, and retrieve the sync status and connection details — mapping
the object model from Module 1 onto real API calls.

## Requirements

- A web browser (Chrome or Firefox)
- Postman (desktop app or web)
- A Fivetran account

## Housekeeping

- You'll be invited to the **PROG_MANAGEMENT_HANDS_ON_LAB** Fivetran account. 
   - **Do not** set up a new account or trial — you'll receive an invite to a dedicated account before the lab.
- Confirm you received the invite. It's sent to the email you registered with and comes from
  `notifications@fivetran.com`.
- The lab uses a **Snowflake** data warehouse (destination) and a **Postgres** database (source).

## What you'll do in this lab

- Access the provided resources by logging into the dedicated Fivetran account.
- Access and sign up for Postman.
- Configure Postman by importing the Fivetran REST API collection and setting up authorization.
- Create a group with the REST API.
- Create a Snowflake destination with the REST API.
- Create a Postgres connection and trigger an initial sync with the REST API.
- Retrieve the sync status and connection details with the REST API.

## Part 0: Credentials

- **Snowflake**
   - Provided by your instructor via a 1Password link.
      - Select **SaaS** as the deployment model wherever the setup form asks.
- **Postgres**
   - Provided by your instructor via a 1Password link.

> Terminology note: Fivetran renamed "connector" to **connection** across the UI, docs, and REST API.
> In Postman the folder may appear as **Connection Management** (older exports say "Connector
> Management"); the request names and the `/v1/connections` paths are what matter. The `/connectors`
> paths still work for backward compatibility.

---

## Part 1: Accessing the provided resources

### Access Fivetran (new users)

1. Log into Fivetran at `https://fivetran.com/login`. If you just signed up, use the username and
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
   - In Postman, select the **Fivetran REST API** collection, open the **Variables** tab.
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
4. Supply a name in the JSON payload. Replace the `<string>` value with `firstname_lastname_snowflake`
   (use your actual first and last name):

   ```json
   {
     "name": "firstname_lastname_snowflake"
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

1. Open the **Destination Management** folder and select **Create a Destination**.
2. Click the **Body** tab and delete the existing JSON payload.
3. Paste in the payload below. (This is generated from the REST API reference for *Create a
   Destination* with **Snowflake** as the type, minimized to only what this lab needs.)
   - Replace `<your_group_id>` with the `id` from Part 4, Step 6.
   - Replace `SNOWFLAKE_PASSWORD` with the password from the 1Password link.
   - Replace `SNOWFLAKE_DATABASE` with the database from the 1Password link.
   - Replace `SNOWFLAKE_USER` with the user from the 1Password link.

   ```json
   {
     "group_id": "<your_group_id>",
     "service": "snowflake",
     "region": "GCP_US_WEST1",
     "time_zone_offset": "-6",
     "trust_certificates": "True",
     "trust_fingerprints": "True",
     "run_setup_tests": "True",
     "daylight_saving_time_enabled": "True",
     "config": {
       "role": "PROG_MANAGEMENT_HOL_ROLE",
       "auth": "PASSWORD",
       "external_storage_cloud_provider": "INTERNAL_STAGE",
       "database": "SNOWFLAKE_DATABASE",
       "password": "SNOWFLAKE_PASSWORD",
       "host": "a3209653506471-sales_eng_hands_on_lab.snowflakecomputing.com",
       "default_virtual_warehouse": "PROG_MANAGEMENT_HOL_WAREHOUSE",
       "port": 443,
       "user": "SNOWFLAKE_USER"
     }
   }
   ```

4. Click **Send**. Because `run_setup_tests` is `True`, Fivetran validates connectivity — this
   counts against the **source-interaction** rate-limit bucket, so avoid spamming it. A success returns
   the destination details.
5. In the dashboard, open the **Destinations** tab — your destination now shows as **Connected**.

> **Checkpoint:** The group owns a Snowflake destination. Note how the `group_id` links the destination
> back to the group — the same hierarchy from Module 1.

---

## Part 6: Creating and syncing a connection

1. Open the **Connection Management** folder and select **Create a Connection**.
2. Click the **Body** tab and delete the existing JSON payload.
3. Paste in the payload below. (Generated from the REST API reference for *Create a Connection* with
   `google_cloud_postgresql` as the type, minimized for this lab.)
   - Replace `<your_group_id>` with the `id` from Part 4, Step 6.
   - In `schema_prefix`, replace `firstname_lastname` with your actual first and last name.
   - Replace `POSTGRES_PASSWORD` with the password from the 1Password link.
   - Replace `POSTGRES_HOSTNAME` with the hostname from the 1Password link.

   ```json
   {
     "group_id": "<your_group_id>",
     "service": "google_cloud_postgresql",
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
     "destination_configuration": {
       "virtual_warehouse": "PROG_MANAGEMENT_HOL_WAREHOUSE"
     },
     "destination_schema_names": "FIVETRAN_NAMING",
     "config": {
       "connection_type": "Directly",
       "update_method": "TELEPORT",
       "auth_method": "PASSWORD",
       "database": "industry",
       "password": "POSTGRES_PASSWORD",
       "port": 5432,
       "host": "POSTGRES_HOSTNAME",
       "user": "fivetran_hol_user",
       "schema_prefix": "firstname_lastname_gcp_postgres"
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