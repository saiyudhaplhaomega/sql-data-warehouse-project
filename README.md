# sql-data-warehouse-project
# 📊 Modern SQL Data Warehouse Project

Welcome to my project repository! This project demonstrates building a modern SQL Data Warehouse from scratch, following the practical approach shared in the "SQL Data Warehouse from Scratch | Full Hands-On Data Engineering Project" YouTube tutorial by Data with Baraa. ✨

This project provided hands-on experience in the roles of a **Data Architect** (designing architecture), **Data Engineer** (writing code for ETL, preparing data), and **Data Modeler** (creating the data model for analysis) . By the end, I have a professional portfolio project showcasing these skills .

The core goal of this data warehouse is to **consolidate sales data** from disparate sources to enable analytical reporting and informed decision making [2].

## Why Build a Data Warehouse? 🤔

Before diving into the build, it's essential to understand *why* companies build data warehouses. Manual reporting processes in companies without proper data management systems face significant issues [3]:

*   **Slow and Time-Consuming:** Manually generating reports can take days, weeks, or even months [3].
*   **Inconsistent Data:** Different reports may show conflicting data states due to varying refresh schedules [3].
*   **Human Errors:** Manual processes are prone to errors, leading to bad decisions [3].
*   **Big Data Challenges:** Struggling to collect and process massive amounts of data [3].
*   **Difficulty with Integrated Reports:** Merging data from multiple sources manually is chaotic and risky [3].

A data warehouse addresses these issues by providing a **single point of truth** for analysis and reporting [4]. It automates the process, reduces human error, is much faster (hours or minutes vs. weeks/months), allows for easier integration of multiple sources, enables building data history, provides consistent data status across reports, and can handle big data sources easily, especially in the cloud [4, 5]. Think of it like an organized restaurant kitchen preparing ingredients (data) for quick dish creation (reports/analysis) [5].

## Project Phases & Decisions 🚀

Building a data warehouse is a complex project. A clear plan is crucial for success, as over 50% of data warehouse projects reportedly fail [6]. We used Notion for planning, breaking down the work into Epics (large tasks) and Tasks (subtasks) [6].

The project followed these main phases:

1.  **Requirements Analysis 📋**
    *   **Task:** Understand the specific needs for *this* data warehouse [7].
    *   **Decision:** The primary requirement was to build a modern data warehouse using **SQL Server** to consolidate **sales data** from **two sources (ERP and CRM)** provided as **CSV files** [2]. The data needed to be **cleaned and integrated** into a single user-friendly model designed for analytics [2]. The focus should be on the **latest data sets**, meaning **no historization** was required in the final model, and **documentation** of the data model was needed [2]. This analysis shaped the subsequent design and implementation decisions [2, 7].

2.  **Designing Data Architecture 🏛️**
    *   **Task:** Design the blueprint for how data will flow, integrate, and be accessed [8]. This is a key role of a data architect [8].
    *   **Decision (Approach):** We chose the **Data Warehouse** approach as the primary data management system because the project requirements focused on structured data for reporting and business intelligence [8, 9]. We did not choose a Data Lake (more flexible for mixed data types/ML but less organized), Data Lakehouse (mix of DW/DL), or Data Mesh (decentralized) as they were not the best fit for the project's specific requirements focusing on structured data and reporting [9].
    *   **Decision (Structure):** Within the data warehouse approach, we chose the **Medallion Architecture** (Bronze, Silver, Gold layers) [10, 11]. This model is easy to understand and build [10].
        *   **Bronze Layer:** For storing **raw, unprocessed data** as it comes from the source [11]. Purpose: traceability and debugging by keeping untouched data [11]. Object Type: **Tables** [12]. Load Method: **Full Load (Truncate and Insert)** for simplicity and aligning with full extraction [12, 13]. Transformations: **None**, strictly no manipulation [12]. Audience: **Data Engineers** only [14].
        *   **Silver Layer:** For storing **clean and standardized data** [11]. Purpose: basic transformations to prepare data for the final layer [11]. Object Type: **Tables** [12]. Load Method: **Full Load (Truncate and Insert)** [12]. Transformations: **Data Cleansing**, Standardizations, Normalizations, Deriving New Columns, Data Enrichment [12]. Business rules pushed to the next layer [12]. Audience: **Data Engineers, Data Analysts, Data Scientists** [14].
        *   **Gold Layer:** Contains **business-ready data** [11]. Purpose: provide data for business users and analysts for reporting, analytics, etc. [11]. Object Type: **Views** for a virtual, dynamic, and fast final layer [12]. Load Method: **None** (views are virtual) [12]. Transformations: **Data Integrations** (combining sources), **Data Aggregations**, Applying **Business Logic/Rules** [12, 14]. Data Model: Designed using **Star Schema** [14, 15]. Audience: **Data Analysts, Business Users** [14].
    *   **Key Principle:** We applied the **Separation of Concerns** principle, ensuring each layer has unique, non-duplicated tasks (e.g., cleansing only in Silver, business logic only in Gold, raw landing only in Bronze) [16]. This makes the architecture organized and easier to maintain [16].
    *   **Documentation:** The architecture was documented visually using Draw.io, showing the layers, source systems (CSV files), consumer types (BI, Ad-hoc, ML), and key specifications for each layer [16-20].

