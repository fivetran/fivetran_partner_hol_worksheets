# Hands-on Lab: Transformations (BigQuery)

*BigQuery edition — October 2026. Adapted from the corresponding lab worksheet; validate with the provisioned lab resources before delivery.*

Thank you for registering for our hands-on lab. This worksheet provides everything you need to prepare and complete the lab. In this lab you will build a complete Fivetran + dbt Core transformation end to end: stand up a BigQuery destination, sync the Fivetran Platform Connector, build and version a dbt project, install a Fivetran data model package, orchestrate it from Fivetran on an Integrated schedule, and review the transformed data in an instructor-led BigQuery demonstration. Please read through the requirements first. If you can't meet them, let us know and we'll rebook you on another workshop.

> **Completing this lab self-paced?** This worksheet is written for our instructor-led sessions, which use a provisioned Fivetran account and a BigQuery destination with instructor-provided credentials. If you're working through the lab on your own, use your own Fivetran partner account and any [Fivetran-supported destination](https://fivetran.com/docs/destinations) that supports dbt Core transformations. Skip the Housekeeping section and Parts 0–1, and use the remaining parts as an example of what each step looks like, adapting the destination setup, dbt profile, credentials, and naming to your own resources. If you can't provision your own resources, contact Fivetran to schedule an instructor-led session.

---

## Objective

Build an end-to-end Fivetran + dbt Core transformation: create a BigQuery **destination**, sync the **Fivetran Platform Connector**, version a **dbt project** in GitHub, install and configure the **fivetran_log** dbt package, and orchestrate it from Fivetran on an **Integrated** schedule to produce transformed tables reviewed in an instructor-led BigQuery demonstration.

## Requirements

