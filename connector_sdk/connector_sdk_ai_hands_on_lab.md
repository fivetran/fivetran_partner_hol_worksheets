# Hands-on Lab: Connector SDK AI Plugin

*Updated September 2026. Verified against the current Fivetran Connector SDK documentation.*

Thank you for registering for our hands-on lab. This worksheet has everything you need to
prepare and follow along. Please read through the requirements first. If you can't meet
them, let us know and we'll rebook you on another workshop.

---

## Objective

Build and deploy a custom connector with the Fivetran Connector SDK using the Claude AI Plugin: set up a Python
environment, create a connector locally with the AI Plugin, test it locally, and then deploy it into a Fivetran environment

## Requirements

- A web browser (Chrome)

## Housekeeping

- You'll be invited to the **CONNECTOR_SDK_HANDS_ON_LAB** Fivetran account.
- **Do not** set up a new account or trial — you'll receive an invite to a dedicated
  Fivetran account before the lab.
- Confirm you received the invite. It's sent to the email you used to register and
  comes from `notifications@fivetran.com`.
    - If you did not receive one, you likely already have a Fivetran account. So you can proceed with logging in with your email and password.
- You'll be provided a **Linux workstation** accessible in your web browser. The **Gateway URL**,
  **Guacamole Username**, and **Guacamole Password** are provided by your instructor via a
  1Password link.

## What you'll do in this lab

1. Set up a Python virtual environment
2. Install the Connector SDK
3. Install the Claude AI Plugin
4. Use Claude Code to build a connector for the News API
5. Test and verify the connector locally using Claude Code
6. Deploy the connector into Fivetran

You have been provided with a Linux Workstation accessible in your web browser. Throughout, all the necessary commands are given for **Linux**.

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
4. In the **Deployment model** section, select **SaaS**.
5. For the **Table Type** select **Snowflake Native Tables**.
6. For the **Connection Method** select **Connect directly**.
7. Enter the provided Snowflake credentials from the 1Password link the instructor provided.
8. For the authentication method, select **KEY PAIR** and paste in the provided private key.
9. Click **Load Virtual Warehouses** and select **CONNECTOR_SDK_HOL_WAREHOUSE**.
10. Leave the remaining fields at their default values. Do not make any changes.
11. Click **Save & Test**.
12. Wait for the setup tests to complete. When they pass, you'll see all connection tests marked successful.
13. Click **View Destination** (or **Continue**) to proceed.

> **Note:** Every destination you create automatically includes the free **Fivetran Platform Connection** (schema `fivetran_metadata`), which loads metadata about your account, connections, and usage. You'll use it in Part 7.

✅ **Checkpoint:** You're successfully connected to the destination and your setup tests are GREEN.

---


## Part 3: Setting up the Python environment

1. Use the **Gateway URL**, **Guacamole Username**, and **Guacamole Password** from the 1Password link to access your workstation

2. Once you have logged in click on the **Terminal** icon on the bottom menu

3. Create a directory called `connector_sdk` and move into it:

   ```bash
   mkdir connector_sdk
   cd connector_sdk
   ```

4. Create a Python virtual environment:

   ```bash
   python3 -m venv sdk
   ```

5. Activate the virtual environment:

   ```bash
   source sdk/bin/activate
   ```

6. Install the Fivetran Connector SDK:

   ```bash
   pip3 install fivetran-connector-sdk
   ```

7. Confirm the install and see the available commands:

   ```bash
   fivetran version
   fivetran --help
   ```

   You should see the installed version and the commands `init`, `debug`, `deploy`,
   `package`, `reset`, and `version`.

8. Back in the Fivetran dashboard, click your **username > API Key**.

9. Click **Generate new API key**. If prompted, confirm — this invalidates any old key
   and generates a new one.

10. Copy the **Base64-encoded API key**.

11. Add the API key you copied as an environment variable using the command:
    - Make sure your replace **your_api_key_here** with the actual API key from Fivetran that you copied
    - `Use CTRL + SHIFT + V` to paste

    ```bash
    echo export FIVETRAN_API_KEY=your_api_key_here >> ~/.profile
    ```

12. Open your Connector SDK project in VS Code
    - If you get prompted to choose a password for a keyring, just hit cancel 
    - You may have to click cancel twice

    ```bash
    code .
    ```
13. If you are prompted to log in for VS Code, just close that pop up. NO LOGIN is actually required. 

14.  Navigate to the **"..."** menu in VS Code, then find **Terminal** and click **New Terminal** and select **Trust Folder & Continue**

15. To create the Connector SDK project scaffolding and install the Claude AI Plugin run the command:

    ```bash
    fivetran init
    ```

