# Hands-on Lab: Transformations (Workstation)

*Updated September 2026. Verified against the current Fivetran Transformations documentation.*

Thank you for registering for our hands-on lab. This worksheet provides everything you need to prepare and complete the lab. In this lab you will build a complete Fivetran + dbt Core transformation end to end: stand up a Snowflake destination, sync the Fivetran Platform Connector, build and version a dbt project, install a Fivetran data model package, orchestrate it from Fivetran on an Integrated schedule, and view the transformed data in Snowflake. Please read through the requirements first. If you can't meet them, let us know and we'll rebook you on another workshop.

---

## Objective

Build an end-to-end Fivetran + dbt Core transformation: create a Snowflake **destination**, sync the **Fivetran Platform Connector**, version a **dbt project** in GitHub, install and configure the **fivetran_log** dbt package, and orchestrate it from Fivetran on an **Integrated** schedule to produce transformed tables you can view in Snowflake.

## Requirements

- Web browser (Chrome)
- A GitHub account

Everything else (Python, Git, VS Code, and the terminal) is pre-installed on the Linux
workstation provided for this lab.

## Housekeeping

- You will receive an invitation to the **TRANSFORMATIONS_HANDS_ON_LAB** Fivetran
  account prior to the lab.
- **Do not set up a new account or trial** — you'll receive an invite to a dedicated
  Fivetran account before the lab.
- Confirm you received the invite. It is sent to the email you used to register and
  originates from **notifications@fivetran.com**.
- You'll be provided a **Linux workstation** accessible in your web browser. The **Gateway URL**,
  **Guacamole Username**, and **Guacamole Password** are provided by your instructor via a
  1Password link.

## What you'll do in this lab

In this lab you will:

1. Create a dbt project
2. Create a GitHub repository for the dbt project
3. Connect the GitHub repository to Fivetran
4. Set up a Fivetran Platform Connector
5. Create a transformation for the Fivetran Platform Connector
6. View the transformed data in Snowflake

You have been provided with a Linux workstation accessible in your web browser. Throughout,
all the necessary commands are given for **Linux** and run on the workstation.

---

## Part 0: Credentials

**Snowflake** — Provided by your instructor via a 1Password link. The link includes a
**private key** (used for the Fivetran destination and the dbt profile) and a **password**
(used only to sign in to the Snowflake web UI).

## Part 1: Accessing the provided resources

**Log into Fivetran**