- **Python 3.10 – 3.12** (dbt Core 1.11 requires Python 3.10+; 3.10–3.12 are the safest)
- Code editor (VS Code, Sublime, Notepad++, etc.)
- Web browser (Chrome)
- Terminal (macOS Terminal or Windows PowerShell)
- Git 2.38+ ([git-scm.com/downloads](https://git-scm.com/downloads))
- A GitHub account

## Housekeeping

- You will receive an invitation to the **TRANSFORMATIONS_HANDS_ON_LAB** Fivetran
  account prior to the lab.
- **Do not set up a new account or trial** — you'll receive an invite to a dedicated
  Fivetran account before the lab.
- Confirm you received the invite. It is sent to the email you used to register and
  originates from **notifications@fivetran.com**.

## What you'll do in this lab

In this lab you will:

1. Create a dbt project
2. Create a GitHub repository for the dbt project
3. Connect the GitHub repository to Fivetran
4. Set up a Fivetran Platform Connector
5. Create a transformation for the Fivetran Platform Connector
6. Review transformed data in an instructor-led BigQuery demonstration

---

## Part 0: Credentials

**Google BigQuery**

- Provided by your instructor via a 1Password link. The link includes the following fields:
   - **BigQuery Project ID**: the GCP project that hosts the lab's BigQuery datasets.
   - **BigQuery Service Account**: the email of the service account Fivetran uses to connect.
   - **BigQuery Service Account JSON**: the service account's private key. Copy the entire value, including the opening `{` and closing `}`.
   - **BigQuery Dataset Prefix**: a prefix that is unique to your lab (for example, `bqp001_`). Prepend it to every destination dataset name in this worksheet.
   - **BigQuery Location**: the data location of the lab's datasets (for example, `US`).
   - When you paste these credentials into the destination setup form, make sure you select **SaaS** as the deployment model.

The instructor provides a dbt service account that can read the landing data, run query jobs and create your transformation datasets. Use a separate dbt key if one is supplied. Attendees do not receive BigQuery Studio access; all destination data previews are instructor-led.

## Part 1: Accessing the provided resources

**Log into Fivetran**

1. Go to [fivetran.com/login](https://fivetran.com/login).
2. If you just signed up for the first time, use the username and password from your
   sign-up. If you already have a Fivetran account, use those credentials.
3. Switch to the account you've been added to using the account drop-down menu.

**BigQuery access for this lab**

Use the provided service-account JSON in Fivetran and your local dbt profile. No Google Cloud Console or BigQuery Studio sign-in is required. The instructor demonstrates the destination data preview in Part 8.

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
11. Wait for the setup tests to complete. When they pass, you'll see all connection tests marked successful (including **BigQuery Connection** and **BigQuery User Permissions**).
12. Click **View Destination** (or **Continue**) to proceed.

> **Dataset naming:** BigQuery uses `project.dataset.table`; Fivetran destination schemas become datasets. Replace `<dataset_prefix>` with the **BigQuery Dataset Prefix** from your credentials, including its trailing underscore (for example, `bqp001_`). Use your own name as well, such as `bqp001_jane_doe_postgres`. Keep the exact dataset names, including case, throughout the lab.

> **Automatic Platform Connection:** If Fivetran creates a connection named `fivetran_metadata` for your new destination, open **Connections**, verify it belongs to **your** destination, then use **Setup → Delete connection** before creating any other connections. The shared project needs uniquely named datasets. This removes the connection; do not delete shared datasets in BigQuery.

## Part 3: Setting up the Fivetran Platform Connector

Before adding the Platform Connection, sync at least one ordinary connection into your destination. Use the [BigQuery Technical Foundations worksheet, Part 3](../technical_foundations/bigquery_technical_foundations_hands_on_lab.md#part-3-creating-a-connection) with the instructor-provided PostgreSQL credentials, selecting the destination you just created. This gives the destination a connection for the Platform Connection to monitor.

1. Go to the **Connections** tab and click **Add connection**.
2. Search for **Fivetran Platform** and click **Set up**.
3. Select the destination with your name.
4. For the **Destination schema** field, use `<dataset_prefix><firstname>_<lastname>_fivetran_log`.
5. Disable **Account Level Connection**.
6. Click **Save & Test** and wait for the setup tests to complete.
7. Click **Continue**.
8. Select **I'll do this later** when prompted to use pre-built models, then click
   **Continue**.
9. Select **Start syncing all my data now** and click **Start initial sync**.
10. You'll be taken to the Connector Status page where you can track sync history.

> **Why the extra sync?** To avoid issues with the Incremental MAR table, we perform an
> incremental sync and then a re-sync. On a typical account this isn't needed; because
> this is an internal account with no MAR tracking, we do this to avoid errors in the
> dbt run.

11. **Perform an incremental sync:** click the **Sync** button in the top right.
12. **Perform a re-sync:** go to the **Settings** tab and click **Re-sync all data**.

> **Checkpoint:** The Fivetran Platform Connector has finished syncing into the `<dataset_prefix><firstname>_<lastname>_fivetran_log` schema in your BigQuery destination.

## Part 4: Setting up the dbt project

1. Open your system terminal.
2. Create a directory called `dbt_hands_on_lab` and navigate into it:

   ```bash
   mkdir dbt_hands_on_lab
   cd dbt_hands_on_lab
   ```

3. Create a Python virtual environment:

   ```bash
   # macOS / Linux
   python3 -m venv dbt_venv

   # Windows (PowerShell)
   py -m venv dbt_venv
   ```

4. Activate the virtual environment:

   ```bash
   # macOS / Linux
   source dbt_venv/bin/activate

   # Windows (PowerShell)
   dbt_venv\Scripts\Activate.ps1
   ```

5. Install the dbt BigQuery adapter (this also installs dbt-core):

   ```bash
   # macOS / Linux
   pip3 install dbt-bigquery

   # Windows (PowerShell)
   pip install dbt-bigquery
   ```

6. Verify your dbt installation:

   ```bash
   dbt --version
   ```

7. Save the complete **BigQuery Service Account JSON** to a file for dbt. Use the dbt-specific service-account JSON if your instructor provides one separately.

   1. Make sure you're still in the `dbt_hands_on_lab` directory.
   2. Create an empty file named `bigquery-service-account.json` and open it in a text editor:

      ```bash
      # macOS / Linux
      touch bigquery-service-account.json
      open -e bigquery-service-account.json      # macOS (TextEdit); on Linux use your editor of choice

      # Windows (PowerShell)
      New-Item bigquery-service-account.json
      notepad bigquery-service-account.json
      ```

   3. Paste the entire service-account JSON object from the 1Password link, including
      the opening `{` and closing `}`, then save
      and close the editor.
   4. Restrict the file so only your user can read it:

      ```bash
      # macOS / Linux
      chmod 600 bigquery-service-account.json

      # Windows (PowerShell) — remove inherited permissions, then grant only your user
      icacls bigquery-service-account.json /inheritance:r /grant:r "$env:USERNAME:R"
      ```

   5. Note the full path to the file — you'll enter it in the next step:

      ```bash
      # macOS / Linux
      echo "$(pwd)/bigquery-service-account.json"

      # Windows (PowerShell)
      (Resolve-Path bigquery-service-account.json).Path
      ```

   > **Keep the key outside the dbt project.** Leave `bigquery-service-account.json` in
   > `dbt_hands_on_lab`, not inside the `dbt_hol` project you create next. The `dbt_hol`
   > folder becomes a GitHub repository later in this lab, and the key must never be
   > committed.

8. Create a new dbt project. This walks you through prompts that set up the connection
   between your local dbt deployment and BigQuery.

   ```bash
   dbt init dbt_hol
   ```

   > **NOTE:** If you hit errors during `dbt init`, run it via the full venv path:
   > ```bash
   > # macOS / Linux
   > ./dbt_venv/bin/dbt init dbt_hol
   > # Windows (PowerShell)
   > .\dbt_venv\Scripts\dbt.exe init dbt_hol
   > ```

   Respond to the prompts using your credentials:

   - **Database** — choose **bigquery** (use its displayed menu number).
   - **Authentication method** — **service-account**.
   - **Keyfile** — the full path to `bigquery-service-account.json`.
   - **Project** — the provided **BigQuery Project ID**.
   - **Dataset** — `<dataset_prefix><firstname>_<lastname>_transformations`, including the assigned dataset prefix.
   - **Threads** — `4`.
   - **Location** — the provided **BigQuery Location**; it must match the landing datasets.

   Verify the generated `dbt_hol` profile in `~/.dbt/profiles.yml` (Windows: `$HOME/.dbt/profiles.yml`). The `dev` output should look like this after substituting your values. Use an absolute path; on Windows use forward slashes in the YAML path.

   ```yaml
   dbt_hol:
     target: dev
     outputs:
       dev:
         type: bigquery
         method: service-account
         project: <bigquery_project_id>
         dataset: <dataset_prefix><firstname>_<lastname>_transformations
         keyfile: /absolute/path/to/bigquery-service-account.json
         location: <bigquery_location>
         threads: 4
   ```

9. Navigate into the project and confirm it initialized correctly:

   ```bash
   cd dbt_hol
   dbt debug
   ```

   A successful run ends with **All checks passed!**

> **Checkpoint:** Your local `dbt_hol` project is initialized, connected to BigQuery, and `dbt debug` reports all checks passed.

## Part 5: Creating a GitHub repository

1. Go to [github.com](https://github.com). Sign in, or create an account if you don't
   have one.
2. Click **New** to create a new repository (on a new account, click **Create
   repository**).
3. Fill out the form:
   - **Owner** — yourself
   - **Repository name** — your dbt project name, `dbt_hol`
   - **Description** — optional
   - Leave everything else at default. **Do not** initialize with a README, .gitignore,
     or license.
4. Click **Create repository**.
5. Set up authentication at
   [github.com/settings/tokens](https://github.com/settings/tokens):
   - Select **Tokens (classic)** and click **Generate new token → Generate new token
     (classic)**.
   - Re-authenticate if prompted.
   - Name the token and set an expiration date.
   - Select the scopes you need. If unsure, start with everything shown for this lab.
   - Click **Generate token** and save it to a temporary location — you'll need it when
     you push for the first time.
6. Back in your repository, make sure **HTTPS** is selected and copy the first block of
   quick-setup commands. Paste and run them in your terminal (inside `dbt_hol`).
7. For the remaining two commands, you must modify the `git remote add origin` line to
   include your token. Paste them into a text editor and prepend your token before
   `github.com`. The format is:

   ```
   git remote add origin https://<your_token>@github.com/<your_user>/dbt_hol.git
   ```

   For example:

   ```
   git remote add origin https://ghp_xxxxxxxx@github.com/ahernadbt/dbt_hol.git
   ```

8. Run that updated command, then push:

   ```bash
   git push -u origin main
   ```

> **Checkpoint:** Your `dbt_hol` project is pushed to your new GitHub repository and visible on GitHub.

## Part 6: Adding and configuring a dbt package

1. Create a `packages.yml` file in the root of your dbt project:

   ```bash
   # macOS / Linux
   touch packages.yml
   open .

   # Windows (PowerShell)
   New-Item packages.yml
   ii .
   ```

2. Open `packages.yml` in your text editor.
3. Go to
   [hub.getdbt.com/fivetran/fivetran_log/latest/](https://hub.getdbt.com/fivetran/fivetran_log/latest/).
4. Copy the code under **Installation** and paste it into your `packages.yml`. It looks
   like this (use the latest version shown on the hub page):

   ```yaml
   packages:
     - package: fivetran/fivetran_log
       version: [">=2.0.0", "<3.0.0"]
   ```

5. Install the packages listed in `packages.yml` and their dependencies:

   ```bash
   dbt deps
   ```

6. Open `dbt_project.yml` in your text editor and add the following `vars` block near
   the end of the file, then save (replace `<firstname>_<lastname>`):

   ```yaml
   vars:
     fivetran_platform_database: <bigquery_project_id>
     fivetran_platform_schema: <dataset_prefix><firstname>_<lastname>_fivetran_log
     fivetran_platform_using_destination_membership: false
     fivetran_platform_using_user: false
     fivetran_platform_using_incremental_mar: false
   ```

7. Run the model locally to test before pushing to Git:

   ```bash
   dbt run --select fivetran_log
   ```

   > It's possible to see an error about the `incremental_mar` table because it doesn't
   > exist — this is expected on an internal Fivetran account with no MAR. That's normal
   > here.

8. Once it succeeds, commit and push your changes:

   ```bash
   git add .
   git commit -m "added vars to dbt_project.yml and added packages.yml"
   git push origin main
   ```

> **Checkpoint:** The `fivetran_log` package is installed and configured, `dbt run --select fivetran_log` succeeds locally, and your changes are pushed to GitHub.

## Part 7: Creating a transformation using Integrated scheduling

1. In the Fivetran dashboard, go to the **Transformations** tab.
2. Under **Orchestrate your custom data models**, click **Connect project for dbt Core**.
3. Select the destination with your name.
4. Copy the **Public key**.
5. In your GitHub repository, go to **Settings → Deploy keys → Add deploy key**.
6. Set the **Title** to `dbt_hol`, paste the key into the **Key** field, and click
   **Add key**. Fivetran can now pull from this repository.
7. Back in GitHub, click **Code** and copy the **SSH** URL for your repository.
8. In Fivetran, paste the URL into the **Repository URL** field.
9. Leave the **Connection Method** at its default.
10. Set the **Default Schema Name** to `<dataset_prefix><firstname>_<lastname>_dbt_prod`.
11. Leave the rest at defaults and click **Save & Test**.
12. When you set up the dbt Core version, you can enable **Automatically use latest patch
    version** so Fivetran keeps your dbt Core patch level current. Let the setup tests
    run, then click **Done**.
13. Click the **Transformations** tab and wait for your project to sync (about a minute).
14. Click **Add transformation** in the top-right corner.
15. Select the destination with your name and click **Create job for dbt Core**.
16. In the **Create job name** field, use `<dataset_prefix><firstname>_<lastname>_fivetran_log`. Use the
    same value for the **Enter job name** field under **Enter dbt command**.
17. For **Enter dbt command**, enter:

    ```bash
    dbt run --select fivetran_log
    ```

18. For **Select schedule**, choose **Integrated** and click **+ Add connection**.
19. Select the connection with your name and click **Save**.

    > The transformation status defaults to **Pending**. It runs whenever the connector
    > sync is kicked off on its schedule (default every 6 hours). You can also run it
    > manually by clicking into it and clicking **Run**. Click **View log** to see logs
    > similar to your local run.

> **Checkpoint:** Your dbt Core project is connected to Fivetran and the `fivetran_log` transformation job is created and running on an Integrated schedule.

## Part 8: Instructor-led preview of the transformed data

Attendees follow the instructor's shared screen. BigQuery Studio access is not required for attendees.

1. In Fivetran, confirm your transformation run succeeded and review its run log.
2. Follow the instructor's demonstration of the resulting dataset and `connection_daily_events` table. Compare the displayed dataset name with the output named in your transformation log.

**Instructor demonstration — perform these steps using your own authorized access:**

1. Open [BigQuery Studio](https://console.cloud.google.com/bigquery) and select the lab project.
2. Expand the attendee's dataset `<dataset_prefix><firstname>_<lastname>_dbt_prod_fivetran_platform`. Use the actual output dataset and table names from the successful transformation run if the package naming differs.
3. Select `connection_daily_events` and open **Preview** while sharing your screen.
4. To demonstrate a query, substitute the attendee's project and dataset in the SQL editor and run:

   ```sql
   select *
   from `<bigquery_project_id>.<dataset_prefix><firstname>_<lastname>_dbt_prod_fivetran_platform.connection_daily_events`
   limit 10;
   ```

> **Checkpoint:** Your transformation succeeded in Fivetran, and the instructor demonstrated its BigQuery output. If there are no event rows yet, review the Platform Connection's sync and transformation logs together.

---

## Troubleshooting notes

**`dbt debug` shows a git `[ERROR]` on Windows** — Git is either not installed or not on
your PATH. Install Git from [git-scm.com/download/win](https://git-scm.com/download/win)
(use default options, which add Git to PATH). Close and reopen your terminal, then rerun
`dbt debug`. If Git is installed but still not found, add it to PATH manually: System
Properties → Environment Variables → edit the `Path` System variable → add
`C:\Program Files\Git\bin` → save and reopen your terminal.

**Errors during `dbt init` or `dbt --version`** — point to the venv's interpreter/dbt
directly:

```bash
# macOS / Linux
./dbt_venv/bin/dbt --version
./dbt_venv/bin/dbt debug --profiles-dir ~/.dbt

# Windows (PowerShell)
.\dbt_venv\Scripts\dbt.exe --version
.\dbt_venv\Scripts\dbt.exe debug --profiles-dir $HOME\.dbt
```

**Wrong Python / dbt version** — dbt Core 1.11 requires Python 3.10 or newer. If you see
version errors, confirm your virtual environment uses Python 3.10–3.12:

```bash
# macOS / Linux
python3 --version

# Windows (PowerShell)
py --version
```

Confirm `dbt --version` lists the BigQuery adapter and a compatible dbt Core version. Adapter and Core version numbers need not match. Use the versions supported by the Fivetran dbt Core runtime selected in Part 7.

---

## What you did

You built a complete Fivetran + dbt Core transformation end to end. You created a BigQuery destination, synced the Fivetran Platform Connector into it, and stood up a local dbt project that you versioned in a GitHub repository. You then installed and configured the fivetran_log dbt package, connected the repository to Fivetran, and orchestrated the transformation on an Integrated schedule. Finally, you checked the Fivetran run log and reviewed the transformed data through an instructor-led BigQuery preview.

- [dbt BigQuery profile configuration](https://docs.getdbt.com/docs/local/connect-data-platform/bigquery-setup)

## BigQuery references

- [Fivetran BigQuery setup](https://fivetran.com/docs/destinations/bigquery/setup-guide)