3.  **Project Initialization 🛠️**
    *   **Task:** Set up the foundational environment for the project, including tools and basic database structure [21, 22].
    *   **Decision (Tools):** We downloaded and installed **SQL Server Express** (the database server), **SQL Server Management Studio (SSMS)** (the client to interact with the database), **Draw.io** (for diagrams), and set up **GitHub** and optionally **Notion** [21]. SQL Server was chosen as per requirements [2].
    *   **Decision (Naming Conventions):** Defined early to ensure consistency across the project, preventing chaos later [23]. We chose **Snake Case** (`all_lowercase_with_underscores`) for object names [23, 24]. Specific rules were defined for tables (`SourceSystem_Entity` for Bronze/Silver, `Category_Entity` with prefixes like `dim_` or `fact_` for Gold) [24, 25], columns (`table_name_key` for surrogate keys, `dw_column_name` for metadata) [25], and stored procedures (`action_layer`) [25].
    *   **Decision (Repository Setup):** Created a public GitHub repository to store code, track changes, and serve as a portfolio [26]. Established a standard folder structure (`datasets`, `documents`, `scripts`, `tests`) to organize project files [27]. Wrote a comprehensive `README.md` file to describe the project [27].
    *   **Decision (Database/Schema Creation):** Created the main database (`Data_Warehouse`) in SQL Server [22]. Created separate schemas within the database for each layer (`bronze`, `silver`, `gold`) to logically separate objects according to the architecture and uphold the separation of concerns principle [22, 28]. Wrote a SQL script for this, including checks to drop the database/schemas if they already exist (`IF OBJECT_ID...DROP TABLE`), adding header comments and `GO` separators [28, 29]. Committed this script to the Git repository [29].

4.  **Building the Bronze Layer 🧱**
    *   **Task:** Implement the data ingestion process to load raw data from source files into the Bronze layer tables [29, 30].
    *   **Process:** Analyzed the source CSV files to understand their structure and metadata [31]. Created DDL scripts to define tables in the `bronze` schema that match the source file structure exactly, following the naming convention (`SourceSystem_Entity`) [32, 33]. Used `IF OBJECT_ID...DROP TABLE` for idempotent script execution [33]. Wrote SQL scripts using the **`BULK INSERT`** command to load data quickly from the CSV files into the corresponding bronze tables [34]. Included options to handle the header row (`FIRSTROW`) and the delimiter (`FIELDTERMINATOR`) [34, 35]. Implemented a **Full Load** strategy for the bronze layer by using `TRUNCATE TABLE` before each `BULK INSERT`, ensuring the tables are refreshed completely each time the script runs [36].
    *   **ETL Script Engineering:** Encapsulated the loading scripts for all bronze tables into a stored procedure (`bronze.load_bronze`) [37]. Added detailed `PRINT` messages to track the process, indicating which source system and table is being loaded [38, 39]. Implemented **Error Handling** using `BEGIN TRY...END CATCH` to catch and report any SQL errors during the load, printing error details like message and number [39, 40]. Added **Load Duration** measurement using `GETDATE()` and `DATEDIFF()` to track how long it takes to load each table and the entire layer, which is crucial for performance monitoring and debugging in real projects [40-42].
    *   **Data Validation:** Performed basic quality checks after loading, like counting rows to ensure all data was loaded [30, 36]. Manually inspected data in SSMS to check for issues like data shifting between columns [35].
    *   **Documentation:** Drew a **Data Flow Diagram** visually representing the flow from Source files (CRM, ERP) to the Bronze tables, showing the lineage [43, 44]. Committed the DDL and stored procedure scripts to the `scripts/bronze` folder in Git [44, 45].

