# :wine_glass: Create a wine country travel assistant with Fivetran, Snowflake, Cortex, and Streamlit
## Scripts and code for the Fivetran + Snowflake RAG-based, Gen AI Hands on Lab (60 minutes)

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

/** Transformation #2 - Using the Snowflake Cortex embed_text_768 LLM function, creates embeddings from the newly created vineyard_data_single_string table and creates a vector table called winery_embedding.
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
* Position your cursor anywhere in the second transformation script in your Snowflake Snowsight worksheet and click run
* This will create your embeddings and a vector table that will be referenced later by Cortex LLM functions and your Streamlit application

```
    /** Create the vector table from the wine review single field table **/
      CREATE or REPLACE TABLE vineyard_data_vectors AS 
            SELECT winery_or_vineyard, winery_information, 
            snowflake.cortex.EMBED_TEXT_768('e5-base-v2', winery_information) as WINERY_EMBEDDING 
            FROM vineyard_data_single_string;
```

### STEP 5: Run a SELECT statement to check out the LLM-friendly "text" document table and embeddings table
* Position your cursor anywhere in the third script **SELECT * FROM vineyard_data_vectors WHERE winery_information LIKE '%winery name is Kohlleffel Vineyards%';** in your Snowflake Snowsight worksheet and click run
* This will show you the complete results of the 2 transformations that you just ran

```
    /** Select a control record to see the LLM-friendly "text" document table and the embeddings table **/
    SELECT *
    FROM vineyard_data_vectors
    WHERE winery_information LIKE '%winery name is Kohlleffel Vineyards%';
```

### STEP 6: Create the a Streamlit app and build a Visit Assistant Chatbot
* Open a New Streamlit application in Snowflake Snowflake (left gray navigation under Projects)
* Highlight the "hello world" Streamlit code and delete it
* Click Run to clear the preview pane
* Copy and paste the [**Streamlit code**](02-streamlit-code.py) in the Streamlit editor

### Step 7: Have some fun checking out the travel assistant features and creating prompts for unique visits using RAG
* Test the streamlit application with your own prompts or check out the sample prompts in the lab guide
