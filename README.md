# Azure-Databricks-1st-Project
From RAW data ingestion to the dashboard and ML model

1.	Initial configuration in Azure
We start by testing the Azure + Databricks option:

We create a Microsoft Azure account. From the Azure portal, we navigate to Azure Databricks and create the resource, which automatically generates all the required infrastructure: networks, storage, identities, and connectors.


 <img width="975" height="311" alt="image" src="https://github.com/user-attachments/assets/46178a48-2f05-433b-a70a-aff1ab863e9b" />

Using Databricks through Azure, instead of accessing it directly via databricks.com, provides key advantages:
-	Azure Active Directory (AAD): enterprise authentication and SSO 
-	 Unity Catalog: data governance, lineage, and permissions 
-	Storage Accounts: integrated external storage 
-	Power BI / Fabric: native connection via AAD authentication

However, the Community Edition:

•	❌ No Azure Active Directory
•	❌ No Storage Accounts
•	❌ No Unity Catalog
•	❌ No Azure clusters
•	❌ No VNET
•	❌ No enterprise security
•	❌ Cannot connect to Power BI with AAD authentication
•	❌ Cannot connect to Fabric
•	❌ Not suitable for professional projects

1.1 Genie AI — Databricks Assistant
Genie AI is the built-in assistant in Databricks. It generates code, provides explanations, creates pipelines, and answers questions about data using natural language. At certain points, we also use it to see how it works and evaluate how reliable it is.

2. Workspace Navigation
Once inside the Databricks workspace, the main sections are:

•	Workspace: personal space with notebooks, folders, scripts, and dashboards
•	Recents: quick access to recently used resources
•	Catalog: the heart of data governance. Contains catalogs, schemas, and tables (Bronze, Silver, Gold). With Unity Catalog, it manages permissions, lineage, and security
•	Jobs & Pipelines: ETL automation, pipelines, and workflows
•	Compute: cluster management and compute configuration
•	Runs: execution history for auditing and debugging
•	Playground (AI/ML): area to experiment with models, prompts, and agents
•	Agents / Experiments / Models / Serving: everything related to MLOps



3. Project Steps+ Medallion Architecture 
The project follows the Medallion Architecture (Bronze → Silver → Gold), the professional standard for organizing a Lakehouse in Databricks.

•	Create the catalog and schema (Unity Catalog)
Defines where tables live. Example: main catalog with bronze, silver, and gold schemas.
•	Create the Storage / External Location
Physical location where RAW data is stored. Azure creates this automatically when the workspace is created.
•	Load RAW data (Bronze)
Datasets are uploaded in CSV, JSON, or Parquet format to the Storage Account.
•	Transform to Silver (cleaning)
SQL or PySpark is used to clean, type, and normalize the data.
•	Create Gold tables (analytics-ready)
Tables ready for consumption by Power BI or Fabric.
•	Connect Power BI
Using the native Azure Databricks connector with AAD authentication.


First Step
We go to Notebook, select SQL, and assign a cluster. If we don’t have one, we create it.
<img width="736" height="338" alt="image" src="https://github.com/user-attachments/assets/6ca8f986-5211-4eb3-9cfd-c70d46e95898" />

<img width="736" height="503" alt="image" src="https://github.com/user-attachments/assets/2d6b4f27-1052-4a97-8810-f03bf17c8e10" />

After that, we can create our first Catalog (main container).
Another option, if we don’t want to create a catalog, is to use the default main catalog provided by Databricks. Then we create schemas, which are like folders: one for RAW data, one for cleaned data, and one for analytics.

<img width="463" height="223" alt="image" src="https://github.com/user-attachments/assets/4ca171e3-45dc-44cd-a9ea-89c81a5f0015" />


⚠️ Important: A Premium version is required to create catalogs and schemas with Unity Catalog.
In the free version, only spark_catalog and hive_metastore.default can be used.

Since we do not have the Premium version, we cannot create catalogs or schemas.

To have a Lakehouse without Unity Catalog, you must create an external Storage Account and organize your zones (raw, cleaned, analytics) as folders in the Data Lake.

If we only have the free version, it is recommended to create a storage account in Azure, an external container, and create folders in our data lake.
<img width="975" height="359" alt="image" src="https://github.com/user-attachments/assets/0f85dde4-41ff-4864-9c7f-0c183e688689" />

In short: Without Unity Catalog (free version), Databricks does not allow creating catalogs or schemas. You can only use spark_catalog and hive_metastore.default.

