# Hands-on Lab: Hybrid Deployment

*Updated July 2026. Verified against the current Fivetran Hybrid Deployment documentation.*

Thank you for registering for our hands-on lab. This worksheet has everything you need to prepare. Please read through the requirements first. If you can't meet them, let us know and we'll rebook you on another workshop.

---

## Objective

Create a **Hybrid Deployment agent** in Fivetran and install it on your own **VM**, stand up a Hybrid **destination** and **connector**, then run an initial sync where data never leaves your network — before confirming the agent install directory and editing `config.json` to enable per-job logging.

## Requirements

- **An SSH client**
  - macOS / Linux: the built-in **Terminal** (`ssh`)
  - Windows: **Windows Terminal / PowerShell** (built-in OpenSSH `ssh`) or **PuTTY**
- **A web browser** — Chrome or Firefox
- **Basic Linux command familiarity** (all VM commands are Linux, regardless of your
  laptop's OS)
- Outbound access on Port 22 from your machine / corporate VPN

## Housekeeping

- You'll be invited to the **HYBRID_DEPLOYMENT_HANDS_ON_LAB** Fivetran account. 
   - **Do not create a new account or trial** — you'll receive an invite before the lab.
- The invite goes to the email you registered with, from `notifications@fivetran.com`.
  Confirm you received it.
- You'll be provided: 
   - An assigned **VM** for the Hybrid Deployment agent
   - A **Snowflake** data warehouse
   - A **PostgreSQL** database.

## What you'll do in this lab

1. Create a Hybrid Deployment agent in Fivetran
2. Install the agent on the provided VM
3. Create a destination in Fivetran that uses your agent
4. Create and sync a connector
5. Confirm the install directory you learned about in the training
6. Configure a custom option via `config.json` and see it take effect

> Module 3 of the training covers the install directory structure in detail (`conf/`, `data/`,
> `data/_samples/`, `logs/`, `tmp/`, `stats/`, `hdagent.sh`, `hd-debug.sh`). In this lab you'll
> confirm it on your own VM and then change one configuration parameter.

---

## Part 0: Credentials

**Hybrid Deployment VM**
- VM IP: provided via email
- SSH username / password: Provided by your instructor via a 1Password link.

**Snowflake**
- Select **Hybrid** as the deployment model when entering these credentials in the setup
  form.
- Credentials: Provided by your instructor via a 1Password link.

**PostgreSQL**
- Credentials: Provided by your instructor via a 1Password link.

---

## Part 1: Accessing the provided resources

### Access Fivetran (new users)
1. Log in at `https://fivetran.com/login` with the username/password you signed up with.
2. In the new-user flow: at **Tell us about yourself**, click **Skip**.
3. At **What software do you currently use**, select **Salesforce** and click **Next**
   (dummy selection — the choice doesn't matter here).
4. At **Get ready to set up your first connection**, click **Let's go**.
5. Close the **You're now on updated pricing** pop-up.

### Access Fivetran (existing users)
1. Log in with your existing Fivetran credentials (use **Forgot your password?** if needed).
2. Switch to the `HYBRID_DEPLOYMENT_HANDS_ON_LAB` account via the account drop-down in the top left.

### Access your Hybrid Deployment VM
Log into the VM over SSH with the SSH command and password you were assigned.
   - The command can be found in your welcome email
      - If you cannot find it, let your instructor know and they will provide it to you
      - DO NOT use someone else's SSH command as they are unique to you
   - The password can be found in the 1Password credentials link

**macOS / Linux (Terminal):**

```bash
ssh <username>@<vm-ip>
```

**Windows (PowerShell / Windows Terminal — built-in OpenSSH):**

```powershell
ssh <username>@<vm-ip>
```

**Windows (PuTTY):** enter `<vm-ip>` as the Host Name, keep port `22`, click **Open**,
then enter `<username>` and the password when prompted.

- When prompted `Are you sure you want to continue connecting?`, enter **yes**.
- Enter the provided password when prompted 
   - It will not appear or be visible when you paste it.
   - This is normal
   - Do not paste it more than once or press any other buttons besides ENTER
- On success you'll see the VM welcome screen.

> All commands from here run **on the VM** (Linux), so they're identical whether your
> laptop is macOS, Linux, or Windows. The only OS difference is the SSH client you used to
> connect above.

---

## Part 2: Creating and installing the agent

1. In Fivetran, go to the **Destinations** tab and click **Add destination**.
2. Search for **Snowflake**.
3. Name it `<firstname>_<lastname>_snowflake` (use your real first/last name), then click
   **Add**.
4. Enter the provided credentials. 
   - For **Auth**, select **Password** and paste the provided password.
5. In **Select deployment model**, choose **Hybrid Deployment**, then click
   **+ Configure a new agent**.
6. Agree to the terms.
7. Select the **Docker** deployment type.
8. Name the agent `<firstname>_<lastname>_agent`, then click **Generate agent token**.
9. Copy the **Install and start agent** command, then click **Save**.

   > The token is shown **once**. Copy the whole command now — you'll paste it into the VM.

10. Switch back to your VM terminal and run the command. It looks like this (your token
    will be filled in):

```bash
TOKEN="YOUR_TOKEN_HERE" RUNTIME=docker bash -c "$(curl -sL "https://raw.githubusercontent.com/fivetran/hybrid_deployment/main/install.sh")"
```

11. Confirm the container is running:

```bash
docker container ls -a
```

You should see the **controller** container running.

12. Follow the live agent logs with the following command:
   - Replace **container_id** with the actual ID of the container shown from the `docker container ls -a` command.

```bash
docker container logs container_id --follow
```

13. Back in Fivetran, under **The storage service you want to use**, select **Snowflake
    Internal Stage**.

14. Leave everything else as the default value

15. Click **Save & Test** and wait for the setup tests to pass.

16. Click **View destination** to continue.

> **Checkpoint:** Your agent's controller container is running on the VM and your Snowflake destination has passed its setup tests.

---

## Part 3: Creating a connector

1. Go to the **Connections** tab and click **Add connector**.
2. Search for **Google Cloud SQL for PostgreSQL** and click **Set up**.
3. Select the destination you created (make sure it's **yours**, not someone else's).
4. In **Destination schema prefix**, use `<firstname>_<lastname>_gcp_postgres` — this
   becomes your connection name.
5. Populate the setup form with the provided PostgreSQL credentials.
6. For **Authentication Method**, select **Connect with username and password**.
7. Under **Hybrid Deployment**, ensure **your** agent is selected.
8. Leave **Require TLS** unchecked.
9. For **Update Method**, select **Query-Based**.
10. For **Destination schema names**, select **Fivetran naming**.
11. Click **Save & Test** and wait for the setup tests to pass, then click **Continue**.
12. Fivetran fetches all tables, schemas, and columns. 
   - The `fivetran_hol_user` only has access to the **coffee_prices** table in the **agriculture** schema
   - Leave only `coffee_prices` selected and click **Save & Continue**.
13. For handling schema changes, select **Allow all**.
14. Click **Start Initial Sync**. While it runs, continue to Part 4.

> **Checkpoint:** Your PostgreSQL connector is created and its initial sync of the `coffee_prices` table completes successfully.

---

## Part 4: Confirm the install directory

In the training (Module 3) you learned what the installer creates under `$HOME/fivetran`.
Let's confirm it on your own VM. (For the full explanation of each directory, refer back to
the training — here we're just verifying it exists and peeking at the config file.)

1. Exit the live log view: press **Ctrl + C**.

2. Go to the install directory and list it:

```bash
cd ~/fivetran
ls
```

   You should see `conf`, `data`, `logs`, `tmp`, `stats`, `hdagent.sh`, and `hd-debug.sh`.

3. View the configuration file. 
   - By default it holds only the `token` needed to connect
   securely to Fivetran's cloud
   - And also the SE Linux configuration toggle, which is automatically set based on the VM settings

```bash
cat conf/config.json
```

4. Confirm the management script is present and check the agent's status:

```bash
./hdagent.sh -r docker status
```

5. Reminder from the training: 
  - `data/` is persistent storage used during processing (and its
  - `_samples/` sub-directory holds hashed sample files for active-row/MAR calculations — never delete anything in `_samples/`). 
  - `tmp/` is transient. 
  - `logs/` holds the agent logs. 
  - `stats/` is where the `hd-debug.sh` diagnostics tool writes its support bundle. 
  - `hdagent.sh` is used for running specific agent commands
  - `hd-debug.sh` is for running diagnostics for troubleshooting

6. The full parameter list lives in the **Hybrid Deployment Agent Configuration Parameters** docs.

> **Checkpoint:** The `~/fivetran` install directory exists with `conf`, `data`, `logs`, `tmp`, and `stats`, and you've viewed `conf/config.json`.

---

## Part 5: Modifying `config.json`

You'll add a configuration parameter that writes per-job log files to disk — useful when
debugging a specific sync.

1. Make sure your initial sync has completed in the dashboard (so we can safely restart the
   agent), then open `config.json` for editing:

```bash
vi ~/fivetran/conf/config.json
```

2. Press **i** to enter insert mode.
3. Add a comma at the end of the existing `token` line, then add the parameter below on a
   new line. This enables per-job log files:

```json
"save_job_logs_to_file": true
```

   Your file should look like:

```json
{
  "token": "YOUR_TOKEN_HERE",
  "se_linux_enabled": false,
  "save_job_logs_to_file": true
}
```

4. Press **Esc** to leave insert mode.
5. Type `:wq!` and press **Enter** to save and quit.
6. Restart the agent so the change takes effect:

```bash
cd ~/fivetran
./hdagent.sh -r docker stop
./hdagent.sh -r docker start
```

7. In the Fivetran dashboard, click **Sync Now** on your connector. Since there's no new
   data, it completes in a few seconds.
8. Back on the VM, look in the `logs` directory — per-job log files are now written to disk:
   - The **random_id** will be different for everyone
   - It's unique to your Hybrid Deployment agent

```bash
cd ~/fivetran/logs/random_id/jobs
ls
```

9. Open a job logfile with `cat` to see the verbose per-job detail (each is uniquely named):

```bash
cat <job-log-file>
```

> By default, `save_job_logs_to_file` is `false` and only the agent (controller) log is kept
> on disk. Turning it on gives you the per-job detail you'd reach for when a specific sync
> misbehaves. Other logging parameters — `save_controller_logs_to_file`, `log_retention_days`,
> and `log_clean_frequency_milliseconds` — are covered in Module 4.

> **Checkpoint:** After restarting the agent, per-job log files now appear under `~/fivetran/logs/<random_id>/jobs`.

---

## Part 6: (Optional) Collect a diagnostics bundle

If you were troubleshooting for real, this is how you'd package everything for Support.

1. From the install directory, run the diagnostics collector as your regular user:

```bash
cd ~/fivetran
./hd-debug.sh -r docker
```

2. Confirm the bundle was written to the `stats` directory:

```bash
ls stats
```

   You'll see an archive named `logs-<controller_id>.tar.gz` — that's the file you'd attach
   to a Fivetran support case. (Use `./hd-debug.sh -r docker -x env` to exclude environment
   variables from the bundle.)

---

## Part 7: Viewing the data in Snowflake

1. The instructor will walk you through the final view of the data in Snowflake

> **Checkpoint:** You can see the synced `coffee_prices` data in your Snowflake destination.

---

## What you did

You created a Hybrid Deployment agent, installed it on your own VM, stood up a Hybrid
destination and connector, ran a sync where **data never left the VM's network** (only
metadata and logs went to Fivetran), confirmed the install directory, enabled a custom
configuration parameter and watched it take effect, and generated a diagnostics bundle. This
is the same workflow you'll guide a customer through in a real Hybrid Deployment.