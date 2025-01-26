# SAP Supply Chain Intelligence Streamlit in Snowflake Data App

## Overview
 The **SAP Supply Chain Intelligence Data Application** combines **Fivetran**, **Snowflake**, **dbt**, **Streamlit**, and **Gen AI** (powered by **Snowflake Cortex**) to deliver insights into SAP supply chain and purchase order data. This application extracts raw data from SAP HANA, transforms it into Generative AI-ready formats, and enables Gen AI-powered analysis through an interactive data application.

---

## Data Flow

### 1. **Bronze Layer**: Raw Data Ingestion
- **Source**: SAP HANA
- **Ingestion Tool**: Fivetran's **Automated Data Movement** platform using the SAP ERP on HANA connector.
- **Raw Tables**: Data is ingested into the `HOL_DATABASE.DHSAPHANA_SAPABAP1` schema.

**List of Tables**:
- `AFIH`, `AFKO`, `AUFK`, `DD07L`, `DD07T`, `EKBE`, `EKES`, `EKET`, `EKKO`, `EKPO`
- `KNA1`, `LFA1`, `LIKP`, `LIPS`, `MAKT`, `MARA`, `PMCO`, `QMEL`, `QMFE`, `QMIH`
- `QMSM`, `QPCD`, `QPCT`, `QPGR`, `QPGT`, `T001`, `T001W`, `T005`, `T005T`, `T024`
- `T024E`, `T025`, `T025T`, `T134`, `T134T`, `T161`, `T161T`, `T165M`, `T165R`, `T179`
- `T179T`, `TCURC`, `TCURT`, `TCURV`, `TKA01`, `TSPA`, `TSPAT`, `TVAG`, `TVAGT`, `TVAU`
- `TVAUT`, `TVKO`, `TVKOT`, `VBAK`, `VBAP`, `VBKD`, `VBPA`, `VBRK`, `VBRP`, `VBUK`, `VBUP`

---

### 2. **Silver Layer**: Data Transformation with dbt
- **Schema**: `DHSAPPROD_DHSAPHANA_STG`
- **Purpose**: Transform raw data into standardized, enriched views for analysis.

**Examples of Views**:
- `VW_COMPANY`, `VW_CUSTOMER`, `VW_MATERIAL`, `VW_PLANT`

---

### 3. **Gold Layer**: Analytics-ready Data
- **Schema**: `DHSAPPROD_DHSAPHANA_BI`
- **Purpose**: Provide optimized views and derived tables for reporting and analytics.

**Examples of Views**:
- `D_CUSTOMER`, `D_MATERIAL`, `D_PLANT`

---

### 4. **Single String Table for Gen AI Analysis**
- **Table**: `HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.PO_DATA_SINGLE_STRING`
- **Purpose**: Create a single descriptive string combining multiple fields from purchase order data for AI-driven analysis.

**SQL**:
```sql
-- Create single string representation of PO data
CREATE OR REPLACE TABLE HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.PO_DATA_SINGLE_STRING AS 
SELECT 
    F_PURCHASING_ORDER.PURCHASING_DOCUMENT_ID,
    CONCAT(
        'Purchase Order ', F_PURCHASING_ORDER.PURCHASING_DOCUMENT_ID, 
        ' was created on ', F_PURCHASING_ORDER.PURCHASING_DOCUMENT_DATE, '.',
        ' The vendor is ', D_VENDOR.NAME, ' (', D_VENDOR.VENDOR_ID, ') from ', D_VENDOR.CITY, '.',
        ' The purchasing organization is ', D_PURCHASING_ORGANIZATION.DESCRIPTION, '.',
        ' The plant is ', D_PLANT.PLANT_NAME, '.',
        ' The material ordered is ', D_MATERIAL.MATERIAL_DESCRIPTION, ' (', D_MATERIAL.MATERIAL_NUMBER, ').',
        ' The order quantity is ', F_PURCHASING_ORDER.PURCHASE_ORDER_QUANTITY, ' ', D_MATERIAL.BASE_UOM_ID, '.',
        ' The order amount is ', F_PURCHASING_ORDER.PURCHASE_ORDER_AMOUNT, ' ', F_PURCHASING_ORDER.DOCUMENT_CURRENCY_ID, '.',
        ' The scheduled delivery date is ', F_PURCHASING_ORDER.SCHEDULED_DELIVERY_DATE, '.',
        ' The document type is ', D_PURCHASING_ORDER.PURCHASING_DOCUMENT_TYPE_TEXT, '.',
        ' The document status is ', D_PURCHASING_ORDER.PURCHASING_DOCUMENT_STATUS_TXT, '.',
        ' The payment terms are ', D_PURCHASING_ORDER.PAYMENT_TERMS
    ) AS po_information
FROM HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.F_PURCHASING_ORDER 
JOIN HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.D_PURCHASING_ORDER 
    ON F_PURCHASING_ORDER.PURCHASING_DOCUMENT_ID = D_PURCHASING_ORDER.PURCHASING_DOCUMENT_ID
JOIN HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.D_PURCHASING_ORGANIZATION 
    ON F_PURCHASING_ORDER.PURCHASING_ORGANIZATION_ID = D_PURCHASING_ORGANIZATION.PURCHASING_ORGANIZATION_ID
JOIN HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.D_VENDOR 
    ON F_PURCHASING_ORDER.VENDOR_ID = D_VENDOR.VENDOR_ID
JOIN HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.D_MATERIAL 
    ON F_PURCHASING_ORDER.MATERIAL_ID = D_MATERIAL.MATERIAL_ID
JOIN HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.D_PLANT 
    ON F_PURCHASING_ORDER.PLANT_ID = D_PLANT.PLANT_ID;
```