5.  **Building the Silver Layer ✨**
    *   **Task:** Implement the data cleansing and standardization logic to transform data from the Bronze layer and load it into the Silver layer tables [45].
    *   **Process:** **Analyzed and explored the data quality issues** present in the Bronze layer tables. This involved writing queries to check for duplicates, nulls, unwanted spaces, invalid values, inconsistent formatting, and range issues in various columns (e.g., Customer IDs, Names, Gender, Marital Status, Product Keys, Dates, Sales/Quantity/Price) [45-58]. Documented the relationships between source tables by drawing an **Integration Model** using Draw.io, which helped in understanding how data from different sources could be linked [59-63]. Based on the identified issues and hypothetical business rules (e.g., master source, sales calculation logic), wrote **Data Transformation** SQL scripts to clean the data [49-52, 54, 56-58, 64-68]. Transformations included:
        *   **Data Cleansing:** Removing duplicates (using `ROW_NUMBER()`) [47, 64], removing unwanted spaces (`TRIM()`) [48, 65], handling missing/invalid data (replacing nulls/empty strings with default values, using `ISNULL()`, `NULLIF()`, `CASE WHEN`) [50, 51, 54, 57, 65-69], fixing invalid date formats/values (`CAST()`, `CASE WHEN`) [54, 70], fixing invalid numerical values/calculations (`CASE WHEN`, `ABS()`, `NULLIF()`) [66, 71, 72].
        *   **Data Standardization/Normalization:** Mapping abbreviated codes to full names (`CASE WHEN`, `UPPER()`, `TRIM()`) [51, 57, 65, 67-69].
        *   **Deriving New Columns:** Splitting a single column (Product Key) into multiple parts (Category ID, Product Key) using string functions (`SUBSTRING()`, `REPLACE()`, `LEN()`) [49, 73].
        *   **Data Enrichment:** Calculating a new end date for historical records based on the next record's start date (`LEAD()`, `DATEDIFF()`) [74, 75].
        *   **Data Type Casting:** Converting data types (e.g., integer dates to DATE, Date/Time to DATE) (`CAST()`) [70, 76].
    *   **Implementation:** Created DDL scripts for Silver tables by copying Bronze DDLs and replacing the schema name (`bronze.` to `silver.`) [63, 77]. Updated DDLs to include necessary columns for derived data (Category ID) and adjusted data types based on transformations (Dates changed from integer/datetime to DATE) [72, 76]. Added **Metadata Columns** (`dw_create_date`) to all Silver tables with a default value (`GETDATE()`) generated by the database, providing audit information [46, 77, 78]. Wrote INSERT statements to load the transformed data from Bronze SELECT queries into the Silver tables [69, 76, 79-81].
    *   **ETL Script Engineering:** Consolidated the TRUNCATE and INSERT scripts for all Silver tables into a stored procedure (`silver.load_silver`) [82]. Maintained the same engineering standards as the Bronze ETL: including `PRINT` messages, `BEGIN TRY...END CATCH` error handling, and load duration calculation for each table and the total batch [82, 83]. Emphasized the importance of keeping engineering standards consistent across all ETL stored procedures [83].
    *   **Data Validation:** After loading, performed comprehensive **quality checks on the Silver tables** using SQL queries (checking for duplicates, nulls, spaces, consistency, etc.) to ensure the cleansing process was successful [59, 69, 78-81]. This is an iterative process, returning to coding if issues are found [59].
    *   **Documentation:** Extended the **Data Flow Diagram** to show the lineage from the Bronze layer to the Silver layer tables [84, 85]. Committed the Silver DDL and stored procedure scripts to `scripts/silver` in Git [85]. Committed the SQL queries used for quality checks on the Silver layer to the `tests` folder [85, 86].

