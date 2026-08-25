# Hands-on Lab: Activations

*Updated July 2026. Verified against the current Fivetran Activations documentation.*

Thank you for registering for our hands-on lab. This worksheet has everything you need to
build a working activation pipeline in Fivetran Activations — from connecting a source and
destination through running and observing a sync. Please read through the requirements first.
If you can't meet them, let us know and we'll rebook you on another workshop.

> Note on naming: the product is **Fivetran Activations** (formerly Census). You'll navigate
> to it from the **Activations** tab in the Fivetran dashboard. A few backend identifiers in
> this lab still literally contain "census".

---

## Objective

Use **Fivetran Activations** to connect a Snowflake **source** and a Braze **destination**, model data with a **Basic Dataset** and a **Segment**, then configure, test, run, and observe an **activation sync** — reverse-ETL from your warehouse into an activation tool, end to end.

---

## Requirements

- **Web browser** — Chrome or Firefox.

- **Fivetran account access**

- You'll be invited to a dedicated lab Fivetran account (**`ACTIVATIONS_HANDS_ON_LAB`**). 
   - **Do not create a new account or trial** — you'll receive an invite before the lab.
- Confirm you received the invite. It's sent to the email you registered with for the training, 
  from `notifications@fivetran.com`.

- **Snowflake**
   - Credentials provided by your instructor (via a 1Password link).

- **Braze**
   - Credentials provided by your instructor (via a 1Password link).

---

## What you'll do in this lab

1. Add an activation source (Snowflake)
2. Add an activation destination (Braze)
3. Create a Basic Dataset with SQL
4. Build a Segment
5. Configure and run an activation sync (with a test sync first)
6. Use the observability tools to track and monitor the sync

---

## Part 0: Credentials

- **Snowflake** — Provided by your instructor via a 1Password link.
- **Braze** — Provided by your instructor via a 1Password link.

Keep the 1Password link open; you'll copy several values from it during the lab.

---

## Part 1: Accessing the provided resources

**New users**

1. Log in to Fivetran at `https://fivetran.com/login` with the username and password you used
   when you signed up.
