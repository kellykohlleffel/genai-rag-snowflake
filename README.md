# :wine_glass: Create a wine country travel assistant with Fivetran, Snowflake, Cortex, and Streamlit
## Scripts and code for the Fivetran + Snowflake RAG-based, Gen AI Hands on Lab (75 minutes)

This repo provides the high-level steps to create a RAG-based, Gen AI travel assistant using Fivetran and Snowflake (detailed instructions are in the lab guide provided by your lab instructor). The required transformation scripts and the required Streamlit code are both included. This repo is the "easy button" to copy/paste the transformations and the code. If you have any issues with copy/paste, you can download the code [here](https://github.com/kellykohlleffel/genai-rag-snowflake/archive/refs/heads/main.zip).

### STEP 0: Prerequisite - Access the Fivetran lab account

* For this hands-on workshop, we have provisioned you into a Fivetran Account specifically for this lab. Let’s access it now.

* Access the email account that you provided for this lab and look for an email from notifications@fivetran.com that was sent to you today - it will look like this:

![Join the Fivetran lab account](./images/join_the_fivetran_hol_account_email.png)

* If you **DO NOT** already have a Fivetran account, click on the Accept Invite link and you will be sent to the Fivetran sign-up page for the lab account. Note that Fivetran requires at least a 12-CHARACTER PASSWORD. Once you’ve entered your information, click Sign up.

* If you **DO** already have a Fivetran account, follow these steps:
    * Accept invite from email
    * **Switch Account** to **Fivetran_HoL** by clicking "Switch account" in the upper left of your Fivetran UI
    * Login to Fivetran_HoL
        * Select forgot password
        * Go to the email for password reset (from fivetran)
        * Set password
        * Login

### STEP 1: Create a Fivetran connector to Snowflake

* **Source**: Google Cloud PostgreSQL (G1 instance)
* **Fivetran Destination**: SNOWFLAKE_LLM_LAB_X
* **Destination Schema prefix (connector name)**: yourlastname 
* **Host**: 34.94.122.157 **(NOTE - see the lab guide or your email for credentials and additional host identifiers)**
```
34.94.122.157
```
* **Data source sync selections:**
    * **Schema**: agriculture
    * **Table**: california_wine_country_visits

### STEP 2: View the new dataset in Snowflake Snowsight

* **Snowflake Account**: **https://dma21732.snowflakecomputing.com** **(NOTE - see the lab guide or your email for credentials)**
* **Snowflake Database**: HOL_DATABASE
* **Schema**: yourlastname_agriculture 
* **Table**: california_wine_country_visits
* Click on **Data Preview** to take a look

### STEP 3: Transform the new structured dataset into a single string to simulate an unstructured document
* Open a New Worksheet in **Snowflake Snowsight** (left gray navigation under Projects)
* Make sure you set the worksheet context at the top: **HOL_DATABASE** and **yourlastname_agriculture** schema name
* Copy and paste these [**transformation scripts**](01-transformations.sql) in your Snowsight worksheet
* Highlight only the first transformation script in the editor and click run
* This will create a new winery_information table using the CONCAT function. Each multi-column record (winery or vineyard) will now be a single string (creates an "unstructured" document for each winery or vineyard)

```
/** Transformation #1 - Create the vineyard_data_single_string table using concat and prefixes for columns (creates an "unstructured" doc for each winery/vineyard)
/** Create each winery and vineyard review as a single field vs multiple fields **/
CREATE OR REPLACE TABLE vineyard_data_single_string AS 
    SELECT WINERY_OR_VINEYARD, CONCAT(
        'The winery name is ', IFNULL(WINERY_OR_VINEYARD, ' Name is not known'), '.',
        ' Wine region: ', IFNULL(CA_WINE_REGION, 'unknown'),
        ' The AVA Appellation is the ', IFNULL(AVA_APPELLATION_SUB_APPELLATION, 'unknown'), '.',
        ' The website associated with the winery is ', IFNULL(WEBSITE, 'unknown'), '.',
        ' The price range is ', IFNULL(PRICE_RANGE, 'unknown'), '.',
        ' Tasting Room Hours: ', IFNULL(TASTING_ROOM_HOURS, 'unknown'), '.',
        ' The reservation requirement is: ', IFNULL(RESERVATION_REQUIRED, 'unknown'), '.',
        ' Here is a complete description of the winery or vineyard: ', IFNULL(WINERY_DESCRIPTION, 'unknown'), '.',
        ' The primary varietal this winery offers is ', IFNULL(PRIMARY_VARIETALS, 'unknown'), '.',
        ' Thoughts on the Tasting Room Experience: ', IFNULL(TASTING_ROOM_EXPERIENCE, 'unknown'), '.',
        ' Amenities: ', IFNULL(AMENITIES, 'unknown'), '.',
        ' Awards and Accolades: ', IFNULL(AWARDS_AND_ACCOLADES, 'unknown'), '.',
        ' Distance Travel Time considerations: ', IFNULL(DISTANCE_AND_TRAVEL_TIME, 'unknown'), '.',
        ' User Rating: ', IFNULL(USER_RATING, 'unknown'), '.',
        ' The secondary varietal for this winery is: ', IFNULL(SECONDARY_VARIETALS, 'unknown'), '.',
        ' Wine Styles for this winery are: ', IFNULL(WINE_STYLES, 'unknown'), '.',
        ' Events and Activities: ', IFNULL(EVENTS_AND_ACTIVITIES, 'unknown'), '.',
        ' Sustainability Practices: ', IFNULL(SUSTAINABILITY_PRACTICES, 'unknown'), '.',
        ' Social Media Channels: ', IFNULL(SOCIAL_MEDIA, 'unknown'), '.',
        ' The address is ', 
            IFNULL(ADDRESS, 'unknown'), ', ',
            IFNULL(CITY, 'unknown'), ', ',
            IFNULL(STATE, 'unknown'), ', ',
            IFNULL(ZIP, 'unknown'), '.',
        ' The Phone Number is ', IFNULL(PHONE, 'unknown'), '.',
        ' Winemaker: ', IFNULL(WINEMAKER, 'unknown'),
        ' Did Kelly Kohlleffel recommend this winery?: ', IFNULL(KELLY_KOHLLEFFEL_RECOMMENDED, 'unknown')
    ) AS winery_information
    FROM california_wine_country_visits;

/** Transformation #2 - Using the Snowflake Cortex embed_text_768 LLM function, this transformation creates embeddings from the newly created vineyard_data_single_string table and creates a vector table called winery_embedding.
/** Create the vector table from the wine review single field table **/
      CREATE or REPLACE TABLE vineyard_data_vectors AS 
            SELECT winery_or_vineyard, winery_information, 
            snowflake.cortex.EMBED_TEXT_768('e5-base-v2', winery_information) as WINERY_EMBEDDING 
            FROM vineyard_data_single_string;

/** Select a control record to see the LLM-friendly "text" document table and the embeddings table **/
    SELECT *
    FROM vineyard_data_vectors
    WHERE winery_information LIKE '%winery name is Kohlleffel Vineyards%';

```

### STEP 4: Create the embeddings and the vector table from the winery_information single string table
* Highlight only the second transformation script in your Snowflake Snowsight worksheet and click run
* This will create your embeddings and a vector table that will be referenced later by Cortex LLM functions and your Streamlit application

```
/** Transformation #2 - Using the Snowflake Cortex embed_text_768 LLM function, this transformation creates embeddings from the newly created vineyard_data_single_string table and creates a vector table called winery_embedding.
/** Create the vector table from the wine review single field table **/
      CREATE or REPLACE TABLE vineyard_data_vectors AS 
            SELECT winery_or_vineyard, winery_information, 
            snowflake.cortex.EMBED_TEXT_768('e5-base-v2', winery_information) as WINERY_EMBEDDING 
            FROM vineyard_data_single_string;
```

### STEP 5: Run a SELECT statement to check out the LLM-friendly "text" document table and embeddings table
* Highlight only the third script **SELECT * FROM vineyard_data_vectors WHERE winery_information LIKE '%winery name is Kohlleffel Vineyards%';** in your Snowflake Snowsight worksheet and click run
* This will show you the complete results of the 2 transformations that you just ran

```
/** Select a control record to see the LLM-friendly "text" document table and the embeddings table **/
    SELECT *
    FROM vineyard_data_vectors
    WHERE winery_information LIKE '%winery name is Kohlleffel Vineyards%';
```

### STEP 6: Create the a Streamlit app and build a Visit Assistant Chatbot
* Open a New Streamlit application in Snowflake Snowflake (left gray navigation under Projects)
* Make sure you set the Streamlit app context: **HOL_DATABASE** and **yourlastname_agriculture** schema name
* Highlight the "hello world" Streamlit code and delete it
* Click Run to clear the preview pane
* Copy and paste the [**Streamlit code**](02-streamlit-code.py) in the Streamlit editor

```
#
# Fivetran Snowflake Cortex Streamlit Lab
# Build a California Wine Country Travel Assistant Chatbot
#


import streamlit as st
from snowflake.snowpark.context import get_active_session
import pandas as pd
import time


# Change this list as needed to add/remove model capabilities.
MODELS = [
    "reka-flash",
    "llama3.2-3b",
    "llama3.1-8b",
    "jamba-1.5-large",
    "llama3.1-70b",
    "llama3.1-405b",
    "mistral-7b",
    "mixtral-8x7b",
    "mistral-large2",
    "snowflake-arctic",
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
      models available in Snowflake Cortex (including Snowflake Arctic). You can even pick the model you want to use or try out 
      all the models. The dataset includes over **700 wineries and vineyards** across all CA wine-producing regions including the 
      North Coast, Central Coast, Central Valley, South Coast and various AVAs sub-AVAs. Let's get started!""")
    user_question_placeholder = "Message your personal CA Wine Country Visit Assistant..."
    st.sidebar.selectbox("Select a Snowflake Cortex model:", MODELS, key="model_name")
    st.sidebar.checkbox('Use your Fivetran dataset as context?', key="dataset_context", help="""This turns on RAG where the 
    data replicated by Fivetran and curated in Snowflake will be used to add to the context of the LLM prompt.""")
    if st.button('Reset conversation', key='reset_conversation_button'):
        st.session_state.conversation_state = []
        st.session_state.reset_key += 1
        st.experimental_rerun()
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
      including **Snowflake Arctic**, trained by researchers at companies like Mistral, Meta, Google, Reka, and Snowflake.\n\nCortex 
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
                snowflake.cortex.embed_text_768('e5-base-v2', ?)) as v_sim
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
                         f"""<span style='color:#808080;'>{token_count} tokens :small_blue_diamond: {tokens_per_second:.2f} tokens/s :small_blue_diamond: 
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

### Step 7: Have some fun checking out the travel assistant features and creating prompts for unique visits using RAG
* Check out the key features of the application and test the Streamlit application with your own prompts or check out the sample prompts below.

* Key features

    * Choose the Snowflake Cortex LLM (model) that you want the Assistant (and Snowflake Cortex) to use. Snowflake Cortex LLM functions and models are specific to each Snowflake region. Our lab is running in aws-us-west2. [Here are the Snowflake Cortex LLMs that are available by region](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#availability).

    * You can toggle between the Assistant **“using”** or **“not using”** your new Fivetran dataset as context (use Retrieval Augmented Generation, RAG, or don't use RAG). If you choose NOT to use RAG and the new Fivetran dataset as context, the models will be operating solely from their original training dataset.

    * Advanced options: The **“Select number of context chunks”** setting allows you to control how many chunks/records are passed to the LLM to be used for RAG processing.  This can be useful when you begin seeing issues like too much data sent to the LLM (too high of a chunk number) or where data returns as “unknown” or badly hallucinates on RAG data (too low of a chunk number).  The names of the wineries used will be displayed at the bottom of the results so you can see which RAG items were selected.

        * Special note about **chunks**: Inspect the number of chunks and which chunks are sent to the LLM.  If your prompt is asking for 7 wineries and chunk size is set to 6, for example, then you would need to increase your chunk size (try 10) to ensure the full context you want sent to the LLM is added.
    
    * You can "reset" your conversation with the CA Wine Country Visit Assistant at any time by clicking the **“Reset conversation”** button. That will clear your input, but your settings in the side panel remain the same.
    
    * After each run, inspect the **token count** at the bottom of the response (see image below). Note the differences in token size when RAG is turned off and on as well as the size and complexity of the prompt. Review the [Snowflake model restrictions](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#model-restrictions) and [cost considerations](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#cost-considerations) to understand more about these models, token inputs, and token costs.

**Example prompt 1** (Note: Kohlleffel Vineyards is a control record that only lives in the PostgreSQL dataset that Fivetran moved into Snowflake)
```
Tell me everything you know about kohlleffel vineyards in 6-7 paragraphs. I want to know about their dog and the types of wood that you can select for the fire pits.
```
**Example prompt 2** (Note: Millman Estate is also a control record that only lives in the PostgreSQL dataset that Fivetran moved into Snowflake)
```
Tell me about kohlleffel vineyards and millman estate. Also, provide me with a 2 day itinerary with a catchy name that includes day 1 stops at kohlleffel vineyards and millman estate. Then on Day 2 I want to visit Continuum Estate and Del Dotto Vineyards. Include information about the dogs at any of the wineries. Also, include recommendations for restaurants and hotels throughout the visit. Also, what types of clothing should I bring for this time of year. Lastly, what will this trip cost me?
```

**Control records in the PostgreSQL dataset (for testing RAG)**
* Kohlleffel Vineyards
* Millman Estate
* Hrncir Family Cellars
* Kai Lee Family Cellars
* Tony Kelly Pamont Vineyards

### Fivetran + Snowflake California Wine Country Visit Assistant

![Travel Assistant Screenshot](./images/2024-10-10%20Streamlit%20-%20Travel%20Assistant.png)

-----