6.  **Building the Gold Layer 🏆**
    *   **Task:** Build the final, business-ready data model optimized for analytics and reporting by integrating and aggregating data and applying business logic from the Silver layer [86].
    *   **Process:** **Analyzed and explored the business objects** present across the source systems (Customers, Products, Sales) based on the integrated view developed in the Silver layer analysis [86-88]. Decided to build a **Star Schema** data model, comprising **Dimension** tables (descriptive data like Customers and Products) and a **Fact** table (transactional/event data like Sales) [15, 89-92]. This model is commonly used and ideal for BI and reporting [15]. We focused on creating a **Logical Data Model** [89, 93].
    *   **Data Integration & Modeling:** Wrote SQL queries to build the dimensions and fact table by selecting data from the **Silver layer** [88, 94].
        *   **Dimension Customer (`dim_customers`):** Integrated customer data from three Silver tables (CRM Customer Info, ERP Customer, ERP Location) using **`LEFT JOIN`**, starting from the master table (CRM Customer Info) to ensure all customers were included [94, 95]. Applied integration logic for conflicting data points (Gender) by prioritizing the master source (CRM) [96, 97]. Gave friendly, business-aligned column names following the naming convention (e.g., `customer_id`, `first_name`, `country`) [90, 98]. Ordered columns logically [90]. Generated a **Surrogate Key** (`customer_key`) using `ROW_NUMBER()` as the primary key for the dimension, providing a stable, system-generated identifier independent of source keys [90, 99].
        *   **Dimension Product (`dim_products`):** Integrated product data from two Silver tables (CRM Product Info, ERP PX Catalog) using **`LEFT JOIN`** [100, 101]. Filtered data to include only the **latest product information** (`end_date IS NULL`) as per the project requirement to focus on current data [100]. Gave friendly, business-aligned column names (e.g., `product_name`, `category`, `cost`) [91]. Ordered columns logically [91, 101]. Generated a **Surrogate Key** (`product_key`) using `ROW_NUMBER()` [91, 92].
        *   **Fact Sales (`fact_sales`):** Based on the Silver CRM Sales Details table [92]. Performed **Data Lookup** by joining the Fact query with the newly created Gold Dimensions (`dim_customers`, `dim_products`) to retrieve the **Surrogate Keys** (`customer_key`, `product_key`) [92, 102]. Replaced the original source system keys with these surrogate keys, as the fact table should reference dimensions via surrogate keys in a star schema [92, 102]. Gave friendly, business-aligned column names (e.g., `order_number`, `sales_amount`) [102, 103]. Ordered columns logically: Surrogate Keys first, then Dates, then Measures [103].
    *   **Implementation:** Created the Gold layer objects as **Views** (`CREATE VIEW gold.dim_customers AS...`, `CREATE VIEW gold.dim_products AS...`, `CREATE VIEW gold.fact_sales AS...`) [92, 99, 103]. Views are preferred for the Gold layer as they are virtual and always reflect the latest data in the underlying Silver tables without needing a separate load process [12].
    *   **Data Validation:** Checked the quality of the Gold views by performing simple SELECTs [92, 99, 103]. Crucially, verified the **integrity and joinability of the data model** by performing `LEFT JOIN` tests between the Fact table and Dimensions to ensure all records in the Fact could be linked to corresponding Dimension records [103]. Also checked the uniqueness of the Surrogate Keys [99, 101].
    *   **Documentation:** Created a **Logical Data Model diagram** using Draw.io, visually representing the Star Schema (Fact in the middle, Dimensions around) with tables listing columns (PK/FK), and showing the **one-to-many relationships** between dimensions and the fact [103-105]. Added textual descriptions and business rules (like the sales calculation formula) to the diagram for clarity [105, 106]. Created a **Data Catalog** document describing each Gold layer view (purpose, columns, descriptions, and examples for columns) to help end-users understand and consume the data model [106, 107]. Extended the **Data Flow Diagram** to show the lineage from the Silver layer to the Gold layer views, completing the data lineage from sources to the final consumption layer [107, 108]. Committed the Gold views script to `scripts/gold` in Git [108]. Committed the SQL queries used for Gold quality checks to the `tests` folder [109]. Uploaded documentation diagrams and the data catalog document to the `documents` folder in Git and updated the README to link to key parts [109].

