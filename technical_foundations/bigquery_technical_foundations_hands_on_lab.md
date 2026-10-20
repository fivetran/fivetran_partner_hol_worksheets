# Hands-on Lab: Technical Foundations (BigQuery)

*Updated October 2026. Verified against the current Fivetran Technical Foundations documentation.*

Thank you for registering for our hands-on lab. This worksheet provides everything you need to prepare and to work through the lab. Please read through the requirements first. If you can't meet them, let us know and we'll rebook you on another workshop.

---

## Objective

Set up a **Destination** and a **Connection** in Fivetran, sync source data into Google BigQuery, deploy a Quickstart transformation, and manage notifications — putting the core Fivetran object model and workflow into practice end to end.

## Requirements

- **Web browser**
  - Chrome or Firefox (latest version recommended)

---

## Housekeeping

**Invitation to the `TECH_FOUNDATIONS_HANDS_ON_LAB` Fivetran account**

- Do **not** set up a new account or trial. You will receive an invite to a dedicated Fivetran account prior to the lab.
- Confirm that you have received the invite to the Fivetran account.
- The invite is sent to the email address you used to sign up for this lab and originates from `notifications@fivetran.com`.

**Google BigQuery data warehouse**

- Credentials provided by your instructor (see Part 0).

**PostgreSQL database**

- Credentials provided by your instructor (see Part 0).

---

## What you'll do in this lab

1. Create a destination in Fivetran
2. Create and sync a connection
3. Create your own Fivetran Platform Connection
4. Deploy a Quickstart transformation
5. Manage notifications

> User, team, and role management is covered in the Operations Management hands-on lab.

---

## Part 0: Credentials

### Google BigQuery

- Provided by your instructor via a 1Password link. The link includes the following fields:
   - **BigQuery Project ID**: the GCP project that hosts the lab's BigQuery datasets.
   - **BigQuery Service Account**: the email of the service account Fivetran uses to connect.
   - **BigQuery Service Account JSON**: the service account's private key. Copy the entire value, including the opening `{` and closing `}`.
   - **BigQuery Dataset Prefix**: a prefix that is unique to your lab (for example, `bqp001_`). You'll use it when naming your connections in Parts 3 and 4.
   - **BigQuery Location**: the data location of the lab's datasets (for example, `US`).
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
2. Search for **BigQuery** and select it.
3. Give the destination the name `<firstname>_<lastname>_bigquery`, replacing `<firstname>` and `<lastname>` with your actual first and last names. Click **Add**.
4. In the **Project ID** field, enter the **BigQuery Project ID** from your credentials.
5. In the **Deployment model** section, select **SaaS**.
6. Set the **Use own Service Account** toggle to **ON**, then paste the full **BigQuery Service Account JSON** value from your credentials into the **Service Account Private Key** field.

   > **Note:** By default, Fivetran connects with a Fivetran-managed service account, which must first be granted the **BigQuery User** role in the GCP project's IAM settings. Attendees don't have IAM access to the lab project, so this lab uses the pre-provisioned service account listed in your credentials instead.

7. For **Data Location**, select the **BigQuery Location** from your credentials.
8. Leave **Use GCP Service Parameter** set to **OFF**.
9. Leave the remaining fields at their default values. Do not make any changes.
10. Click **Save & Test**.
11. Wait for the setup tests to complete. When they pass, you'll see all connection tests marked successful (including **Connection** and **Check Permissions**).
12. Click **View Destination** (or **Continue**) to proceed.

> **Note:** BigQuery has no separate database layer: each Fivetran schema is created as a **dataset** in the GCP project. Every destination you create automatically includes a free **Fivetran Platform Connection**, which loads metadata about your account, connections, and usage into a dataset named `fivetran_metadata`.

Everyone in this lab shares the same BigQuery project, so every attendee's automatically created Platform Connection would load into the same `fivetran_metadata` dataset and overwrite each other's data. Before you add any other connection, delete the automatically created one. You'll create your own, uniquely named Platform Connection in Part 4.

13. Navigate to the **Connections** tab.
14. Find the **Fivetran Platform** connection with the schema `fivetran_metadata` that belongs to your destination (`<firstname>_<lastname>_bigquery`), and click it.

    > **Note:** Make sure you select the connection for your own destination, not someone else's.

15. Click the **Setup** tab, then click **Delete connection** and confirm the deletion.

---

## Part 3: Creating a connection

> In current Fivetran terminology, a **connector** is the reusable source type (for example, PostgreSQL), and a **connection** is the configured instance you set up from it. You can create many connections from the same connector.

1. Navigate to the **Connections** tab and, in the top-right corner, click **Add connection**.
2. Search for **PostgreSQL**, hover over the connector tile, and click **Set up**.
3. Select the destination you created in Part 2 (make sure you don't use someone else's destination), then click **Select**.
4. The value you enter in the **Destination schema prefix** field becomes the name of your connection and of the BigQuery dataset it loads into. Use the format `<dataset_prefix><firstname>_<lastname>_postgres`, replacing `<dataset_prefix>` with the **BigQuery Dataset Prefix** from your credentials and `<firstname>` and `<lastname>` with your actual first and last names (for example, `bqp001_jane_doe_postgres`).
5. Populate the setup form with the provided PostgreSQL credentials.
6. For **Authentication Method**, select **Connect with username and password**.
7. Leave **Connection Method** at its default value, **Connect directly**.
8. For **Update Method**, select **Query-Based**.
   - *Context:* PostgreSQL's older XMIN and Fivetran Teleport Sync methods have been sunset and replaced by **Query-Based** change data capture. The other available method is **Logical replication** (using the `pgoutput` plugin). Existing Teleport/XMIN connections keep working.
