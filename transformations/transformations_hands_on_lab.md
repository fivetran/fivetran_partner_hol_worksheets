# Hands-on Lab: Transformations

*Updated July 2026. Verified against the current Fivetran Transformations documentation.*

Thank you for registering for our hands-on lab. This worksheet provides everything you need to prepare and complete the lab. In this lab you will build a complete Fivetran + dbt Core transformation end to end: stand up a Snowflake destination, sync the Fivetran Platform Connector, build and version a dbt project, install a Fivetran data model package, orchestrate it from Fivetran on an Integrated schedule, and view the transformed data in Snowflake. Please read through the requirements first. If you can't meet them, let us know and we'll rebook you on another workshop.

---

## Objective

Build an end-to-end Fivetran + dbt Core transformation: create a Snowflake **destination**, sync the **Fivetran Platform Connector**, version a **dbt project** in GitHub, install and configure the **fivetran_log** dbt package, and orchestrate it from Fivetran on an **Integrated** schedule to produce transformed tables you can view in Snowflake.

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
6. View the transformed data in Snowflake

---

## Part 0: Credentials

**Snowflake** — Provided by your instructor via a 1Password link.

## Part 1: Accessing the provided resources

**Log into Fivetran**

1. Go to [fivetran.com/login](https://fivetran.com/login).
2. If you just signed up for the first time, use the username and password from your
   sign-up. If you already have a Fivetran account, use those credentials.
3. Switch to the account you've been added to using the account drop-down menu.

**Log into Snowflake**

1. Click the provided link in **Part 0: Credentials**.
2. Enter the provided credentials and click **Sign in**.

## Part 2: Creating a destination

1. In Fivetran, go to the **Destinations** tab and click **Add destination**.
2. Search for **Snowflake**.
3. Name it `<firstname>_<lastname>_snowflake` (replace with your actual first and last
   name), then click **Add**.
4. Enter the provided credentials. Make sure you select **PASSWORD** for the Auth field.
5. Select **SaaS Deployment** for the deployment model.
6. Leave the remaining fields at their defaults and click **Save & Test**.
7. Wait for the setup tests to complete, then click **View Destination** to continue.

> **Checkpoint:** Your Snowflake destination named `<firstname>_<lastname>_snowflake` is created and its setup tests have passed.

## Part 3: Setting up the Fivetran Platform Connector

1. Go to the **Connections** tab and click **Add connection**.
2. Search for **Fivetran Platform** and click **Set up**.
3. Select the destination with your name.
4. For the **Destination schema** field, use `<firstname>_<lastname>_fivetran_log`.
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

> **Checkpoint:** The Fivetran Platform Connector has finished syncing into the `<firstname>_<lastname>_fivetran_log` schema in your Snowflake destination.

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

5. Install the dbt Snowflake adapter (this also installs dbt-core):

   ```bash
   # macOS / Linux
   pip3 install dbt-snowflake

   # Windows (PowerShell)
   pip install dbt-snowflake
   ```

6. Verify your dbt installation:

   ```bash
   dbt --version
   ```

7. Create a new dbt project. This walks you through prompts that set up the connection
   between your local dbt deployment and Snowflake.

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

   Respond to the prompts. The credentials needed can be found in the 1Password link
   - **Which database would you like to use?** — select **snowflake** (enter `1`)
   - **account** — 
   - **user** — 
   - **authentication type** — select **password** (enter `1`)
   - **password** — the password from the 1Password link 
      - it won't display as you paste, that's normal; press Enter once pasted
   - **role**
   - **warehouse** 
   - **database** 
   - **schema** — `<firstname>_<lastname>_transformations`
   - **threads** — `8`

8. Navigate into the project and confirm it initialized correctly:

   ```bash
   cd dbt_hol
   dbt debug
   ```

   A successful run ends with **All checks passed!**

> **Checkpoint:** Your local `dbt_hol` project is initialized, connected to Snowflake, and `dbt debug` reports all checks passed.

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
     fivetran_platform_schema: <firstname>_<lastname>_fivetran_log
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
10. Set the **Default Schema Name** to `<firstname>_<lastname>_dbt_prod`.
11. Leave the rest at defaults and click **Save & Test**.
12. When you set up the dbt Core version, you can enable **Automatically use latest patch
    version** so Fivetran keeps your dbt Core patch level current. Let the setup tests
    run, then click **Done**.
13. Click the **Transformations** tab and wait for your project to sync (about a minute).
14. Click **Add transformation** in the top-right corner.
15. Select the destination with your name and click **Create job for dbt Core**.
16. In the **Create job name** field, use `<firstname>_<lastname>_fivetran_log`. Use the
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

## Part 8: Viewing the transformed data in Snowflake

1. Go to the Snowflake web UI, sign in with the provided credentials, and click **Not
   Now** if prompted for MFA.
2. Click the **Data** tab.
3. Open **TRANSFORMATIONS_HOL_DATABASE** and search for your name (the first part of your
   connector name).
4. Click the schema with your name ending in `_DBT_PROD_FIVETRAN_PLATFORM`.
5. Expand the **Tables** dropdown.
6. Click the **CONNECTION_DAILY_EVENTS** table.
7. Click the **Data Preview** tab to view your transformed data.

> **Checkpoint:** You can see rows of transformed data in the `CONNECTION_DAILY_EVENTS` table in your `_DBT_PROD_FIVETRAN_PLATFORM` schema in Snowflake.

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

A healthy environment looks like: dbt Core 1.11.x and the Snowflake adapter 1.11.x, both
reporting "Up to date."

---

## What you did

You built a complete Fivetran + dbt Core transformation end to end. You created a Snowflake destination, synced the Fivetran Platform Connector into it, and stood up a local dbt project that you versioned in a GitHub repository. You then installed and configured the fivetran_log dbt package, connected the repository to Fivetran, and orchestrated the transformation on an Integrated schedule. Finally, you confirmed the results by viewing the transformed data in Snowflake.
