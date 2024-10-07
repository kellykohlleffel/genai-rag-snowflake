# Create a wine country travel assistant with Fivetran, Snowflake, Cortex, and Streamlit
## Scripts and code for the Fivetran + Snowflake RAG-based, Gen AI Hands on Lab (60 minute version)

This repo provides the high level steps to create a RAG-based, Gen AI travel assistant using Fivetran and Snowflake (detailed instructions are in the lab guide provided by your lab instructor). The required transformation scripts and the required Streamlit code is included. This repo is the "easy button" to copy/paste the transformations and the code. If you have any issues with copy/paste, you can download the code [here](https://github.com/rikthefrog/rag-fivetran/archive/refs/heads/main.zip).

### STEP 1: Create a Fivetran connector to Snowflake

* Source: Google Cloud PostgreSQL (G1 instance)
* Fivetran Destination: DATABRICKS_UNITY_CATALOG_SERVERLESS
* Schema name: yourlastname_yourfirstname 
* Host: 34.94.122.157 (see the lab guide for credentials)
* Schema: agriculture
* Table: california_wine_country_visits

### STEP 2: View the new dataset in Snowflake Snowsight

* Snowflake Account: https://dma21732.snowflakecomputing.com (see the lab guide for credentials)
* Snowflake Database: HOL_DATABASE
* Schema: yourlastname_yourfirstname_agriculture 
* Table: california_wine_country_visits
* Click on **Data Preview** to take a look

### STEP 3: Transform the new structured dataset into a single string to simulate an unstructured document
* Open a new worksheet in Snowflake Snowsight
* Make sure you set the worksheet context at the top: **HOL_DATABASE** and **yourlastname_yourfirstname schema name**
* Copy and paste these [**transformation scripts**](01-transformations.sql) in your Snowsight worksheet 
* Highlight the first transformation script and click run
* This will create a winery_information table using CONCAT to create a single string for each winery or vineyard (creates an "unstructured" document for each winery or vineyard)

### STEP 4: Create the embeddings and the vector table from the winery_information single string table
* Highlight the second transformation script and click run
* This will create your embeddings and a vector table that will be referenced later by Cortex LLM functions and your Streamlit application



### Create the streamlit application

* Copy this [code](./04-streamlit-code.py) in the streamlit editor.

### Finaly have fun

* Test the streamlit application with your questions

## Fill out our questionnaire

[questionnaire](https://forms.gle/jn8nNqjzTnCeZLQT7)
