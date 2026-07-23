# Hands-on Lab: Operations Management

*Updated July 2026. Verified against the current Fivetran Operations Management documentation.*

Thank you for registering for our hands-on lab. This worksheet is the practical companion to the Operations Management training. You'll build the access model hands-on: a destination, a connection created via the REST API, and then a team, a user, and a custom role. SSO/SAML and Customer-Managed Keys are covered as guided walkthroughs in the deck rather than in this lab, because they require an identity provider and a cloud KMS that we can't provision in a shared lab environment. Please read through the requirements first. If you can't meet them, let us know and we'll rebook you on another workshop.

---

## Objective

Build the Fivetran Operations Management access model end to end: create a **Destination**, create a **Connection** via the REST API, then build a **Team**, invite a **User**, and create a **custom Role** — putting the role-based access concepts from the training into practice.

## Requirements

- **Web Browser** — Chrome or Firefox
- **Terminal** — Terminal (macOS/Linux) or Command Prompt / PowerShell (Windows)

## Housekeeping

- Invitation to the `ADMIN_MANAGEMENT_HANDS_ON_LAB` Fivetran account.
- **Do not set up a new account or trial** — you'll receive an invite to a dedicated
  Fivetran account prior to the lab.
- Confirm you received the invite. It was sent to the email you used to register and
  originates from `notifications@fivetran.com`.
- **Snowflake data warehouse** — provided by your instructor via a 1Password link.
- **Postgres database** — provided by your instructor via a 1Password link.

## What you'll do in this lab

In this lab you will:

- Take an admin tour of the Fivetran dashboard
- Create a destination in Fivetran
- Create a connection using the REST API
- Create a team
- Add a user
- Create a custom role and attach it to the team

## Part 0: Credentials

**Snowflake**
   - Provided by your instructor via a 1Password link.
      - Select **SaaS** as the deployment model when entering these
     credentials into the setup form. 

**Postgres**
   - Provided by your instructor via a 1Password link.

---

## Part 1: Accessing the provided resources

### Access Fivetran (new users)

1. Log into Fivetran at `https://fivetran.com/login`.
2. If you just signed up for the first time, use the username and password from your
   sign-up.
3. As a new user you'll go through a new-user flow. When prompted to *Tell us about
   yourself*, click **Skip**.
4. When prompted *What software do you currently use*, click **Other**
5. When prompted *Get ready to set up your first connection*, click **Let's go**.
6. Close out of the *You're now on updated pricing* pop-up.

### Access Fivetran (existing users)

1. If you already have a Fivetran account, log in with your existing credentials. 
   - If you forgot your password, click the **Forgot your password?** link.
2. Switch to the account you've been added to using the account drop-down menu.

---

## Part 2: Fivetran dashboard overview (admin lens)

The instructor will navigate through each of the sections discussed in Module 1 —
Overview, Connections, Destinations, Transformations, Alerts, and **Account Settings →
Users & Permissions / Billing & Usage** — and discuss them from an administrator's
point of view. Follow along to get a lay of the land, paying special attention to where
**Users & Permissions** lives, since that's where the rest of this lab happens.

**Checkpoint:** you can locate Users & Permissions under Account Settings, and you can
find Billing & Usage.

---

## Part 3: Creating a destination

1. Navigate to the **Destinations** tab and click **Add Destination**.
2. Search for **Snowflake**.
3. Name it `<firstname>_<lastname>_snowflake` (replace with your actual first and last
   name), then click **Add**.
4. Enter the provided credentials. Make sure you select **PASSWORD** for the Auth and
   paste in the password.
5. Select **SaaS Deployment** in the *Select deployment model* section.
6. Leave the remaining fields as their default values — do not make any changes.
7. Click **Save & Test** and wait for the setup tests to complete.
8. On success, click **View Destination** to continue.

**Checkpoint:** the destination shows a **Connected** status.

---

## Part 4: Creating a connection with the REST API

This part ties directly to Module 7 (Programmatic Management). You'll authenticate with
an API key and create a connection with a single API call.

### Generate an API key

1. Navigate to your **Username → API Key**.
2. Click **Generate new API key**. If prompted, confirm — note that if a key already
   exists, this invalidates the old key and generates a new one.
3. Copy the **Base64-encoded API key** and save it somewhere you can access later (a
   notepad or text file).

### Get your destination group ID

1. Navigate to the **Destinations** tab and search for the destination you created.
2. Click into it to open the **Destination Overview** tab.
3. Scroll to find the **Destination Group ID**, copy it, and save it.

### Prepare the request

Take the code below and replace the placeholders:

- `YOUR_BASE64_ENCODED_API_KEY` → the API key you copied
- `YOUR_GROUP_ID` → the Destination Group ID you copied
- `YOUR_CONNECTION_NAME` → your first and last name followed by `_postgres`
- `POSTGRES_HOSTNAME` → the actual Postgres hostname shared with you
- `POSTGRES_PASSWORD` → the actual Postgres password shared with you

**macOS / Linux (Terminal):**