Databricks Community Edition
At this point, due to the limitations, we choose the alternative path: creating a personal Databricks Community Edition account.
We learned part of Azure, but at this stage we decided to continue directly in Databricks with a personal account, not via Azure.
For this exercise, we select the dataset provided by Databricks: TPCH.
<img width="188" height="263" alt="image" src="https://github.com/user-attachments/assets/588604d5-519d-4e2a-bd85-65be6efe7989" />

It is important to clarify that this catalog does not come with tables already created. The catalog is actually empty until we create the tables ourselves.
4. TPCH Dataset TPCH – Sample Data
For the exercise, we use the TPCH dataset provided by Databricks, available under /databricks-datasets/. These are CSV and JSON files without headers, so schemas must be defined manually and data must be converted into Delta tables.

4.1 Raw Tables (Bronze)
We create a notebook, define the schema manually, and generate the Delta table orders_raw for sales orders.
<img width="789" height="318" alt="image" src="https://github.com/user-attachments/assets/15d8e8b4-d495-49df-8df6-df18e92be9a3" />

The original text is separated by |, has no headers, and no data types, so we must define them manually.
The false value indicates that the column cannot contain null values.

<img width="789" height="161" alt="image" src="https://github.com/user-attachments/assets/0c212dd4-ae2f-4dd3-9bce-1c38073b8b8e" />

We generate and save a Delta table called orders_raw.

<img width="785" height="58" alt="image" src="https://github.com/user-attachments/assets/9516c3d9-f298-461e-8542-2f06704d20fc" />


*A Delta table is Databricks' native format: it combines CSV + Parquet + version control into a single solution.

Now, using SQL we create a delta table orders_cleaned (silver) from the orders_raw

4.2 Cleaned Tables (Silver)
Using SQL, we create orders_cleaned from orders_raw by applying quality filters:
•	Orders with total price ≤ 0 are removed
•	Only orders with status F (Completed), O (Open), or P (Pending) are kept

<img width="397" height="403" alt="image" src="https://github.com/user-attachments/assets/0b6d3d8b-2291-4abb-a745-63d4ef9669b5" />

We repeat the same process (RAW → CLEANED) for the rest of the datasets.
For example, lineitem.
<img width="774" height="636" alt="image" src="https://github.com/user-attachments/assets/eaef1af3-81d8-4ec8-b6af-80aa6909d5a9" />
Here is your silver table (cleaned) 

<img width="886" height="572" alt="image" src="https://github.com/user-attachments/assets/f40e0cf6-61ba-4655-9e59-4b422e793f08" />

5. Dimensional Model — Fact & Dim Tables
With cleaned tables, we build the dimensional model: one central fact table and multiple dimension tables.
5.1 Fact Table — fact_orders
We create a table using an INNER JOIN between orders_cleaned and lineitem_cleaned.
The join condition is:

 
* We use an inner join to keep only rows where the order exists in both tables.

<img width="604" height="773" alt="image" src="https://github.com/user-attachments/assets/aab55cc2-16c6-4bd4-ad0c-e984d60e0aac" />

5.2 Dimension Tables
dim_date
This table contains one row per day within the detected date range.
Minimum and maximum dates are calculated from:
•	orderdate
•	shipdate
•	commitdate
•	receiptdate

<img width="575" height="423" alt="image" src="https://github.com/user-attachments/assets/e36502d3-df49-49ff-860e-0f7ccf2f4f84" />

Now we say: create or replace a table called dim_date as if it were (as) the following temporary table CTE (with date_range as..)

<img width="484" height="516" alt="image" src="https://github.com/user-attachments/assets/0c415898-a958-4e4f-9407-4d784c564b29" />

Then use sequence and pass it (first parameter, second parameter, step) and call it “dates”. Explode() turns a list into many rows. In our case, it turns the list we have created with the CTE above called dates into many rows that it calls date. That is, it creates one row for each date, and now we have a column called date. Then we extract the year, month… generating new columns.

dim_customer y dim_supplier
RAW and CLEANED tables are created for both customer and supplier using the same Bronze → Silver pattern.

6.1 Jobs and Pipelines
A Job is a notebook that runs automatically based on a schedule or trigger. It is used for executing notebooks, refreshing tables, and ETL jobs.
⚠️ Jobs are not available in Community Edition.

In this project, a notebook is created to count rows and a Job is executed manually to test functionality.
<img width="906" height="445" alt="image" src="https://github.com/user-attachments/assets/08b95320-023e-44be-95aa-0712cbf20fd9" />

