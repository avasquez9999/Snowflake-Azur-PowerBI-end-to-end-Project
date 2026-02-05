# Snowflake-Azur-PowerBI-end-to-end-Project
Snowflake &amp; Power BI: Consumer Insights Lab data enginering and analytics
# ❄️ Snowflake & Power BI: Consumer Insights Lab

![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=power-bi&logoColor=black)
![Azure](https://img.shields.io/badge/Azure-0089D6?style=for-the-badge&logo=microsoft-azure&logoColor=white)

## 📊 Project Overview
This project demonstrates an end-to-end data engineering and analytics pipeline. It integrates **Snowflake's Data Cloud** with **Microsoft Power BI** to process over **1.7 billion rows** of point-of-sale data, providing actionable insights through high-performance dashboards.

### Key Objectives
* **Data Ingestion:** Automate the flow from Azure Blob Storage to Snowflake.
* **Elastic Scaling:** Scale Snowflake compute from `X-Small` to `X-Large` for high-speed ingestion.
* **Data Warehousing:** Implement a **Star Schema** to optimize query performance.
* **BI Optimization:** Utilize **Composite Models** (DirectQuery + Aggregations) in Power BI.

---

## 🏛 Data Warehouse Architecture: The Star Schema
To ensure the best performance in Power BI, this project organizes data into a **Star Schema**. This minimizes joins and allows Power BI to scan data more efficiently.

* **Fact Table:** `SALES_ORDERS_V` (Central table containing quantitative metrics like Quantity and Price).
* **Dimension Tables:** `LOCATION_V`, `ITEMS_V`, `CHANNELS`, and `STATES` (Descriptive tables that filter and group the Fact data).



---

## 💻 Technical Implementation (Full SQL Code)

### 1. Environment & Security Setup
Initialize the warehouses and create a dedicated service user for the Power BI connection.

```sql
USE ROLE ACCOUNTADMIN;

-- Create Warehouses
CREATE OR REPLACE WAREHOUSE ELT_WH WITH WAREHOUSE_SIZE = 'X-SMALL' AUTO_SUSPEND = 120 AUTO_RESUME = true;
CREATE OR REPLACE WAREHOUSE POWERBI_WH WITH WAREHOUSE_SIZE = 'MEDIUM' AUTO_SUSPEND = 120 AUTO_RESUME = true;

-- Role & User Security
CREATE OR REPLACE ROLE POWERBI_ROLE;
GRANT USAGE ON WAREHOUSE POWERBI_WH TO ROLE POWERBI_ROLE;

CREATE OR REPLACE USER POWERBI 
    PASSWORD = 'PowerBI_AV_2026!'
    DEFAULT_ROLE = 'POWERBI_ROLE'
    DEFAULT_WAREHOUSE = 'POWERBI_WH';

GRANT ROLE POWERBI_ROLE TO USER POWERBI;
ALTER USER POWERBI SET DISABLE_MFA = TRUE;
-- Create External Stage
CREATE OR REPLACE STAGE LAB_DATA_STAGE 
URL='azure://ab12345lab2.blob.core.windows.net/labdata'
CREDENTIALS=(AZURE_SAS_TOKEN='[REDACTED_SAS_TOKEN]');

-- Scale Up for Ingestion
ALTER WAREHOUSE ELT_WH SET WAREHOUSE_SIZE = 'X-LARGE';

COPY INTO ITEMS_IN_SALES_ORDERS FROM @LAB_DATA_STAGE/items_in_sales_orders/ 
FILE_FORMAT = (TYPE = 'CSV' FIELD_DELIMITER = ',' SKIP_HEADER = 0);

-- Scale Down to Save Credits
ALTER WAREHOUSE ELT_WH SET WAREHOUSE_SIZE = 'X-SMALL';
. Data Ingestion PipelineStaging data from Azure Blob Storage and executing high-volume COPY commands.SQL-- Create External Stage
CREATE OR REPLACE STAGE LAB_DATA_STAGE 
URL='azure://ab12345lab2.blob.core.windows.net/labdata'
CREDENTIALS=(AZURE_SAS_TOKEN='[REDACTED_SAS_TOKEN]');

-- Scale Up for Ingestion
ALTER WAREHOUSE ELT_WH SET WAREHOUSE_SIZE = 'X-LARGE';

COPY INTO ITEMS_IN_SALES_ORDERS FROM @LAB_DATA_STAGE/items_in_sales_orders/ 
FILE_FORMAT = (TYPE = 'CSV' FIELD_DELIMITER = ',' SKIP_HEADER = 0);

-- Scale Down to Save Credits
ALTER WAREHOUSE ELT_WH SET WAREHOUSE_SIZE = 'X-SMALL';
3. Star Schema Modeling (Views)Creating the semantic layer for Power BI reporting.SQL-- Location Dimension View
CREATE OR REPLACE VIEW PUBLIC.LOCATION_V AS
SELECT
    L.LOCATION_ID, L.COUNTRY, L.REGION, L.MUNICIPALITY,
    S.STATE_NAME, L.LONGITUDE, L.LATITUDE
FROM LOCATIONS L
INNER JOIN STATES S ON L.REGION = S.REGION;

-- Aggregated Fact Table (For Summary Dashboards)
CREATE OR REPLACE VIEW PUBLIC.SALES_ORDERS_V_AGG AS
SELECT
    ITEM_ID, CHANNEL_CODE AS CHANNEL_ID, LOCATION_ID,
    SUM(QUANTITY) TOTAL_QUANTITY
FROM PUBLIC.SALES_ORDERS_V
GROUP BY 1, 2, 3;
🚀 Connection ParametersUse these credentials in Power BI Desktop to connect to your Snowflake instance:PropertyValueServer[Account_Identifier].snowflakecomputing.comWarehousePOWERBI_WHDatabaseLAB_DBAuth ModeDatabase (User: POWERBI / Password: PowerBI_AV_2026!)🧹 Environment ResetRun this script to clean up your Snowflake trial account:SQLUSE ROLE ACCOUNTADMIN;
DROP DATABASE IF EXISTS LAB_DB;
DROP WAREHOUSE IF EXISTS ELT_WH;
DROP WAREHOUSE IF EXISTS POWERBI_WH;

### Next Steps
If you want to dive deeper into the Power BI side of this project, these guides are excellent for mastering the "Star Schema" and "Composite Model" concepts discussed in your lab.

The [Power BI Data Modeling eBook](http://googleusercontent.com/shopping_content/0_link) focuses heavily on architectural best practices like the ones you just implemented.
http://googleusercontent.com/shopping_content/1_card

The [Microsoft Power BI Quick Start Guide](http://googleusercontent.com/shopping_content/2_link) is great for learning the visual side of "Consumer Insights" reporting once your Snowflake data is ready.
http://googleusercontent.com/shopping_content/3_card

Would you like me to help you write a **Python script** to automate the creation of these Snowflake tables from a local CSV file?