16. Enter **1** to select the Claude Code Plugin. `fivetran init` automatically detects which AI model providers are installed on your system

✅ **Checkpoint:** You're successfully added your API keys to the project. And installed the Claude AI Plugin

---

## Part 4: Creating and Deploying a Custom Connector

1. Go to [https://newsapi.org/](https://newsapi.org/) to get an API key.
   - Register with your email address.
   - Enter your first name, email, and a password.
   - Select "I am an individual," agree to the terms, and submit.
   - Copy the displayed API key.

2. Save the News API key as an environment variable using the command:
    - Make sure your replace **your_api_key_here** with the actual API key from Fivetran that you copied
    - `Use CTRL + SHIFT + V` to paste

    ```bash
    echo export NEWS_API_KEY=your_api_key_here >> ~/.profile
    ```

3. In the VS Code Explorer double click `configuration.json` to open it, and replace what's currently in there with the following and save via File > Save:

   ```json
   {
       "top_headlines_country": "us",
       "top_headlines_category": "technology"
   }
   ```

4. In the VS Code Terminal run the following command to start Claude Code:
    ```bash
    claude
    ```
    
    - Select **Yes, I trust this folder**
    - Disregard the "Welcome" pop-up in VS Code, we're going to be using Claude via the Terminal   
    - Press "Enter" to continue and bring up the Claude prompt window

6. Use `Shift + Tab` to cycle through the permissions mode until you see **auto mode on**, this will save us some time here so we don't have to constantly approve Claude's requests

7. Run the following to use the `build-connector` skill provided the Connector SDK AI Plugin along with the prompt
    - Claude will load the skill and do some thinking based on what we have in the `configuration.json` it may figure out that we're trying to build a connector for the News API

    ```
    /fivetran-connector-sdk:build-connector

    Build a connector for the News API. Documentation: https://newsapi.org/docs.
    Use the endpoints: Everything, Top Headlines, and Sources, to create 3 tables.
    Use a 7 day lookback from today to start pulling data for Everything and Top Headlines. 

    The configuration.json contains the filters I want to use for Everything and Top Headlines. Use the country filter for Sources.

    NewsAPI uses an API Key for authentication. It is loaded as an environment variable called NEWS_API_KEY in ~/.profile. Encrypt it and ensure it is included in the configuration.json as Fivetran will need it.
    
    Gather all the information required to build the connector and then test it. Once that is done, stop. I will verify the data myself. Do not deploy the connector into Fivetran until I give the approval to so.
    ```

8. Claude will perform the research needed and start to build out the code in `connector.py` and also run the `fivetran debug` command to test the connector. This can take a few minutes, so just wait.

9. Open up the **files** directory in the VS Code Explorer, find `warehouse.db`, right click it, and select **Copy Path**

10. Preview the data by navigating to the **Application Finder** in the bottom desktop menu, click to open it, search for **DBeaver** then double click to open it.
    - It will take a moment to load, but once loaded go through the onboarding wizard
        - Select the **Simple** view
        - Deselect/decline any other options when prompted

11. Create a **New Connection** in DBeaver, search for DuckDB, select it, and then paste in the copied path to `warehouse.db`
    - Click **Test Connection** and select **yes** when prompted to download the DuckDB drivers
    - Confirm and then click Finish

12. Click the drop down next to `warehouse.db` in the Connection Explorer and navigate down to `warehouse > tester`
    - You should see 3 tables: `everything`, `top_headlines`, and `sources`
    - Double click on any of those tables to preview the data

13. Once you are done confirming the data looks right, we can give Claude the "okay" to deploy into Fivetran with the following prompt:
    - Replace `firstname_lastname` with your actual first and last name
    - The destination you choose should match the name of the Destination you created in Part 2. 
        - The name below assumes you followed the naming convention in Part 2. 

    ```
    Approved. Deploy the connector to Fivetran.
    The FIVETRAN_API_KEY is an environment variable in ~/.profile.
    Connector Name: firstname_lastname_news_api.
    Destination Name: firstname_lastname_snowflake.
    Once deployed start the initial sync.
    ```
14. Navigate back to the Fivetran UI and refresh the page to find your connector. Ideally it should be running the historical sync, but if any errors occur check the UI and iterate with Claude to resolve them
---

## What you did

You've set up the SDK environment, built and tested a custom connector with Claude Code, inspected the
data in `warehouse.db`, deployed with Claude Code, and initialized the first sync. 

From here, explore the
[sample connectors](https://github.com/fivetran/fivetran_connector_sdk) for patterns
like authentication, pagination, and incremental syncs — and remember the **Save Me
Time** program can pair you with Fivetran Professional Services for your first
production connector.