### 5. **Vector Embeddings Table**
- **Table**: `HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.PO_DATA_VECTORS`
- **Purpose**: Generate text embeddings for AI-driven insights using Snowflake Cortex.

**SQL**:
```sql
-- Create vector embeddings
CREATE OR REPLACE TABLE HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.PO_DATA_VECTORS AS 
SELECT 
    PURCHASING_DOCUMENT_ID,
    po_information,
    snowflake.cortex.EMBED_TEXT_1024('snowflake-arctic-embed-l-v2.0', po_information) AS PO_EMBEDDINGS
FROM HOL_DATABASE.DHSAPPROD_DHSAPHANA_BI.PO_DATA_SINGLE_STRING;
```
# Streamlit Application

## Application Code

The application logic is implemented in [`streamlit_in_snowflake.py`](./files/streamlit_in_snowflake.py). 

## Features

### Analysis Types:
- Spend Analysis
- Vendor Performance
- Material Usage
- Process Efficiency

### Interactive Metrics Dashboard:
- Display key metrics using `st.metric`.
- Visualize purchase order data and supply chain performance.

### Gen AI Integration:
- Powered by **Snowflake Cortex** using models such as Claude, Llama, Mistral, and Snowflake Arctic.
- AI-driven natural language analysis of supply chain data.

---

## Gen AI Functionality

The application integrates **Snowflake Cortex** for **Gen AI** capabilities:

- **Text Embeddings**: Generate vector embeddings for textual similarity analysis.
- **Custom AI Prompts**: Enable natural language insights based on purchase order data.

### Example Gen AI Prompt

Analyze spend patterns across vendors:

- Total Purchase Orders: 10,000
- Total Vendors: 200
- Total Spend: $5,000,000.00
- Average Purchase Order Value: $500.00

Top Vendors by Spend:
- Vendor A: $1,200,000.00 (24.0%)
- Vendor B: $1,000,000.00 (20.0%)
- Vendor C: $800,000.00 (16.0%)
- Vendor D: $700,000.00 (14.0%)
- Vendor E: $600,000.00 (12.0%)

Provide insights and recommendations based on the data.

# Deployment

## Fivetran

- Automates data movement from SAP HANA to Snowflake using the SAP ERP on HANA connector.
- Ensures reliable, incremental data synchronization for the **Bronze Layer**.
- Leverages **Fivetran Transformation** capabilities with **dbt Core** to transform raw data into business-ready views for the **Silver Layer** and **Gold Layer**.
- Simplifies data ingestion and transformation with minimal setup and maintenance.

## Snowflake
- **Bronze Layer**: Ingest SAP HANA tables using Fivetran.
- **Silver Layer**: Transform raw data into business-ready views with dbt.
- **Gold Layer**: Prepare analytics-ready views and embeddings for Gen AI using **Snowflake Cortex**.

## Streamlit and Cortex
- Hosted natively on **Snowflake** for seamless integration.
- Provides an intutive data application powered by **Gen AI** and **Snowflake Cortex**.