```bash
curl -X POST "https://api.fivetran.com/v1/connections" \
  -H "Authorization: Basic YOUR_BASE64_ENCODED_API_KEY" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{
    "group_id": "YOUR_GROUP_ID",
    "service": "google_cloud_postgresql",
    "trust_certificates": true,
    "trust_fingerprints": true,
    "run_setup_tests": true,
    "paused": false,
    "pause_after_trial": false,
    "sync_frequency": 360,
    "data_delay_sensitivity": "NORMAL",
    "schedule_type": "auto",
    "networking_method": "Directly",
    "destination_schema_names": "FIVETRAN_NAMING",
    "config": {
      "connection_type": "Directly",
      "update_method": "QUERY_BASED",
      "host": "POSTGRES_HOSTNAME",
      "port": 5432,
      "database": "industry",
      "user": "fivetran_hol_user",
      "password": "POSTGRES_PASSWORD",
      "schema_prefix": "YOUR_CONNECTION_NAME"
    }
  }'
```

**Windows (PowerShell):** PowerShell handles quotes differently, so put the JSON body
in a variable and pass it with `--data`. `curl.exe` is included on Windows 10+ — call
it as `curl.exe` (not the `curl` alias) so the flags work as written:

```powershell
$body = @'
{
  "group_id": "YOUR_GROUP_ID",
  "service": "google_cloud_postgresql",
  "trust_certificates": true,
  "trust_fingerprints": true,
  "run_setup_tests": true,
  "paused": false,
  "pause_after_trial": false,
  "sync_frequency": 360,
  "data_delay_sensitivity": "NORMAL",
  "schedule_type": "auto",
  "networking_method": "Directly",
  "destination_schema_names": "FIVETRAN_NAMING",
  "config": {
    "connection_type": "Directly",
    "update_method": "QUERY_BASED",
    "host": "POSTGRES_HOSTNAME",
    "port": 5432,
    "database": "industry",
    "user": "fivetran_hol_user",
    "password": "POSTGRES_PASSWORD",
    "schema_prefix": "YOUR_CONNECTION_NAME"
  }
}
'@

curl.exe -X POST "https://api.fivetran.com/v1/connections" `
  -H "Authorization: Basic YOUR_BASE64_ENCODED_API_KEY" `
  -H "Accept: application/json" `
  -H "Content-Type: application/json" `
  --data $body
```

### Run and verify

1. Open a Terminal (macOS/Linux) or PowerShell (Windows) and run your prepared command.
2. On success you'll receive a JSON response with `"code": "Success"` and the new
   connection's details.
3. Verify by going to the **Connections** tab and searching for your connection name.

**Checkpoint:** your connection appears in the Connections tab.

---

## Part 5: Creating a team

Now you'll create a team for managing access to connections and destinations.

1. Navigate to **Account Settings** in the bottom left and click the drop-down.
2. Click **Users & Permissions**.
3. Click the **Teams** tab, then click **+ Add Team**.
4. In the **Team Name** field, enter `<firstname>_<lastname>_team` (replace with your
   actual first and last name).
5. For **Account role**, select **Destination Creator**.
6. Click **Create team**.

**Checkpoint:** your team appears in the Teams list with the Destination Creator role.

> **Why it matters:** assigning the role to the team (not the person) means anyone you
> add to this team inherits Destination Creator automatically — the scalable pattern
> from Module 3.

---

## Part 6: Creating and inviting a user

1. Navigate to the **Users** tab and click **+ Add User**.
2. Fill out **First Name** and **Last Name** with your actual names.
3. Fill out the **Email** field using an alias of your email address. For example, if
   your email is `angel.hernandez@fivetran.com`, alias it as
   `angel.hernandez+1@fivetran.com`.
4. **Skip adding an Account Role** — you'll assign access via the team you created, and
   the user will inherit the team's role. Click **Add User**.
5. The user appears in the list with a team assigned. It shows **invite pending** until
   the user accepts the emailed invite.

**Checkpoint:** the new user shows as invite pending with your team assigned.

> **Why it matters:** the user has no directly assigned account role — all of their
> access comes from the team. This is least privilege by inheritance.

---

## Part 7: Creating a custom role

Next you'll create a custom role and see how it attaches to a team. (Custom roles
require Enterprise or Business Critical — your lab account is provisioned for this.)

1. Click the **Roles** tab, then click **+ Add Role**.
2. In the **Role name** field, enter `<firstname>_<lastname>_role`.
3. In the **Description** field, enter `Google Sheets Only Role`.
4. Toggle **Account access** and leave the default values. *Making this an account role
   is required so it can be assigned to teams.*
5. Toggle **Destination access** and leave the default values.
6. For **Connection access**, click **Selected**.
7. For **Connections**, select **Create**.
8. For **Connector types**, search for **Google Sheets** and select it.
9. Scroll down and click **Save**.
10. Scroll to the available roles — your new role now appears and can be assigned to
    teams or users.
11. Edit your team and confirm you can now assign your custom role to it.

**Checkpoint:** your custom role appears in the roles list and is selectable when
editing your team.

> **Why it matters:** the custom role scopes connection creation to a single connector
> type (Google Sheets). Attaching it to a team is how you grant that narrow capability
> to a group without touching account-wide admin rights.

---

## What you did

You've now exercised the full Operations Management access model end to end: you toured
the dashboard as an admin, created a destination, used the REST API to create a
connection, and built a team, a user, and a custom role that all fit together through
inheritance. Everything you did here maps to a module in the deck — revisit Modules 1–3
and 7 to reinforce the concepts, and see the SSO/SAML and CMK modules for the security
topics that live outside this lab.