1. Go to [fivetran.com/login](https://fivetran.com/login).
2. If you just signed up for the first time, use the username and password from your
   sign-up. If you already have a Fivetran account, use those credentials.
3. Switch to the account you've been added to using the account drop-down menu.

**Log into Snowflake**

1. Click the provided link in **Part 0: Credentials**.
2. Enter the provided username and password and click **Sign in**.

   > **MFA enrollment:** Snowflake requires multi-factor authentication for password
   > sign-ins. The first time you sign in, Snowflake will prompt you to enroll and
   > offer several options. We recommend choosing **Passkey**. Complete the enrollment
   > before continuing.

## Part 2: Creating a destination

1. In Fivetran, go to the **Destinations** tab and click **Add destination**.
2. Search for **Snowflake**.
3. Name it `<firstname>_<lastname>_snowflake` (replace with your actual first and last
   name), then click **Add**.
4. Enter the provided credentials. Select **KEY PAIR** for the Auth field and paste in the
   provided private key.
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

1. Use the **Gateway URL**, **Guacamole Username**, and **Guacamole Password** from the
   1Password link to access your workstation.
2. Once you have logged in, click the **Terminal** icon on the bottom menu.

   > **Tip:** In the workstation terminal, paste with **CTRL + SHIFT + V**.

3. Create a directory called `dbt_hands_on_lab` and navigate into it:

   ```bash
   mkdir dbt_hands_on_lab
   cd dbt_hands_on_lab
   ```

4. Open the directory in VS Code:

   ```bash
   code .
   ```

   - If you get prompted to choose a password for a keyring, just hit **Cancel** (you may
     have to click it twice).
   - If you are prompted to log in to VS Code, close that pop-up. No login is required.

5. Navigate to the **"..."** menu in VS Code, find **Terminal**, click **New Terminal**,
   and select **Trust Folder & Continue**. The rest of the lab runs in this VS Code
   terminal.

6. Create a Python virtual environment:

   ```bash
   python3 -m venv dbt_venv
   ```

7. Activate the virtual environment:

   ```bash
   source dbt_venv/bin/activate
   ```

8. Install the dbt Snowflake adapter (this also installs dbt-core):

   ```bash
   pip3 install dbt-snowflake
   ```

9. Verify your dbt installation:

   ```bash
   dbt --version
   ```

10. Save the Snowflake private key to a file. dbt authenticates with a key pair and needs
    the key on disk. The 1Password item contains only the key text, so you'll create the
    file yourself.

    1. In the VS Code terminal, create an empty file named `snowflake_key.p8`:

       ```bash
       touch snowflake_key.p8
       ```

    2. In the VS Code Explorer, double-click `snowflake_key.p8` to open it.
    3. Paste the full private key from the 1Password link into the file, including the
       `-----BEGIN PRIVATE KEY-----` and `-----END PRIVATE KEY-----` lines, then save
       via **File > Save**.
    4. Back in the VS Code terminal, restrict the file so only your user can read it:

       ```bash
       chmod 600 snowflake_key.p8
       ```

    5. Note the full path to the file — you'll enter it in the next step:

       ```bash
       echo "$(pwd)/snowflake_key.p8"
       ```

    > **Keep the key outside the dbt project.** Leave `snowflake_key.p8` in
    > `dbt_hands_on_lab`, not inside the `dbt_hol` project you create next. The `dbt_hol`
    > folder becomes a GitHub repository later in this lab, and the key must never be
    > committed.

11. Create a new dbt project. This walks you through prompts that set up the connection
   between your local dbt deployment and Snowflake.

    ```bash
    dbt init dbt_hol
    ```

    > **NOTE:** If you hit errors during `dbt init`, run it via the full venv path:
    > ```bash
    > ./dbt_venv/bin/dbt init dbt_hol
    > ```

    Respond to the prompts. The credentials needed can be found in the 1Password link
    - **Which database would you like to use?** — select **snowflake** (enter `1`)
    - **account** — 
    - **user** — 
    - **authentication type** — select **keypair** (enter `2`)
    - **private_key_path** — the full path to the `snowflake_key.p8` file you created in
      the previous step
    - **private_key_passphrase** — leave blank and press Enter (the key has no passphrase)
    - **role**
    - **warehouse** 
    - **database** 
    - **schema** — `<firstname>_<lastname>_transformations`
    - **threads** — `8`

12. Navigate into the project and confirm it initialized correctly:

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
   quick-setup commands. Paste and run them in the VS Code terminal (inside `dbt_hol`).
7. For the remaining two commands, you must modify the `git remote add origin` line to
   include your token. Paste them into a new VS Code editor tab and prepend your token
   before `github.com`. The format is:

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
   touch packages.yml
   ```

2. In the VS Code Explorer, expand `dbt_hol` and double-click `packages.yml` to open it.
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

6. In the VS Code Explorer, open `dbt_project.yml` and add the following `vars` block near
   the end of the file, then save via **File > Save** (replace `<firstname>_<lastname>`):

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

1. Go to the Snowflake web UI and sign in with the provided username and password.
   Complete the MFA challenge using the Passkey you enrolled in Part 1.
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

**`dbt` command not found in a new terminal** — each new VS Code terminal starts without
the virtual environment. Re-activate it from the `dbt_hands_on_lab` directory:

```bash
source dbt_venv/bin/activate
```

**Errors during `dbt init` or `dbt --version`** — point to the venv's interpreter/dbt
directly:

```bash
./dbt_venv/bin/dbt --version
./dbt_venv/bin/dbt debug --profiles-dir ~/.dbt
```

**Wrong Python / dbt version** — dbt Core 1.11 requires Python 3.10 or newer. If you see
version errors, confirm your virtual environment uses Python 3.10–3.12:

```bash
python3 --version
```

A healthy environment looks like: dbt Core 1.11.x and the Snowflake adapter 1.11.x, both
reporting "Up to date."

---

## What you did

You built a complete Fivetran + dbt Core transformation end to end. You created a Snowflake destination, synced the Fivetran Platform Connector into it, and stood up a local dbt project that you versioned in a GitHub repository. You then installed and configured the fivetran_log dbt package, connected the repository to Fivetran, and orchestrated the transformation on an Integrated schedule. Finally, you confirmed the results by viewing the transformed data in Snowflake.
