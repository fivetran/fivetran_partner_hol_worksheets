# Hands-on Lab: Technical Foundations + AI (RAG Chatbot with Snowflake Cortex)

*Updated September 2026. Parts 1–3 are verified against the current Fivetran Technical Foundations documentation. Parts 4–7 are adapted from the Snowflake developer guide [Build a RAG-based, GenAI chatbot using Structured Data with Snowflake and Fivetran](https://www.snowflake.com/en/developers/guides/fivetran-vineyard-assistant-chatbot/#transform-the-wine-structured-dataset).*

Thank you for registering for our hands-on lab. This worksheet provides everything you need to prepare and to work through the lab. Please read through the requirements first. If you can't meet them, let us know and we'll rebook you on another workshop.

---

## Objective

Set up a **Destination** and a **Connection** in Fivetran, sync a structured wine-country dataset from PostgreSQL into Snowflake, then use **Snowflake Cortex** to transform that data into vector embeddings and build a **RAG-based (retrieval-augmented generation) chatbot** in **Streamlit in Snowflake** that answers questions using *your* Fivetran-delivered data.

## Requirements

- **Web browser**
  - Chrome or Firefox (latest version recommended)

---

## Housekeeping

**Invitation to the `TECH_FOUNDATIONS_HANDS_ON_LAB` Fivetran account**

- Do **not** set up a new account or trial. You will receive an invite to a dedicated Fivetran account prior to the lab.
- Confirm that you have received the invite to the Fivetran account.
- The invite is sent to the email address you used to sign up for this lab and originates from `notifications@fivetran.com`.

**Snowflake data warehouse**

- Credentials provided by your instructor (see Part 0).
- You will need to log in to the **Snowsight** web UI for Parts 4–7, not just configure the Fivetran destination.

**PostgreSQL database**

- Credentials provided by your instructor (see Part 0).

---

## What you'll do in this lab

1. Create a destination in Fivetran
2. Create and sync a connection
3. Transform the wine dataset into LLM-readable text and vector embeddings using Snowflake Cortex
4. Build a chatbot as a Streamlit application in Snowflake
5. Run the chatbot and compare responses with and without RAG
6. Test the chatbot with simple and complex prompts

---

## Part 0: Credentials

### Snowflake

- Provided by your instructor via a 1Password link. The Fivetran destination uses **key pair authentication**, so the link includes a private key rather than a password.
   - When you paste these credentials into the destination setup form, make sure you select **SaaS** as the deployment model.
- The link also includes the **Snowsight login** (account URL, username, and password) you will use to run SQL and build the Streamlit app in Parts 4–7.
- Your Snowflake user is provisioned with the `SNOWFLAKE.CORTEX_USER` database role, which grants access to the Cortex functions (`EMBED_TEXT_1024`, `COMPLETE`, `COUNT_TOKENS`) used in this lab. There is nothing you need to do to enable it.

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

### Access Snowflake (Snowsight)

1. In a **new browser tab**, open the Snowflake account URL provided in **Part 0: Credentials**.
2. Enter the provided username and password and click **Sign in**.

   > **MFA enrollment:** Snowflake requires multi-factor authentication for password
   > sign-ins. The first time you sign in, Snowflake will prompt you to enroll and
   > offer several options. We recommend choosing **Passkey**. Complete the enrollment
   > before continuing.

**Checkpoint:** you are signed into the `TECH_FOUNDATIONS_HANDS_ON_LAB` Fivetran account and into the Snowflake account, each in its own browser tab. Keep both open — you will move to Snowflake in Part 4.

---

## Part 2: Creating a destination

1. In Fivetran, navigate to the **Destinations** tab and click **Add destination**.
2. Search for **Snowflake** and select it.
3. Give the destination the name `<firstname>_<lastname>_snowflake`, replacing `<firstname>` and `<lastname>` with your actual first and last names. Click **Add**.
4. Enter the provided Snowflake credentials.
5. For the authentication method, select **Key Pair** and paste in the provided private key.
6. In the **Deployment model** section, select **SaaS**.
7. Leave the remaining fields at their default values. Do not make any changes.
8. Click **Save & Test**.
9. Wait for the setup tests to complete. When they pass, you'll see all connection tests marked successful.
10. Click **View Destination** (or **Continue**) to proceed.

> **Note:** Every destination you create automatically includes the free **Fivetran Platform Connection** (schema `fivetran_metadata`), which loads metadata about your account, connections, and usage.

---

## Part 3: Creating a connection

> In current Fivetran terminology, a **connector** is the reusable source type (for example, PostgreSQL), and a **connection** is the configured instance you set up from it. You can create many connections from the same connector.

1. Navigate to the **Connections** tab and, in the top-right corner, click **Add connection**.
2. Search for **PostgreSQL**, hover over the connector tile, and click **Set up**.
3. Select the destination you created in Part 2 (make sure you don't use someone else's destination), then click **Select**.
4. The value you enter in the **Destination schema prefix** field becomes the name of your connection. Use the format `<firstname>_<lastname>_postgres`, replacing `<firstname>` and `<lastname>` with your actual first and last names.
5. Populate the setup form with the provided PostgreSQL credentials.
6. For **Authentication Method**, select **Connect with username and password**.
7. Leave **Connection Method** at its default value, **Connect directly**.
8. For **Update Method**, select **Query-Based**.
   - *Context:* PostgreSQL's older XMIN and Fivetran Teleport Sync methods have been sunset and replaced by **Query-Based** change data capture. The other available method is **Logical replication** (using the `pgoutput` plugin). Existing Teleport/XMIN connections keep working.
9. For destination naming, select **Fivetran naming** (the default, which standardizes schema, table, and column names). The alternative, **Source naming**, preserves original UTF-8 names.
10. Click **Save & Test**.
11. When prompted, confirm the TLS certificate by selecting the certificate and clicking **Confirm**.
12. Wait for the setup tests to complete. When they pass, you'll see all connection tests marked successful.
13. Click **Continue**.
14. Fivetran now fetches all tables, schemas, and columns for the database.
15. The `fivetran_hol_agriculture` user has access to the `agriculture` schema. Make sure the `agriculture` schema and all of its tables are selected, then click **Save & Continue** to proceed.
    - The table you will use for the rest of this lab is `california_wine_country_visits`. Confirm it is included.
16. For handling schema changes, select **Allow all**, then click **Continue** (or **Save**).
17. Click **Start Initial Sync**. Wait for the initial (historical) sync to finish.
18. The sync will finish in about 1–2 minutes. A successful sync shows the connection status as **Active** with the synced tables and row counts.

> **Checkpoint — sync schedule and sync modes:**
> - By default, connections sync on a **fixed interval of every 6 hours**. You can change this on the connection's **Settings** tab (options range from 1 minute up to 24 hours; a **Cron** schedule is also available on Enterprise/Business Critical plans). There is no change data capture happening behind the scenes for this lab, so leave it at 6 hours.
> - Fivetran supports two **sync modes**: **Soft delete** (the default — deleted source rows are marked with `_fivetran_deleted = TRUE` rather than removed) and **History mode** (SCD Type 2, which tracks every version of a row using `_fivetran_start`, `_fivetran_end`, and `_fivetran_active`).

> **What just happened:** No allocating resources. No development. No code. No column mapping. No pre-building schemas or tables in the destination. A fully automated, production data pipeline in a few steps. In Snowflake, your data now lives in the schema `<firstname>_<lastname>_postgres_agriculture` (your destination schema prefix plus the source schema name).

---

## Part 4: Transforming the wine structured dataset

Now that Fivetran has landed the structured dataset into tables in Snowflake, it's time to convert that data into a format an LLM can read. LLMs don't work well with columnar data, so first you will transform each row into a single human-readable text chunk. Then you will transform each chunk into a **vector embedding**. Snowflake offers a managed vector search service (Cortex Search), but you'll do this manually in this lab so you understand every step.

**You will work in the Snowflake Snowsight UI for the rest of the lab.** Switch to the Snowflake browser tab you signed into in Part 1.

> **Note:** The Cortex functions used in this lab require the `SNOWFLAKE.CORTEX_USER` database role. Your lab user already has it, so no setup is needed.

1. **Review your data in Snowflake.** In the left navigation, select **Data** > **Databases**. Expand the database used by your Fivetran destination, then your schema (`<firstname>_<lastname>_postgres_agriculture`), and select the `CALIFORNIA_WINE_COUNTRY_VISITS` table. Click through the **Columns** and **Data Preview** tabs to get familiar with the data. This is the table you will transform into RAG context for the chatbot.
2. **Create a new SQL worksheet.** Select **Projects** > **Worksheets**, then click the **+** in the upper-right corner and choose **SQL Worksheet**.
3. **Set the worksheet context.** In the worksheet's left panel, expand your database and locate the schema Fivetran created. Click the **…** (ellipsis) next to the schema and select **Set worksheet context**. The database and schema shown in the context box at the top of the worksheet should now be yours. This means you don't need to fully qualify table names in your SQL.
4. **Paste the transformation SQL.** Copy the SQL below into the worksheet.

    ```sql
    /** Create each winery and vineyard review as a single field vs multiple fields **/
    CREATE or REPLACE TABLE vineyard_data_single_string AS
        SELECT WINERY_OR_VINEYARD, CONCAT(' The winery name is ', IFNULL(WINERY_OR_VINEYARD, ' Name is not known')
        , ' and resides in the California wine region of ', IFNULL(CA_WINE_REGION, 'unknown'), '.'
        , ' The AVA Appellation is the ', IFNULL(AVA_APPELLATION_SUB_APPELLATION, 'unknown'), '.'
        , ' The website associated with the winery is ', IFNULL(WEBSITE, 'unknown'), '.'
        , ' The price range is ', IFNULL(PRICE_RANGE, 'unknown'), '.'
        , ' Tasting Room Hours: ', IFNULL(TASTING_ROOM_HOURS, 'unknown'), '.'
        , ' The reservation requirement is: ', IFNULL(RESERVATION_REQUIRED, 'unknown'), '.'
        , ' The Winery Description is: ', IFNULL(WINERY_DESCRIPTION, 'unknown'), ''
        , ' The Primary Varietals this winery offers is ', IFNULL(PRIMARY_VARIETALS, 'unknown'), '.'
        , ' Thoughts on the Tasting Room Experience: ', IFNULL(TASTING_ROOM_EXPERIENCE, 'unknown'), '.'
        , ' Amenities: ', IFNULL(AMENITIES, 'unknown'), '.'
        , ' Awards and Accolades: ', IFNULL(AWARDS_AND_ACCOLADES, 'unknown'), '.'
        , ' Distance Travel Time considerations: ', IFNULL(DISTANCE_AND_TRAVEL_TIME, 'unknown'), '.'
        , ' User Rating: ', IFNULL(USER_RATING, 'unknown'), '.'
        , ' The Secondary Varietals for this winery: ', IFNULL(SECONDARY_VARIETALS, 'unknown'), '.'
        , ' Wine Styles: ', IFNULL(WINE_STYLES, 'unknown'), '.'
        , ' Events and Activities: ', IFNULL(EVENTS_AND_ACTIVITIES, 'unknown'), '.'
        , ' Sustainability Practices: ', IFNULL(SUSTAINABILITY_PRACTICES, 'unknown'), '.'
        , ' Social Media Channels: ', IFNULL(SOCIAL_MEDIA, 'unknown'), ''
        , ' The address is ', IFNULL(ADDRESS, 'unknown'), ', '
        , IFNULL(CITY, 'unknown'), ', '
        , IFNULL(STATE, 'unknown'), ', '
        , IFNULL(cast(ZIP as varchar(10)), 'unknown'), '.'
        , ' The Phone Number is ', IFNULL(PHONE, 'unknown'), '.'
        , ' The Winemaker is ', IFNULL(WINEMAKER, 'unknown'), '.'
        , ' Did Kelly Kohlleffel recommend this winery?: ', IFNULL(KELLY_KOHLLEFFEL_RECOMMENDED, 'unknown'), ''
    ) AS winery_information FROM california_wine_country_visits;

    /** Create the vector table from the wine review single field table **/
    CREATE or REPLACE TABLE vineyard_data_vectors AS
        SELECT winery_or_vineyard, winery_information,
        snowflake.cortex.EMBED_TEXT_1024('snowflake-arctic-embed-l-v2.0', winery_information) as WINERY_EMBEDDING
        FROM vineyard_data_single_string;
    ```

5. **Run the SQL.** Highlight all of the SQL and click the **Run** button (▶) in the upper-right corner. Both statements should complete successfully.
6. **Preview the results.** Click the refresh icon in the left panel. You will see two new tables:
   - `VINEYARD_DATA_SINGLE_STRING` — one row per winery, with all of its attributes concatenated into a single `WINERY_INFORMATION` text column.
   - `VINEYARD_DATA_VECTORS` — the same rows plus a `WINERY_EMBEDDING` column holding a 1024-dimension vector generated by the `snowflake-arctic-embed-l-v2.0` embedding model.

   Select either table and click the preview icon to view the transformations.

> **Checkpoint — what you just built:** The first statement turns each structured row into an unstructured "chunk" of natural language. The second statement calls the Cortex `EMBED_TEXT_1024` function to convert each chunk into a vector. In Part 5, the chatbot will embed the user's question with the same model and use `VECTOR_COSINE_SIMILARITY` to find the most relevant wineries to include as context for the LLM.

That's it for transforming the data. Now you're ready to build the Streamlit app.

---

## Part 5: Building the chatbot as a Streamlit application

Streamlit in Snowflake makes creating and sharing data applications easy. You will build the chatbot entirely inside Snowsight, with no local setup.

1. In the left navigation, select **Projects** > **Streamlit**.
2. Click **+ Streamlit App** in the upper-right corner.
3. Enter a name for your chat app (for example, `<firstname>_<lastname>_wine_assistant`).
   - **Very important:** For the app location, choose the **database and schema containing your data** (the schema Fivetran created, where `VINEYARD_DATA_VECTORS` lives). Select a warehouse when prompted.
   - Click **Create**.
4. **Get familiar with the editor and remove the default code.**
   - The **upper-right** area contains application controls. The vertical three dots (**⋮**) open settings such as changing the warehouse. The main features here are **Run** and **Edit**. You won't see **Edit** right now because a new app opens in edit mode. The next time you open this app it will be in run mode, and **Edit** will appear.
   - The **bottom-left** area has three toggles: the left navigation panel, the code panel, and the running application panel. Try turning each on and off. When editing, it's easiest to hide the left nav and the app panel so the code editor has the full screen.
   - Once you're comfortable, make sure the code panel is visible, click into the code, select all, and delete it. This is just placeholder code for a new app.
5. **Paste the chatbot code.** Copy the Python code below into the empty editor.

    > **Note:** No additional packages are needed. You do not need to use the **Packages** menu at the top of the editor.

    ```python
    #
    # Fivetran Snowflake Cortex Lab
    # Build a California Wine Assistant Chatbot
    #

    import streamlit as st
    from snowflake.snowpark.context import get_active_session
    import pandas as pd
    import time

    # Change this list as needed to add/remove model capabilities.
    MODELS = [
        "llama3.2-3b",
        "claude-3-5-sonnet",
        "mistral-large2",
        "llama3.1-8b",
        "llama3.1-405b",
        "llama3.1-70b",
        "mistral-7b",
        "jamba-1.5-large",
        "mixtral-8x7b",
        "reka-flash",
        "gemma-7b"
    ]

    # Change this value to control the number of tokens you allow the user to change to control RAG context. In
    # this context for the data used, 1 chunk would be approximately 200-400 tokens.  So a limit is placed here
    # so that the LLM does not abort if the context is too large.
    CHUNK_NUMBER = [4,6,8,10,12,14,16]

    def build_layout():
        #
        # Builds the layout for the app side and main panels and return the question from the dynamic text_input control.
        #

        # Setup the state variables.
        # Resets text input ID to enable it to be cleared since currently there is no native clear.
        if 'reset_key' not in st.session_state:
            st.session_state.reset_key = 0
        # Holds the list of responses so the user can see changes while selecting other models and settings.
        if 'conversation_state' not in st.session_state:
            st.session_state.conversation_state = []

        # Build the layout.
        #
        # Note:  Do not alter the manner in which the objects are laid out.  Streamlit requires this order because of references.
        #
        st.set_page_config(layout="wide")
        st.title(":wine_glass: California Wine Country Visit Assistant :wine_glass:")
        st.write("""I'm an interactive California Wine Country Visit Assistant. A bit about me...I'm a RAG-based, Gen AI app **built
          with and powered by Fivetran, Snowflake, Streamlit, and Cortex** and I use a custom, structured dataset!""")
        st.caption("""Let me help plan your trip to California wine country. Using the dataset you just moved into the Snowflake Data
          Cloud with Fivetran, I'll assist you with winery and vineyard information and provide visit recommendations from numerous
          models available in Snowflake Cortex (including Claude 3.5 Sonnet). You can even pick the model you want to use or try out
          all the models. The dataset includes over **700 wineries and vineyards** across all CA wine-producing regions including the
          North Coast, Central Coast, Central Valley, South Coast and various AVAs sub-AVAs. Let's get started!""")
        user_question_placeholder = "Message your personal CA Wine Country Visit Assistant..."
        st.sidebar.selectbox("Select a Snowflake Cortex model:", MODELS, key="model_name", index=3)
        st.sidebar.checkbox('Use your Fivetran dataset as context?', key="dataset_context", help="""This turns on RAG where the
        data replicated by Fivetran and curated in Snowflake will be used to add to the context of the LLM prompt.""")
        if st.button('Reset conversation', key='reset_conversation_button'):
            st.session_state.conversation_state = []
            st.session_state.reset_key += 1
            st.rerun()
        processing_placeholder = st.empty()
        question = st.text_input("", placeholder=user_question_placeholder, key=f"text_input_{st.session_state.reset_key}",
                                 label_visibility="collapsed")
        if st.session_state.dataset_context:
            st.caption("""Please note that :green[**_I am_**] using your Fivetran dataset as context. All models are very
              creative and can make mistakes. Consider checking important information before heading out to wine country.""")
        else:
            st.caption("""Please note that :red[**_I am NOT_**] using your Fivetran dataset as context. All models are very
              creative and can make mistakes. Consider checking important information before heading out to wine country.""")
        with st.sidebar.expander("Advanced Options"):
            st.selectbox("Select number of context chunks:", CHUNK_NUMBER, key="num_retrieved_chunks", help="""Adjust based on the
            expected number of records/chunks of your data to be sent with the prompt before Cortext calls the LLM.""", index=1)
        st.sidebar.caption("""I use **Snowflake Cortex** which provides instant access to industry-leading large language models (LLMs),
          including Claude, Llama, and Snowflake Arctic that have been trained by researchers at companies like Anthropic, Meta, Mistral, Google, Reka, and Snowflake.\n\nCortex
          also offers models that Snowflake has fine-tuned for specific use cases. Since these LLMs are fully hosted and managed by
          Snowflake, using them requires no setup. My data stays within Snowflake, giving me the performance, scalability, and governance
          you expect.""")
        for _ in range(6):
            st.sidebar.write("")
        url = 'https://i.imgur.com/9lS8Y34.png'
        col1, col2, col3 = st.sidebar.columns([1,2,1])
        with col2:
            st.image(url, width=150)
        caption_col1, caption_col2, caption_col3 = st.sidebar.columns([0.22,2,0.005])
        with caption_col2:
            st.caption("Fivetran, Snowflake, Streamlit, & Cortex")

        return question

    def build_prompt (question):
        #
        # Format the prompt based on if the user chooses to use the RAG option or not.
        #

        # Build the RAG prompt if the user chooses.  Defaulting the similarity to 0 -> 1 for better matching.
        chunks_used = []
        if st.session_state.dataset_context:
            # Get the RAG records.
            context_cmd = f"""
              with context_cte as
              (select winery_or_vineyard, winery_information as winery_chunk, vector_cosine_similarity(winery_embedding,
                    snowflake.cortex.embed_text_1024('snowflake-arctic-embed-l-v2.0', ?)) as v_sim
              from vineyard_data_vectors
              having v_sim > 0
              order by v_sim desc
              limit ?)
              select winery_or_vineyard, winery_chunk from context_cte
              """
            chunk_limit = st.session_state.num_retrieved_chunks
            context_df = session.sql(context_cmd, params=[question, chunk_limit]).to_pandas()
            context_len = len(context_df) -1
            # Add the vineyard names to a list to be displayed later.
            chunks_used = context_df['WINERY_OR_VINEYARD'].tolist()
            # Build the additional prompt context using the wine dataset.
            rag_context = ""
            for i in range (0, context_len):
                rag_context += context_df.loc[i, 'WINERY_CHUNK']
            rag_context = rag_context.replace("'", "''")
            # Construct the prompt.
            new_prompt = f"""
              Act as a California winery visit expert for visitors to California wine country who want an incredible visit and
              tasting experience. You are a personal visit assistant named Snowflake CA Wine Country Visit Assistant. Provide
              the most accurate information on California wineries based only on the context provided. Only provide information
              if there is an exact match below.  Do not go outside the context provided.
              Context: {rag_context}
              Question: {question}
              Answer:
              """
        else:
            # Construct the generic version of the prompt without RAG to only go against what the LLM was trained.
            new_prompt = f"""
              Act as a California winery visit expert for visitors to California wine country who want an incredible visit and
              tasting experience. You are a personal visit assistant named Snowflake CA Wine Country Visit Assistant. Provide
              the most accurate information on California wineries.
              Question: {question}
              Answer:
              """

        return new_prompt, chunks_used

    def get_model_token_count(prompt_or_response) -> int:
        #
        # Calculate and return the token count for the model and prompt or response.
        #
        token_count = 0
        try:
            token_cmd = f"""select SNOWFLAKE.CORTEX.COUNT_TOKENS(?, ?) as token_count;"""
            tc_data = session.sql(token_cmd, params=[st.session_state.model_name, prompt_or_response]).collect()
            token_count = tc_data[0][0]
        except Exception:
            # Negative value just denoting that tokens could not be counted for some reason.
            token_count = -9999

        return token_count

    def calc_times(start_time, first_token_time, end_time, token_count):
        #
        # Calculate the times for the execution steps.
        #

        # Calculate the correct durations
        time_to_first_token = first_token_time - start_time  # Time to the first token
        total_duration = end_time - start_time  # Total time to generate all tokens
        time_for_remaining_tokens = total_duration - time_to_first_token  # Time for the remaining tokens

        # Calculate tokens per second rate
        tokens_per_second = token_count / total_duration if total_duration > 0 else 1

        # Ensure that time to first token is realistically non-zero
        if time_to_first_token < 0.01:  # Adjust the threshold as needed
            time_to_first_token = total_duration / 2  # A rough estimate if it's too small

        return time_to_first_token, time_for_remaining_tokens, tokens_per_second

    def run_prompt(question):
        #
        # Run the prompt against Cortex.
        #
        formatted_prompt, chunks_used = build_prompt (question)
        token_count = get_model_token_count(formatted_prompt)
        start_time = time.time()
        cortex_cmd = f"""
                 select SNOWFLAKE.CORTEX.COMPLETE(?,?) as response
               """
        sql_resp = session.sql(cortex_cmd, params=[st.session_state.model_name, formatted_prompt])
        first_token_time = time.time()
        answer_df = sql_resp.collect()
        end_time = time.time()
        time_to_first_token, time_for_remaining_tokens, tokens_per_second = calc_times(start_time, first_token_time, end_time, token_count)

        return answer_df, time_to_first_token, time_for_remaining_tokens, tokens_per_second, int(token_count), chunks_used

    def main():
        #
        # Controls the flow of the app.
        #
        question = build_layout()
        if question:
            with st.spinner("Thinking..."):
                try:
                    # Run the prompt.
                    token_count = 0
                    data, time_to_first_token, time_for_remaining_tokens, tokens_per_second, token_count, chunks_used = run_prompt(question)
                    response = data[0][0]
                    # Add the response token count to the token total so we get a better prediction of the costs.
                    if response:
                        token_count += get_model_token_count(response)
                        # Conditionally append the token count line based on the checkbox
                        rag_delim = ", "
                        st.session_state.conversation_state.append(
                            (f":information_source: RAG Chunks/Records Used:",
                             f"""<span style='color:#808080;'> {(rag_delim.join([str(ele) for ele in chunks_used])) if chunks_used else 'none'}
                             </span><br/><br/>""")
                        )
                        st.session_state.conversation_state.append(
                            (f":1234: Token Count for '{st.session_state.model_name}':",
                             f"""<span style='color:#808080;'>{token_count} tokens • {tokens_per_second:.2f} tokens/s •
                             {time_to_first_token:.2f}s to first token + {time_for_remaining_tokens:.2f}s.</span>""")
                        )
                        # Append the new results.
                        st.session_state.conversation_state.append((f"CA Wine Country Visit Assistant ({st.session_state.model_name}):", response))
                        st.session_state.conversation_state.append(("You:", question))
                except Exception as e:
                    st.warning(f"An error occurred while processing your question: {e}")

            # Display the results in a stacked format.
            if st.session_state.conversation_state:
                for i in reversed(range(len(st.session_state.conversation_state))):
                    label, message = st.session_state.conversation_state[i]
                    if 'Token Count' in label or 'RAG Chunks' in label:
                        # Display the token count in a specific format
                        st.markdown(f"**{label}** {message}", unsafe_allow_html=True)
                    elif i % 2 == 0:
                        st.write(f":wine_glass:**{label}** {message}")
                    else:
                        st.write(f":question:**{label}** {message}")

    if __name__ == "__main__":
        #
        # App startup method.
        #
        session = get_active_session()

        main()
    ```

6. **Understand the code before you run it.** Here's how it fits together:

   - **Imports and constants.** The top of the file imports the Streamlit, Snowpark, pandas, and time packages. The `MODELS` list populates the model drop-down in the sidebar, and `CHUNK_NUMBER` populates the "number of context chunks" drop-down. Both are at the top so they're easy to change.
     > **Note:** The models listed are the ones that were available in Snowflake Cortex when the source guide was written. Model availability varies by Snowflake region and changes over time. If a model returns an error, pick another from the list, or edit `MODELS` to match what's available in your account (see the [Cortex LLM function docs](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions)).
   - **Chunks and RAG.** The chunk number is how many winery records (chunks) will be retrieved from your vector table and inserted into the prompt sent to the LLM. Simple prompts about a few wineries need only a few chunks; complex itinerary prompts need more. If you start seeing hallucinations, or data you know is in your dataset comes back as "unknown", increase the chunk count. Each model has a token limit, so the values in `CHUNK_NUMBER` are capped to stay within safe bounds.
   - `build_layout` renders the main panel (where you type your prompt) and the sidebar (model, RAG toggle, chunk count). Streamlit renders objects top-to-bottom like HTML, so the order matters. This function returns the user's question.
   - `build_prompt` builds the assistant's persona and either the RAG or non-RAG prompt, depending on whether the "Use your Fivetran dataset as context?" checkbox is checked. In RAG mode, it embeds the question with the same `snowflake-arctic-embed-l-v2.0` model used in Part 4, ranks wineries by `VECTOR_COSINE_SIMILARITY`, and pastes the top N chunks into the prompt as context.
   - `get_model_token_count` calls Cortex `COUNT_TOKENS` to estimate what Snowflake will charge for the prompt (and the response). See [Cortex cost considerations](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#cost-considerations).
   - `calc_times` computes timing stats so you can benchmark different models.
   - `run_prompt` is the controller: it formats the prompt, calls Cortex `COMPLETE`, and captures timings.
   - `main` is the entry point. It runs the prompt and displays results in reverse order so your most recent response is at the top.

---

## Part 6: Running the chatbot

1. Click the **Run** button in the upper-right corner to run the application.
2. **Get to know the interface.**

   **Content area (main panel):**
   - **Prompt input textbox.** Type your prompt and press Enter/Return to execute it.
   - **Reset conversation** clears the prompt and response history but leaves the sidebar settings alone, so you can start a new prompt with the same settings.
   - Next to the **Run** button is a status indicator. It spins while a prompt is running. If you're unsure whether your prompt is executing, check whether the status is spinning.

   **Side panel:**
   - Changing any sidebar setting automatically re-runs the current prompt, so you can compare results across settings.
   - **Model drop-down.** Try different models to see how each responds and which fits your use case best.
   - **Use your Fivetran dataset as context?** This checkbox enables RAG. The caption under the prompt changes to tell you whether RAG is on. Try a prompt with it checked, then uncheck it, to see how RAG lets the chatbot answer questions about *your* data. Use at least one **control record** (see Part 7) so you can confirm RAG is actually being used.
   - **Advanced Options > number of context chunks.** Sets how many records are added to the LLM context. This is the core of RAG. Remember each model has a token limit; the values in the list are safe for every model in the drop-down.

3. **Review the response.** The newest response bubbles to the top, so scroll up if needed. Each response includes two extra lines:
   - **Token Count** for the prompt plus response, with timings. This helps you understand the efficiency and cost of the model run.
   - **RAG Chunks/Records Used** lists the winery/vineyard names that were added to the context sent to the LLM. If no RAG records were sent, you'll see `none`. Only the names are shown here; the full `WINERY_INFORMATION` text was sent to the LLM.

> **Notes:**
> - The chatbot does **not** use previous responses to refine results. Every prompt is a fresh call to the LLM, possibly with a new set of RAG context.
> - If you edit the code (for example, adding a model to the list), click **Run** again so Streamlit picks up the change.

---

## Part 7: Testing the chatbot

Now that you know your way around the key features, it's time to put the California Wine Country Visit Assistant to work.

The assistant is designed to:

- Tell you about wineries and vineyards in California
- Create a trip/travel itinerary for a California wine country visit

When **Use your Fivetran dataset as context** is checked, the assistant uses the dataset you moved into Snowflake. The dataset has 700+ wineries and vineyards across California, each with: winery name, CA wine region, AVA/appellation/sub-appellation, website, price range, tasting room hours, reservation requirement, winery description, primary and secondary varietals, tasting room experience, amenities, awards and accolades, distance/travel time considerations, user rating, wine styles, events and activities, sustainability practices, social media channels, address, city, state, zip, phone, winemaker, and whether Kelly Kohlleffel recommended it.

### Control records

The dataset includes **control records** (phantom wineries) that are guaranteed not to exist in any model's training data:

- Millman Estate
- Tony Kelly Pamont Vineyards
- Hrncir Family Cellars
- Kai Lee Family Cellars
- Kohlleffel Vineyards
- Picchetti Winery

Use these to confirm RAG is working: with RAG on, you'll see the winery name in the **RAG Chunks/Records Used** line. With RAG off, if the model returns details about these wineries, it hallucinated. Ideally you'll get something like "...could not find the vineyard...".

There are also unique details in real winery descriptions that aren't in any model's training data, for example:

- **Continuum Estate:** The owners also have an energetic vizsla dog that runs around the property.
- **Hirsch Vineyards:** Kelly Kohlleffel recommends this winery for its location on the extreme Sonoma Coast. You will need your mapping app to navigate here, but you'll find terrific views and world-class pinot noir and chardonnay.
- **Alpha Omega Winery:** This winery is highly recommended by Kelly Kohlleffel based on enjoying an afternoon glass of wine on the patio facing the fountains. Also, the AO Era is a must try as well.

**Remember the chunks!** The more wineries you ask about, the more chunks you need to pass to the LLM, or it will try to fill in the gaps on its own.

### Simple prompts (4–6 chunks)

If you want more winery names to try, browse the `CALIFORNIA_WINE_COUNTRY_VISITS` table.

- `Tell me about the following wineries: Kohlleffel, Millman, Hrncir, Caymus`
- `Tell me about the difference in wine styles between Hrncir Family Cellars and Peju.`

### More complex prompts (6–8 chunks)

- `Plan a trip to visit 3 wineries during a 1 day trip that are all based in the Sonoma coast. Be sure to include Kohlleffel Vineyards as one of the three wineries.`
- `Plan me a 2 day trip covering 4 wineries in Yountville and Sonoma and include local eateries. Be sure to include Chandon Estates on day 1 and Tony Kelly on day 2.`

### Very complex prompts (10–16 chunks)

- `Provide a winery visit itinerary to visit six wineries during a two day trip. I'd like to visit the Sonoma coast on day 1 and Yountville on day 2. Be sure to include Kohlleffel Vineyards as one of the six wineries on day 1. Provide driving times as well. Organize this into a two day trip. Provide a hotel recommendation for the evening of day one. Also let me know about other activities that you recommend on the Sonoma Coast and in Yountville such as hiking trails. Also provide a catchy name for this trip of no more than seven words. Take all of the information and organize it with the trip name at the top and all the information in a good printable format. Lastly, what else would you suggest to make this trip even better?`

- `Provide a winery visit itinerary to visit nine wineries during a three day trip. I'd like to visit the Sonoma coast on day one and Yountville on day two and St. Helena on day three. Be sure to include Kohlleffel Vineyards and Millman Estate as two of the wineries on day one. Ensure that Hrncir Family Cellars is included on day two. Provide driving times as well. Organize this into a three day trip. Provide a hotel recommendation for the evening of day one and a different hotel for the evening of day two. Also let me know about other activities that you recommend on the Sonoma Coast and in Yountville and in St. Helena such as hiking trails. Also provide a catchy name for this trip of no more than seven words. Take all of the information and organize it with the trip name at the top and all the information in a good printable format. Lastly, what else would you suggest to make this trip even better?`

- `Provide a winery visit itinerary to visit 9 wineries during a 3 day trip. I'd like to visit the Sonoma coast on day 1 and Yountville on day 2 and St. Helena on day 3. Be sure to include Kohlleffel Vineyards and Millman Estate as 2 of the wineries on day 1. Provide driving times as well to get to the first winery and driving times between wineries and other venues. Organize this into a 3 day trip. Provide a hotel recommendation for the evening of day 1 and a different hotel for the evening of day 2. Also let me know about other activities that you recommend on the Sonoma Coast and in Yountville and in St. Helena such as hiking trails. Also provide a catchy name for this trip of no more than seven words. Provide your estimate for what this trip will cost and show me the detail on how you estimated the cost. Take all of the information and organize it with the trip name at the top and all the information in a good printable format. What else would you suggest to make this trip even better?`

- `Provide a winery visit itinerary to visit 9 wineries during a 3 day trip. I'd like to visit the Sonoma Coast on day 1 and Yountville on day 2 and Howell Mountain on day 3. Be sure to include Kohlleffel Vineyards and Millman Estate as 2 of the wineries on day 1. Del Dotto as one of the wineries on Day 2. and Sumit Lake Vineyards as one of the wineries on Day 3. Provide addresses of the wineries. Provide driving times as well to get to the first winery and driving times between wineries and other venues. Organize this into a 3 day trip. Provide a hotel recommendation for the evening of day 1 and a different hotel for the evening of day 2. Also let me know about other activities that you recommend on the Sonoma Coast and in Yountville and in St. Helena such as hiking trails. Also provide a catchy name for this trip of no more than seven words. Provide your estimate for what this trip will cost and show me the detail on how you estimated the cost. Take all of the information and organize it with the trip name at the top and all the information in a good printable format. What else would you suggest to make this trip even better? What types of clothing should I bring if I am planning my trip for early June?`

> **Cortex Search note:** This lab performs RAG manually by embedding text, storing vectors, and running a cosine-similarity query yourself, so you can see every moving part. Snowflake's **Cortex Search** service can simplify this by managing the embedding, indexing, and retrieval for you. Review [Cortex Search](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-search/cortex-search-overview) to understand other RAG implementation options, including searching your data without invoking an LLM.

> **Chunking note:** You didn't need to write a chunking (text-splitting) function because this is a controlled dataset where no single concatenated record exceeds 2,000 tokens. With very large records, you would need to split each record into appropriately sized chunks before embedding.

---

## What you did

You've completed the Fivetran Technical Foundations + AI hands-on lab. You:

- Created a production-ready data pipeline from PostgreSQL to Snowflake with Fivetran in a few clicks
- Used Snowflake Cortex to convert a structured dataset into an unstructured, vectorized dataset
- Built a RAG-based chatbot as a Streamlit application in Snowflake
- Planned wine-country trips that include places found only in *your* data

This demonstrates how easily Fivetran's fully automated pipelines make structured datasets available for GenAI use cases in Snowflake, without worrying about data freshness.

## Resources

- Source guide: [Build a RAG-based, GenAI chatbot using Structured Data with Snowflake and Fivetran](https://www.snowflake.com/en/developers/guides/fivetran-vineyard-assistant-chatbot/)
- [Snowflake Cortex LLM functions](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions)
- [Streamlit in Snowflake](https://docs.snowflake.com/en/developer-guide/streamlit/about-streamlit)
- [Fivetran PostgreSQL connector docs](https://fivetran.com/docs/connectors/databases/postgresql)
- [Fivetran Snowflake destination docs](https://fivetran.com/docs/destinations/snowflake)