9. For destination naming, select **Fivetran naming** (the default, which standardizes schema, table, and column names). The alternative, **Source naming**, preserves original UTF-8 names but is not compatible with Quickstart transformations.
10. Click **Save & Test**.
11. When prompted, confirm the TLS certificate by selecting the certificate and clicking **Confirm**.
12. Wait for the setup tests to complete. When they pass, you'll see all connection tests marked successful.
13. Click **Continue**.
14. Fivetran now fetches all tables, schemas, and columns for the database.
15. The fivetran database user provided to you via 1Password has access to the `agriculture` schema. Make sure the `agriculture` schema and all of its tables are selected, then click **Save & Continue** to proceed.
16. For handling schema changes, select **Allow all**, then click **Continue** (or **Save**).
17. Click **Start Initial Sync**. Wait for the initial (historical) sync to finish, then let at least one **incremental** sync complete. At least one incremental sync must complete for this lab.
18. The sync will finish in about 1–2 minutes. A successful sync shows the connection status as **Active** with the synced tables and row counts.

> **Checkpoint — sync schedule and sync modes:**
> - By default, connections sync on a **fixed interval of every 6 hours**. You can change this on the connection's **Settings** tab (options range from 1 minute up to 24 hours; a **Cron** schedule is also available on Enterprise/Business Critical plans).
> - Fivetran supports two **sync modes**: **Soft delete** (the default — deleted source rows are marked with `_fivetran_deleted = TRUE` rather than removed) and **History mode** (SCD Type 2, which tracks every version of a row using `_fivetran_start`, `_fivetran_end`, and `_fivetran_active`).

---

## Part 4: Creating your Fivetran Platform Connection

The **Fivetran Platform Connection** (formerly the Fivetran Log Connector) is free and loads logs and metadata about your connections, destination, and usage. You'll create your own in this part, so its dataset is unique to you.

> **Note:** A Platform Connection needs at least one other connection in the destination, which is why you create it after the PostgreSQL connection.

1. Navigate to the **Connections** tab and, in the top-right corner, click **Add connection**.
2. Search for **Fivetran Platform**, hover over the connector tile, and click **Set up**.
3. Select the destination you created in Part 2, then click **Select**.
4. In the **Destination schema** field, use the format `<dataset_prefix><firstname>_<lastname>_fivetran_log` (for example, `bqp001_jane_doe_fivetran_log`).
5. Leave the **Account Level Connector** toggle set to **OFF**. When it's on, the connection syncs logs for every destination in the account, not only yours.
6. Click **Save & Test**.
7. Wait for the setup tests to complete, then click **Continue**.
8. Leave the default schema selection unchanged, then click **Save & Continue** (or **Continue**).
9. Click **Start Initial Sync** and wait for the sync to finish. A successful sync shows the connection status as **Active**.

---

## Part 5: Creating transformations

Now we'll explore adding **Transformations**. For simplicity, we'll start with **Quickstart transformations** (no-code, pre-built data models that run in your destination).

> If you'd like to go deeper on the dbt Core and dbt Cloud integrations, we recommend our Transformations training and hands-on lab, which focus on those integrations.

1. Navigate to the **Transformations** tab.
2. Quickstart data models can be added for any connection you configure that has a supported pre-built package.
3. Select the **Fivetran Platform Connection** you created in Part 4 (schema `<dataset_prefix><firstname>_<lastname>_fivetran_log`) and click **Add & Run**.
4. That's it. The Quickstart package for the Fivetran Platform Connection has been added and will run whenever your Platform Connection finishes syncing.
5. Click into the **Fivetran Platform** transformation for your connection to explore its **Run log**, **Data Models**, **Schedule**, and **Details**.

> **Note:** Transformations are billed in monthly **model runs** (not MAR), and the first **5,000 model runs per month are free**. They are not supported on the Managed Data Lake Service.

---

## Part 6: Managing notifications

Now we'll learn how to manage **Notifications**. In Fivetran, a notification is an automatically generated email that informs you about important activity, such as connection failures and other errors. Notification settings are unique to your user.

1. Navigate to your user in the bottom-left and click the drop-down.
2. Click **Notifications** to open the Notifications page.
3. On the **Connections** tab, the **Active Notification Subscriptions** section lists all connections you currently receive notifications for.
4. Remove notifications for the PostgreSQL connection by selecting its checkbox and clicking **Remove selected**.
5. To add notifications for a connection, click **+ Add**, select the connection(s) you want, and click **Apply changes**. Add back the connection you just removed.
6. You can do the same for transformations by clicking the **Transformations** tab and repeating the process.
7. You can also control which account-level notifications you receive. Click the **Account** tab.
   - Here you'll find toggles for notifications such as **Monthly spend warning**, **Low spend warning**, and **System Outages**.
8. To disable all notifications for your user, set the **Notifications enabled** toggle at the top of the page to **OFF**.
9. As an Account Administrator, you can also manage notifications for other users and email recipients. Click **Switch Recipient**.
10. Select another user or email recipient in your account.
11. Enable **System Outages** notifications for this user.
12. When finished, click **Back to your notifications** to return to your own settings.

---

## What you did

You've completed the Fivetran Technical Foundations hands-on lab. You created a destination, set up and synced a connection, created your own Fivetran Platform Connection, deployed a Quickstart transformation, and managed notifications.