## Project Structure 📁

This repository is organized into the following folders:

*   `datasets`: Contains the raw source data files (CSV) [27].
*   `documents`: Includes project documentation such as data architecture diagrams, data flow diagrams, data integration diagrams, data model diagrams, and the data catalog [27, 109].
*   `scripts`: Houses the SQL scripts for database initialization and the ETL processes for the Bronze, Silver, and Gold layers [27, 44, 85, 108].
    *   `scripts/init_database.sql`: Script to create the database and schemas [28].
    *   `scripts/bronze/ddl_bronze.sql`: DDL scripts for creating Bronze layer tables [44].
    *   `scripts/bronze/proc_load_bronze.sql`: Stored procedure for loading the Bronze layer [45].
    *   `scripts/silver/ddl_silver.sql`: DDL scripts for creating Silver layer tables [85].
    *   `scripts/silver/proc_load_silver.sql`: Stored procedure for loading the Silver layer with cleansed data [85].
    *   `scripts/gold/ddl_gold.sql`: Scripts for creating Gold layer views (Dimensions and Fact) [108].
*   `tests`: Contains SQL scripts for data quality checks on the Silver and Gold layers [85, 109].
    *   `tests/quality_checks_silver.sql`: Queries to validate data quality in the Silver layer [85, 86].
    *   `tests/quality_checks_gold.sql`: Queries to validate data integrity and relationships in the Gold layer [109].

## How to Use 🛠️

1.  Download and install **SQL Server Express** and **SSMS** [21].
2.  Clone this repository to your local machine.
3.  Place the downloaded project dataset files (CRM and ERP folders with CSVs) into the `datasets` folder of the cloned repository [21, 27].
4.  Open SSMS and connect to your local SQL Server instance.
5.  Open and execute the `scripts/init_database.sql` script to create the `Data_Warehouse` database and the `bronze`, `silver`, and `gold` schemas [22, 28, 29].
6.  Open and execute the `scripts/bronze/ddl_bronze.sql` script to create the tables in the `bronze` schema [44].
7.  Open and execute the `scripts/bronze/proc_load_bronze.sql` script to create the stored procedure for loading the bronze layer. **Make sure to update the file paths within the script** to match the location of your `datasets` folder on your local machine [34, 37, 45].
8.  Execute the bronze load stored procedure: `EXEC bronze.load_bronze;` [38].
9.  Open and execute the `scripts/silver/ddl_silver.sql` script to create the tables in the `silver` schema [85].
10. Open and execute the `scripts/silver/proc_load_silver.sql` script to create the stored procedure for loading the silver layer [85].
11. Execute the silver load stored procedure: `EXEC silver.load_silver;` [82].
12. Open and execute the `scripts/gold/ddl_gold.sql` script to create the views in the `gold` schema [108].
13. You can now explore the views in the `gold` schema and use them for reporting and analysis! ✨

Feel free to explore the documentation in the `documents` folder and the quality check scripts in the `tests` folder to gain a deeper understanding of the project.

## Showcase Your Skills! 🌟

Having built this project demonstrates practical data engineering skills, including:

*   Designing data architecture (Medallion) [1, 11].
*   Implementing ETL/ELT processes using SQL [1, 110, 111].
*   Building multi-layered data warehouses [4, 10].
*   Performing comprehensive data cleansing and transformation [45, 84, 111, 112].
*   Integrating data from multiple sources [4, 86, 94, 112].
*   Designing and implementing a data model (Star Schema) [1, 14, 89].
*   Creating and using surrogate keys [90, 99].
*   Implementing ETL best practices (stored procedures, logging, error handling, performance measurement) [37-40].
*   Documenting technical processes and data models (diagrams, data catalog) [2, 43, 84, 103, 106].
*   Using Git for version control and collaboration [26].

Feel free to take, modify, and share this project. Showcase your skills on platforms like LinkedIn! [1, 113]

## License 📝

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details [26].

## About Me 👋

[Add a short section about yourself here, perhaps linking to your profiles as suggested in the video] [109]

## Support the Creator 🙏

If you found this project helpful and appreciate the tutorial it's based on, consider supporting the original creator! [113]
