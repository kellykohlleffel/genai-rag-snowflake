# :wine_glass: Create a wine country travel assistant with Fivetran, Snowflake, Cortex, and Streamlit
## Scripts and code for the Fivetran + Snowflake RAG-based, Gen AI Hands on Lab
### 60 minute version

This repo provides the high level steps to create a RAG-based, Gen AI travel assistant using Fivetran and Snowflake (detailed instructions are in the lab guide provided by your lab instructor). The required transformation scripts and the required Streamlit code are both included. This repo is the "easy button" to copy/paste the transformations and the code. If you have any issues with copy/paste, you can download the code [here](https://github.com/kellykohlleffel/genai-rag-snowflake/archive/refs/heads/main.zip).

### STEP 1: Create a Fivetran connector to Snowflake

* **Source**: Google Cloud PostgreSQL (G1 instance)
* **Fivetran Destination**: SNOWFLAKE_LLM_LAB
* **Schema name**: yourlastname_yourfirstname 
* **Host**: 34.94.122.157 **(see the lab guide for credentials)**
* **Schema**: agriculture
* **Table**: california_wine_country_visits

### STEP 2: View the new dataset in Snowflake Snowsight

* **Snowflake Account**: **https://dma21732.snowflakecomputing.com** **(see the lab guide for credentials)**
* **Snowflake Database**: HOL_DATABASE
* **Schema**: yourlastname_yourfirstname_agriculture 
* **Table**: california_wine_country_visits
* Click on **Data Preview** to take a look

### STEP 3: Transform the new structured dataset into a single string to simulate an unstructured document
* Open a New Worksheet in **Snowflake Snowsight** (left gray navigation under Projects)
* Make sure you set the worksheet context at the top: **HOL_DATABASE** and **yourlastname_yourfirstname schema name**
* Copy and paste these [**transformation scripts**](01-transformations.sql) in your Snowsight worksheet 
* Position your cursor anywhere in the first transformation script and click run
* This will create a new winery_information table using the CONCAT function. Each multi-column record (winery or vineyard) will now be a single string (creates an "unstructured" document for each winery or vineyard)

### STEP 4: Create the embeddings and the vector table from the winery_information single string table
* Position your cursor anywhere in the second transformation script in your Snowflake Snowsight worksheet and click run
* This will create your embeddings and a vector table that will be referenced later by Cortex LLM functions and your Streamlit application

### STEP 5: Run a SELECT statement to check out the LLM-friendly "text" document table and embeddings table
* Position your cursor anywhere in the third script **SELECT * FROM vineyard_data_vectors WHERE winery_information LIKE '%winery name is Kohlleffel Vineyards%';** in your Snowflake Snowsight worksheet and click run
* This will show you the complete results of the 2 transformations that you just ran

### STEP 6: Create the a Streamlit app and build a Visit Assistant Chatbot
* Open a New Streamlit application in Snowflake Snowflake (left gray navigation under Projects)
* Highlight the "hello world" Streamlit code and delete it
* Click Run to clear the preview pane
* Copy and paste the [**Streamlit code**](02-streamlit-code.py) in the Streamlit editor

### Step 7: Have some fun checking out the travel assistant features and creating prompts for unique visits using RAG
* Test the streamlit application with your own prompts or check out the sample prompts in the lab guide