2. You'll go through a new-user flow:
   - When prompted to *Tell us about yourself*, click **Skip**.
   - When asked *What software do you currently use*, select **Salesforce** and click
     **Next** (this is a throwaway selection — the choice doesn't matter here).
   - At *Get ready to set up your first connection*, click **Let's go**.
   - Close the *You're now on updated pricing* pop-up.

**Existing users**

1. Log in with your existing Fivetran credentials (use *Forgot your password?* if needed).

**Everyone**

2. Using the account drop-down menu at the top, switch to the **`ACTIVATIONS_HANDS_ON_LAB`**
   account you were added to.

✅ **Checkpoint:** You're logged in and viewing the `ACTIVATIONS_HANDS_ON_LAB` account.

---

## Part 2: Adding an activation source (Snowflake)

1. Click the **Activations** tab.
2. Click **Activations Sources** at the top, then **+ Add an Activation Source**.
3. Select **Snowflake**.
4. In **Step 1**, select the **Basic Sync Engine**.
5. In **Step 2 — Configure Credentials**, enter the Snowflake credentials from the 1Password link.
   - **Name:** `{firstname}_{lastname}_snowflake` (replace with your actual first and last
     name).
   - **Snowflake Account Name** 
   - **Query Execution Warehouse**
   - **User** 
   - **Uncheck** *Use Key Authentication (Advanced)*.
   - **Password**
6. Leave **Step 3 — Configure Advanced Settings** as-is.
7. In **Step 4 - Configure Processing Region and Storage Backend**, Select:
   - **Data Processing Location** US
   - **Fivetran Processing Cloud Provider** GCP
   - **GCP Region** us-central1
   - **General object storage backend** Fivetran-managed storage
9. Click **Connect** (bottom right), then **Confirm**.
10. Let the setup tests run. When they pass, click **Finish**.

✅ **Checkpoint:** You're on the Sources overview screen and your Snowflake source shows as
connected.

---

## Part 3: Creating an activation destination (Braze)

1. Click the **Activation Destinations** tab, then **+ Add an Activations Destination**.
2. Search for **Braze** and select it.
3. Enter the Braze credentials and click **Connect**:
   - **Name:** `{firstname}_{lastname}_braze`
   - Set the **Endpoint URL**, **API Key**, and **Data Import Key (for Cohorts Only)** to
     the values from the 1Password link.
4. Let the connection tests run. When they pass, click **Finish**.

✅ **Checkpoint:** You're on the overview of your Braze destination.

---

## Part 4: Creating a Dataset

1. Go to the **Datasets** tab and click **+ New Dataset**.
2. Select **Basic Dataset** and click **Next**.
3. In *Create a New Dataset*, set:
   - **Name:** `{firstname}_{lastname}_high_intent_users`
   - **Data Source:** the Snowflake source you created in Part 2.
   - **Query Data Source with:** ensure **SQL** is selected.
4. Paste this query into the editor:

```sql
SELECT
  u.user_id::STRING AS user_id,
  u.first_name,
  u.last_name,
  u.email,
  u.city,
  u.state,
  TO_TIMESTAMP_NTZ(u.joined_at, 'MM/DD/YYYY HH24:MI:SS') AS joined_at
FROM CENSUS_HOL_DATABASE.BRAZE_ANALYTICS.USERS u
LEFT JOIN CENSUS_HOL_DATABASE.BRAZE_ANALYTICS.PURCHASES p
  ON u.user_id = p.user_id
WHERE p.user_id IS NULL;
```

5. Click **Show preview** at the bottom. When the query finishes, click **Create Dataset**.
6. In the **Type** box, click **Edit** and ensure **`USER_ID`** is selected as the unique ID.
7. In the **Columns** tab, find the **State** column and check the box under **Enumerated**. You will need this for your **Segment**.

✅ **Checkpoint:** You land on the dataset preview page showing your query results.

---

## Part 5: Creating a Segment

1. Go to the **Segments** tab and click **+ Add a Segment**.
2. **Segment Name:** `{firstname}_{lastname}_segment`.
3. Add the first condition:
   - Select the **Joined At** column.
   - Operator **after**, value **2025-11-09**.
4. Click **+ Add condition**.
5. Add the second condition:
   - Select the **State** column.
   - Operator **is**, value **Michigan**, which should be a value from a drop-down menu.
6. Click **Preview Results** (top right). Calculation takes ~1 minute.
7. Click the refresh button in the **Current Size** preview to confirm.

✅ **Checkpoint:** The current size shows **~326** and **~3%**. These may differ for you as the demo data we have in Braze can often change.

8. Click **Save Segment**.

---

## Part 6: Activating data (build and run a sync)

1. Go to the **Activations** tab and click **+ Create a sync**.
2. Select **Existing Dataset & Segments**.
3. Select the **Snowflake** connection, then the data source and the **segment** you created.
4. Select the **Braze** destination you created.
5. Select the **User** object.
6. **Select a Sync Behavior:** choose **Update or Create**.
7. **Select a Sync Key:** map **`USER_ID`** (segment) to **External User ID** (Braze).
8. Under **Set Up Braze Field Mappings**, choose **Sync All Columns** and leave the
   auto-generated mappings as-is. Then click **+ Add Mapping**:
   - Click **Source value → Constant Value**, enter **`Y`**, and click **Save**.
   - Click the **Destination** field, enter **`High_Intent_Flag`**, and click
     **Create 'High_Intent_Flag' as a custom field**.
9. Under **Run a Test Sync**, click **Run Test** (then confirm) to load a single record and
   verify your configuration before the full run. The test completes in ~30 seconds.
   - Your instructor will demonstrate viewing the record in Braze.
10. Click **Next**.
11. **Label** the sync: `{firstname}_{lastname}_snowflake_to_braze`.
12. Under **Set up a Trigger**, select **Scheduled**, set the schedule to **Weekly, every
    Thursday at 9:00 UTC**, and click **Create**.
13. On the sync overview, click **Run Now** to execute. The sync completes in ~1 minute.

✅ **Checkpoint:** The sync run shows as completed on the sync overview.

---

## Part 7: Observing the sync

1. Click **Sync History** to view results over time.
2. With your instructor, review the sync dashboard — the **Activity**, **Alerts**, and **API
   Inspector** views — to see how you'd monitor and troubleshoot syncs in production.

✅ **Checkpoint:** You can locate your sync run in Sync History and open the Activity and API
Inspector views.

---

## What you did

You connected a Snowflake **activation source** and a Braze **activation destination**,
modeled data with a **Basic Dataset**, targeted it with a **Segment**, configured an
**activation sync** (sync key, field mappings, a constant custom field, and a schedule),
validated it with a **test sync**, ran it, and observed it in the **observability** tools —
the full Activations workflow end to end.