6.2 Pipelines — Delta Live Tables
Pipelines are similar to Jobs but specialized for creating and refreshing Delta tables declaratively.
⚠️ Pipelines do not support magic commands or multiple languages per notebook.

Here, as an example, a notebook is created to calculate daily sales and a Pipeline from an asset.

<img width="859" height="613" alt="image" src="https://github.com/user-attachments/assets/7904ed56-5ee0-44fe-a9f9-3690fc962058" />


7. Dashboard in Databricks
Dashboards are built in the Dashboards section. They can be based on catalog tables, files, or SQL queries.
We run queries in the SQL Editor and save them as datasets.

Process: Go to the SQL Editor and execute a query. For example: sales per year by joining fact_orders with dim_date.

<img width="906" height="341" alt="image" src="https://github.com/user-attachments/assets/4af8d345-7211-41c6-8abe-640388e9603e" />

Then we create three more files or queries, making a total of 4 different queries. After that, in all of them we click on "Add to dashboard" and select the previously created dashboard. We have called it Sales dashboard
<img width="519" height="391" alt="image" src="https://github.com/user-attachments/assets/b603dc24-9b8b-469f-913f-1b1918f17380" />

In this way we will have everything in one. Then, when we go to the dashboard in the data section, we will be able to see which data feed the visualizations, which in our case are the 4 SQL queries.

<img width="403" height="384" alt="image" src="https://github.com/user-attachments/assets/704aefb9-0868-4c09-998a-e56666a618bb" />

Then, in the design or visualization part, you must select on the right which data set to read the data from

<img width="969" height="422" alt="image" src="https://github.com/user-attachments/assets/77f680d9-93ee-4149-bbb5-e9a297d1a88a" />

8. PBI Connection
To connect Power BI Desktop to Databricks, two parameters are required:
•	Hostname: identifies the Databricks workspace
•	HTTP Path: identifies the SQL Warehouse


<img width="906" height="514" alt="image" src="https://github.com/user-attachments/assets/33cf37a1-76b9-4e6b-b6c7-b60261142bde" />

8.1 Connector Options
There are two connector options:
<img width="234" height="106" alt="image" src="https://github.com/user-attachments/assets/f05a15c0-d92b-4d5e-be9d-d4118affb041" />

Azure Databricks
Uses Azure Active Directory (AAD) for authentication. Intended for workspaces in Azure with corporate users (name@company.com) and SSO with Azure AD.

Databricks (conector genérico)
It works with personal accounts (Gmail, Outlook) and supports Personal Access Token (PAT), username/password, and Service Principal.

For this project, the recommended option is PAT (Personal Access Token)

9. AI/ML — AutoML Forecasting

We explore Databricks Machine Learning capabilities by creating a real sales forecasting model.
Databricks allows three approaches:

•	AutoML (fast, no code)
•	MLflow + Python (professional)
•	Feature engineering + Prophet / ARIMA (classical)

In this project, AutoML is used to predict total sales per month. AutoML automatically generates the notebook, the model, the experiment, the pipeline, and the metrics dashboard.
For this, we go to the Experiments section and select Forecasting and select our set
<img width="438" height="98" alt="image" src="https://github.com/user-attachments/assets/f8a4cf5c-16aa-4d85-97da-01ff85f70f7b" />

After this, autoML has created a model and a table with the predictions. From here we can create an endpoint, view the data, or create a dashboard

<img width="719" height="259" alt="image" src="https://github.com/user-attachments/assets/b07a0d4f-2f5f-434e-9188-6beaa7c1f512" />

10. LLM – Conversational AI over Data Notebook)
Now we want to see if we can, with normal language, ask the AI questions and have it consult our dataset and give us the answers.

In our case, the workspace does not have the advanced features of Databricks Agents or AI Playground with persistent agents enabled. Connecting to an external AI (like ChatGPT via API) requires a paid version, so it is also ruled out.

Implemented solution: conversational chat within a Databricks notebook with the following flow:

1. The user writes a question in a text box in the notebook.
2. Qwen (open-source model available in Databricks) generates the corresponding SQL query.
3. The notebook executes the SQL in the SQL Warehouse.
4. Qwen interprets the result and returns it in natural language.

This workflow was implemented with the support of Claude and Genie (Databricks AI) to generate the notebook code. After generating all the code, a checkbox is enabled in that same notebook asking us what we want to know.

Here is an example of the result:
<img width="813" height="681" alt="image" src="https://github.com/user-attachments/assets/b9e33dbc-b42c-47bd-8720-9d53aad848b0" />






