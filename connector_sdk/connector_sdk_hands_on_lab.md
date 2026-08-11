# Hands-on Lab: Connector SDK

*Updated July 2026. Verified against the current Fivetran Connector SDK documentation.*

Thank you for registering for our hands-on lab. This worksheet has everything you need to
prepare and follow along. Please read through the requirements first. If you can't meet
them, let us know and we'll rebook you on another workshop.

---

## Objective

Build and deploy a custom connector with the Fivetran Connector SDK: set up a Python
environment, test a sample connector locally with `fivetran debug`, deploy it with
`fivetran deploy`, then build a real connector for an external API from scratch.

## Requirements

- **Python 3.10–3.14** (we'll use **3.13**, the SDK default). Python 3.9 is no longer supported.
- A code editor (VS Code, Sublime, Notepad++, etc.)
    - We recommend VS Code for this lab
- A web browser (Chrome)

## Optional

- A SQL workbench (DBeaver)
- The DuckDB CLI

## Housekeeping

- You'll be invited to the **CONNECTOR_SDK_HANDS_ON_LAB** Fivetran account.
- **Do not** set up a new account or trial — you'll receive an invite to a dedicated
  Fivetran account before the lab.
- Confirm you received the invite. It's sent to the email you used to register and
  comes from `notifications@fivetran.com`.
    - If you did not receive one, you likely already have a Fivetran account. So you can proceed with logging in with your email and password.

## What you'll do in this lab

1. Set up a Python virtual environment
2. Install the Connector SDK
3. Test SDK functionality using a sample connector
4. Deploy a test connector to review the flow
5. Create and deploy a connector for a custom API

Throughout, commands are given for **macOS/Linux** and **Windows**. Run the one that
matches your system.

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

2. Using the account drop-down menu at the top, switch to the **`CONNECTOR_SDK_HANDS_ON_LAB`**
   account you were added to.

✅ **Checkpoint:** You're logged in and viewing the `CONNECTOR_SDK_HANDS_ON_LAB` account.


---

## Part 2: Creating a destination

1. In Fivetran, navigate to the **Destinations** tab and click **Add destination**.
2. Search for **Snowflake** and select it.
3. Give the destination the name `<firstname>_<lastname>_snowflake`, replacing `<firstname>` and `<lastname>` with your actual first and last names. Click **Add**.
4. Enter the provided Snowflake credentials from the 1Password link the instructor provided.
5. For the authentication method, select **PASSWORD** and paste in the provided password.
6. In the **Deployment model** section, select **SaaS**.
7. Leave the remaining fields at their default values. Do not make any changes.
8. Click **Save & Test**.
9. Wait for the setup tests to complete. When they pass, you'll see all connection tests marked successful.
10. Click **View Destination** (or **Continue**) to proceed.

> **Note:** Every destination you create automatically includes the free **Fivetran Platform Connection** (schema `fivetran_metadata`), which loads metadata about your account, connections, and usage. You'll use it in Part 7.

✅ **Checkpoint:** You're successfully connected to the destination and your setup tests are GREEN.

---


## Part 3: Setting up the Python environment

1. Open your system's terminal.

2. Create a directory called `connector_sdk` and move into it:

   ```bash
   mkdir connector_sdk
   cd connector_sdk
   ```

3. Create a Python virtual environment:

   **macOS/Linux**
   ```bash
   python3 -m venv sdk
   ```
   **Windows (PowerShell)**
   ```powershell
   py -m venv sdk
   ```

4. Activate the virtual environment:

   **macOS/Linux**
   ```bash
   source sdk/bin/activate
   ```
   **Windows (PowerShell)**
   ```powershell
   sdk\Scripts\Activate.ps1
   ```

5. Install the Fivetran Connector SDK:

   ```bash
   pip install fivetran-connector-sdk
   ```

6. Confirm the install and see the available commands:

   ```bash
   fivetran version
   fivetran --help
   ```

   You should see the installed version and the commands `init`, `debug`, `deploy`,
   `package`, `reset`, and `version`.

7. Back in the Fivetran dashboard, click your **username > API Key**.

8. Click **Generate new API key**. If prompted, confirm — this invalidates any old key
   and generates a new one.

9. Copy the **Base64-encoded API key**.

10. Open your Connector SDK project in VS Code
    ```bash
    code .
    ```
11. Navigate to **Terminal** and click **New Terminal**

12. Create a `test.env` file in the root of your project:

    **macOS/Linux**
    ```bash
    touch test.env
    ```
    **Windows (PowerShell)**
    ```powershell
    New-Item test.env
    ```
13. Open the `test.env` file in your code editor

14. Add the following to `test.env`, pasting in the API key you copied:

    ```
    FIVETRAN_API_KEY=<paste-your-base64-api-key-here>
    ```

15. Load the env file into your terminal session:

    **macOS/Linux**
    ```bash
    export $(grep -v '^#' test.env | xargs)
    ```
    **Windows (PowerShell)**
    ```powershell
    Get-Content test.env | Where-Object { $_ -notmatch '^#' -and $_ -match '=' } | ForEach-Object {
      $name, $value = $_ -split '=', 2
      Set-Item -Path "env:$name" -Value $value
    }
    ```

16. Confirm the key loaded:

    **macOS/Linux**
    ```bash
    echo $FIVETRAN_API_KEY
    ```
    **Windows (PowerShell)**
    ```powershell
    echo $env:FIVETRAN_API_KEY
    ```

> **Tip:** `fivetran init` can scaffold a whole project for you (including a runnable
> example and an AI-assistant `AGENTS.md`). We'll build the files by hand in this lab so
> you see each one, but `fivetran init` is the fastest way to start a real project.

✅ **Checkpoint:** You're successfully added your API keys to the project.

---

## Part 4: Creating a sample connector

1. Create the connector files:

   **macOS/Linux**
   ```bash
   touch connector.py configuration.json
   ```
   **Windows (PowerShell)**
   ```powershell
   New-Item connector.py; New-Item configuration.json
   ```

2. Open the Fivetran **Connector SDK Quickstart** examples on GitHub:
   [Hello Example](https://github.com/fivetran/connector_sdk/blob/main/examples/quickstart/hello/connector.py)

3. Copy the `connector.py` code from the example and paste it into your `connector.py`,
   then save.

4. Run the debug command to execute your code locally. The first run downloads the
   connector tester, so give it a moment:

   ```bash
   fivetran debug
   ```

   A successful run prints an operations summary (upserts, checkpoints) and finishes
   without errors.

5. List the generated files. You'll see a new `files` directory (and a `__pycache__`):

   **macOS/Linux**
   ```bash
   ls
   ls files
   ```
   **Windows (PowerShell)**
   ```powershell
   dir
   dir files
   ```

6. You'll see `state.json` and `warehouse.db` in `files`. Look at the saved state:

   **macOS/Linux**
   ```bash
   cat files/state.json
   ```
   **Windows (PowerShell)**
   ```powershell
   type files\state.json
   ```

7. Preview the extracted data in `warehouse.db` with DBeaver or DuckDB (if you don't have either, that's okay the instructor will demonstrate):
   - Open DBeaver and create a new connection.
   - Search for and select **DuckDB**.
   - Browse to and select `files/warehouse.db`, then open it.
   - Click **Finish** (you may be prompted to install the DuckDB driver).
   - Expand the `warehouse.db` object in the DBeaver navigator and preview the table.

> **Note:** DuckDB allows only one connection to the database file at a time. If you run
> `fivetran debug` while `warehouse.db` is open in DBeaver, you'll get a "Could not set
> lock on file" error. Close the DBeaver connection before re-running debug.

✅ **Checkpoint:** You have successfully created and tested your first connector.

---

## Part 5: Deploying a connector

Now that you've run the connector locally, push it into Fivetran with `fivetran deploy`.

1. In the command below, replace `firstname_lastname` with your actual first and last
   name. If you get an overwrite prompt, there's a name conflict — cancel and add a few
   random digits to the connection name.

   ```bash
   fivetran deploy \
     --destination {firstname_lastname}_destination \
     --connection {firstname_lastname}_hello \
     --configuration configuration.json
   ```
   - Replace {firstname_lastname} with your actual first and last names.

   > For automated (CI/CD) deploys, add `--non-interactive` so the command doesn't stop
   > to ask about overwriting. (The older `--force` / `-f` flag is deprecated.)

2. In the Fivetran dashboard, search for your connection using by your first and last name.

3. Click your connection and start the initial sync. It should take about 30 seconds.

4. Open the **Connector SDK Logs** tab to view the logs your connector generated. Click
   the most recent log to see its details.

5. Open the **Schema** tab to see the tables defined by your `connector.py`, including
   columns and primary keys. This is also where you can de-select or hash columns.

6. Open the **Usage** tab to see how many Monthly Active Rows your connector is using.
   Remember the first 14 days after the initial sync completes are a free-use period.

7. Open the **Setup** tab to test and edit the connection, adjust sync frequency,
   allow/deny schema changes, set delay thresholds, re-sync historical data, or delete
   the connection.

8. Click **Edit Connection**. Here you can change the Python version the connector runs
   on and, if you provided a `configuration.json`, edit configuration values in the UI.

✅ **Checkpoint:** You've successfully deployed your connector and started syncing it.

---

## Part 6: Creating and deploying a "real" connector

You've seen how to set up the environment and deploy a sample connector. Now build one
from scratch using **NewsAPI** as the source. NewsAPI lets you find articles and
breaking headlines from sources across the web through a JSON API.

1. Go to [https://newsapi.org/](https://newsapi.org/) to get an API key.
   - Register with your email address.
   - Enter your first name, email, and a password.
   - Select "I am an individual," agree to the terms, and submit.
   - Copy the displayed API key and save it — you'll use it in `configuration.json`.

2. Set up your development environment as in **Part 3** (new folder, virtual
   environment, SDK install).
   - Create a new working directory
   - Create a new python virtual environment
   - Install the Connector SDK
   - You're credentials should still be in your terminal session, but you can create a new file to drop your API Keys in

3. Create `configuration.json` and paste in the following, replacing the placeholder
   with your NewsAPI key:

   ```json
   {
       "api_key": "YOUR_NEWS_API_KEY_HERE",
       "pageSize": "5",
       "language": "en"
   }
   ```

4. Create `connector.py` and paste in the code skeleton below. Your job is to fill in each numbered step. Use the
   [Connector SDK technical reference](https://fivetran.com/docs/connector-sdk/technical-reference)
   to complete it.

   ```python
   import requests as rq
   import traceback
   import datetime

   # Step 1: Add the three required imports
   #   Connector, Operations as op, Logging as log


   def schema(configuration: dict):
       # Step 2: Return a list defining two tables:
       #   - top_headlines (primary key: url)
       #   - sources        (primary key: id)
       


   def update(configuration: dict, state: dict):
       # Step 3: Implement the update logic
       top_headlines_url = "https://newsapi.org/v2/top-headlines"
       sources_url = "https://newsapi.org/v2/top-headlines/sources"

       try:
           # Step 3.1: If state has a saved timestamp, start from it;
           #           otherwise look back 7 days for the first run.
           # Step 3.2: Define the end of the time range (now).
           # Step 3.3: Build the request headers using the api_key from configuration.
           # Step 3.4: Build the top_headlines params (from, to, page, language,
           #           sortBy, pageSize) — language and pageSize come from config.

           sync_top_headlines(top_headlines_url, headers, top_headlines_params, state)

           # Step 3.5: Build the sources params (language from config) and sync sources.
           sync_sources(sources_url, headers, sources_params)

           # Step 3.6: Build new_state (the "to" timestamp) and checkpoint it —
           #           call op.checkpoint(...) directly (no yield).
           log.info(f"state updated, new state: {repr(new_state)}")

       except Exception as e:
           raise RuntimeError(f"{e}\n{traceback.format_exc()}")


   def sync_top_headlines(base_url, headers, params, state):
       has_more_pages = True
       while has_more_pages:
           # Step 4: Get the API response via get_api_response().
           # Step 4.1: For each article, op.upsert(...) into "top_headlines".
           # Step 4.2: op.checkpoint(state) after each page.
           has_more_pages, params = pagination(params, response_page)


   def sync_sources(base_url, headers, params):
       # Step 5: Get the response and op.upsert(...) each source into "sources".
       


   def get_api_response(endpoint_path, headers, params):
       response = rq.get(endpoint_path, headers=headers, params=params)
       response.raise_for_status()
       return response.json()


   def pagination(params, response_page):
       has_more_pages = True
       current_page = int(params["page"])
       total_pages = divmod(int(response_page["totalResults"]), int(params["pageSize"]))[0] + 1
       increment = (current_page and total_pages and current_page < total_pages
                    and current_page * int(params["pageSize"]) < 100)
       if increment:
           params["page"] = current_page + 1
       else:
           has_more_pages = False
       return has_more_pages, params


   # Step 6: Create the Connector object from update and schema.
   ```

   You'll use the [NewsAPI docs](https://newsapi.org/docs/endpoints) to sync the **Top
   Headlines** and **Sources** endpoints. There are six numbered steps; you provide the
   schema and update logic and the two helper functions that pull the data.

5. When you think the code is complete, test it locally:

   ```bash
   fivetran debug --configuration configuration.json
   ```

   A successful run prints an operations summary. The number of upserts will vary based
   on when you run it, since headlines and sources change over time. In DBeaver, you
   should see two tables — `top_headlines` and `sources` — with data.

6. Once local testing succeeds, deploy it into Fivetran by following the steps in
   **Part 4** (use a connection name like `firstname_lastname_news`).
   - Prior to running the deploy command, remove the API Key from the configuration.json

7. In Fivetran, you'll navigate to your connector > **Settings** > **Edit Connection**

8. Add the API key to the **api_key** field and **Test Connection**
    - This is how you can add sensitive credentials or modify settings or credentials without having
    to go through code changes and a connector redeployment.

9. Sync your Connector by clicking **Start Initial Sync**

---

## Full code for Part 6

Try to write the code yourself first — the practice with `schema`, `update`, the
operations, and checkpointing is the point of the exercise, and so is navigating the
docs. That said, the complete, functional version is below. Use it to compare against
yours, or to understand what a working connector looks like. Your code may differ and
still be correct — there are many ways to do this in Python.

**Note the modern style:** operations are called **directly** (`op.upsert(...)`,
`op.checkpoint(...)`) — no `yield`. This is the current SDK convention.

```python
import requests as rq
import traceback
import datetime

from fivetran_connector_sdk import Connector
from fivetran_connector_sdk import Logging as log
from fivetran_connector_sdk import Operations as op


def schema(configuration: dict):
    return [
        {
            "table": "top_headlines",
            "primary_key": ["url"],
            "columns": {
                "source_id": "STRING",
                "source_name": "STRING",
                "published_at": "UTC_DATETIME",
                "author": "STRING",
                "title": "STRING",
                "description": "STRING",
                "url": "STRING",
                "content": "STRING",
            },
        },
        {
            "table": "sources",
            "primary_key": ["id"],
            "columns": {
                "id": "STRING",
                "name": "STRING",
                "description": "STRING",
                "url": "STRING",
                "category": "STRING",
                "language": "STRING",
                "country": "STRING",
            },
        },
    ]


def update(configuration: dict, state: dict):
    top_headlines_url = "https://newsapi.org/v2/top-headlines"
    sources_url = "https://newsapi.org/v2/top-headlines/sources"

    try:
        if "end_timestamp" in state:
            start_timestamp = state["end_timestamp"]
        else:
            start_timestamp = (datetime.datetime.now() - datetime.timedelta(days=7)).strftime(
                "%Y-%m-%dT%H:%M:%S"
            )

        end_timestamp = datetime.datetime.now().strftime("%Y-%m-%dT%H:%M:%S")

        headers = {
            "Authorization": "Bearer {}".format(configuration["api_key"]),
            "accept": "application/json",
        }

        top_headlines_params = {
            "from": start_timestamp,
            "to": end_timestamp,
            "page": 1,
            "language": configuration["language"],
            "sortBy": "publishedAt",
            "pageSize": configuration["pageSize"],
        }

        sync_top_headlines(top_headlines_url, headers, top_headlines_params, state)

        sources_params = {"language": configuration["language"]}
        sync_sources(sources_url, headers, sources_params)

        new_state = {"end_timestamp": end_timestamp}
        log.info(f"state updated, new state: {repr(new_state)}")
        op.checkpoint(state=new_state)

    except Exception as e:
        exception_message = str(e)
        stack_trace = traceback.format_exc()
        raise RuntimeError(f"Error Message: {exception_message}\nStack Trace:\n{stack_trace}")


### helper methods ###
def sync_top_headlines(base_url, headers, params, state):
    has_more_pages = True

    while has_more_pages:
        response_page = get_api_response(base_url, headers, params)
        log.info(str(response_page["totalResults"]) + " results")

        items = response_page.get("articles", [])
        if not items:
            break

        for item in items:
            op.upsert(
                table="top_headlines",
                data={
                    "source_id": item["source"]["id"],
                    "source_name": item["source"]["name"],
                    "published_at": item["publishedAt"],
                    "author": item["author"],
                    "title": item["title"],
                    "description": item["description"],
                    "url": item["url"],
                    "content": item["content"],
                },
            )

        op.checkpoint(state)
        has_more_pages, params = pagination(params, response_page)


def sync_sources(base_url, headers, params):
    response_page = get_api_response(base_url, headers, params)

    for item in response_page.get("sources", []):
        op.upsert(
            table="sources",
            data={
                "id": item["id"],
                "name": item["name"],
                "description": item["description"],
                "url": item["url"],
                "category": item["category"],
                "language": item["language"],
                "country": item["country"],
            },
        )


def get_api_response(endpoint_path, headers, params):
    response = rq.get(endpoint_path, headers=headers, params=params)
    response.raise_for_status()
    return response.json()


def pagination(params, response_page):
    has_more_pages = True

    current_page = int(params["page"])
    total_pages = divmod(int(response_page["totalResults"]), int(params["pageSize"]))[0] + 1

    increment_page_number = (
        current_page
        and total_pages
        and current_page < total_pages
        and current_page * int(params["pageSize"]) < 100
    )

    if increment_page_number:
        params["page"] = current_page + 1
    else:
        has_more_pages = False

    return has_more_pages, params


connector = Connector(update=update, schema=schema)
```

---

## What you did

You've set up the SDK, tested a connector locally with `fivetran debug`, inspected the
data in `warehouse.db`, deployed with `fivetran deploy`, and built a real connector for
a custom API. From here, explore the
[sample connectors](https://github.com/fivetran/fivetran_connector_sdk) for patterns
like authentication, pagination, and incremental syncs — and remember the **Save Me
Time** program can pair you with Fivetran Professional Services for your first
production connector.
