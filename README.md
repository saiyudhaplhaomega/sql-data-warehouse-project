# sql-data-warehouse-project
# 📊 Modern SQL Data Warehouse Project

Welcome to my project repository! This project demonstrates building a modern SQL Data Warehouse from scratch, following the practical approach shared in the "SQL Data Warehouse from Scratch | Full Hands-On Data Engineering Project" YouTube tutorial by Data with Baraa. ✨

This project provided hands-on experience in the roles of a **Data Architect** (designing architecture), **Data Engineer** (writing code for ETL, preparing data), and **Data Modeler** (creating the data model for analysis) . By the end, I have a professional portfolio project showcasing these skills .

The core goal of this data warehouse is to **consolidate sales data** from disparate sources to enable analytical reporting and informed decision making .

## Why Build a Data Warehouse? 🤔

Before diving into the build, it's essential to understand *why* companies build data warehouses. Manual reporting processes in companies without proper data management systems face significant issues :

*   **Slow and Time-Consuming:** Manually generating reports can take days, weeks, or even months .
*   **Inconsistent Data:** Different reports may show conflicting data states due to varying refresh schedules .
*   **Human Errors:** Manual processes are prone to errors, leading to bad decisions .
*   **Big Data Challenges:** Struggling to collect and process massive amounts of data .
*   **Difficulty with Integrated Reports:** Merging data from multiple sources manually is chaotic and risky .

A data warehouse addresses these issues by providing a **single point of truth** for analysis and reporting . It automates the process, reduces human error, is much faster (hours or minutes vs. weeks/months), allows for easier integration of multiple sources, enables building data history, provides consistent data status across reports, and can handle big data sources easily, especially in the cloud . Think of it like an organized restaurant kitchen preparing ingredients (data) for quick dish creation (reports/analysis) .

## Project Phases & Decisions 🚀

Building a data warehouse is a complex project. A clear plan is crucial for success, as over 50% of data warehouse projects reportedly fail. We used Notion for planning, breaking down the work into Epics (large tasks) and Tasks (subtasks) .

The project followed these main phases:

1.  **Requirements Analysis 📋**
    *   **Task:** Understand the specific needs for *this* data warehouse .
    *   **Decision:** The primary requirement was to build a modern data warehouse using **SQL Server** to consolidate **sales data** from **two sources (ERP and CRM)** provided as **CSV files** . The data needed to be **cleaned and integrated** into a single user-friendly model designed for analytics . The focus should be on the **latest data sets**, meaning **no historization** was required in the final model, and **documentation** of the data model was needed. This analysis shaped the subsequent design and implementation decisions .

2.  **Designing Data Architecture 🏛️**
    *   **Task:** Design the blueprint for how data will flow, integrate, and be accessed. This is a key role of a data architect .
    *   **Decision (Approach):** We chose the **Data Warehouse** approach as the primary data management system because the project requirements focused on structured data for reporting and business intelligence . We did not choose a Data Lake (more flexible for mixed data types/ML but less organized), Data Lakehouse (mix of DW/DL), or Data Mesh (decentralized) as they were not the best fit for the project's specific requirements focusing on structured data and reporting.
    *   **Decision (Structure):** Within the data warehouse approach, we chose the **Medallion Architecture** (Bronze, Silver, Gold layers). This model is easy to understand and build .
        *   **Bronze Layer:** For storing **raw, unprocessed data** as it comes from the source. Purpose: traceability and debugging by keeping untouched data . Object Type: **Tables** . Load Method: **Full Load (Truncate and Insert)** for simplicity and aligning with full extraction . Transformations: **None**, strictly no manipulation. Audience: **Data Engineers** only .
        *   **Silver Layer:** For storing **clean and standardized data** . Purpose: basic transformations to prepare data for the final layer . Object Type: **Tables**. Load Method: **Full Load (Truncate and Insert)** . Transformations: **Data Cleansing**, Standardizations, Normalizations, Deriving New Columns, Data Enrichment . Business rules pushed to the next layer . Audience: **Data Engineers, Data Analysts, Data Scientists** .
        *   **Gold Layer:** Contains **business-ready data**. Purpose: provide data for business users and analysts for reporting, analytics, etc. . Object Type: **Views** for a virtual, dynamic, and fast final layer . Load Method: **None** (views are virtual). Transformations: **Data Integrations** (combining sources), **Data Aggregations**, Applying **Business Logic/Rules** . Data Model: Designed using **Star Schema** . Audience: **Data Analysts, Business Users** .
    *   **Key Principle:** We applied the **Separation of Concerns** principle, ensuring each layer has unique, non-duplicated tasks (e.g., cleansing only in Silver, business logic only in Gold, raw landing only in Bronze). This makes the architecture organized and easier to maintain.
    *   **Documentation:** The architecture was documented visually using Draw.io, showing the layers, source systems (CSV files), consumer types (BI, Ad-hoc, ML), and key specifications for each layer.

3.  **Project Initialization 🛠️**
    *   **Task:** Set up the foundational environment for the project, including tools and basic database structure .
    *   **Decision (Tools):** We downloaded and installed **SQL Server Express** (the database server), **SQL Server Management Studio (SSMS)** (the client to interact with the database), **Draw.io** (for diagrams), and set up **GitHub** and optionally **Notion** [21]. SQL Server was chosen as per requirements [2].
    *   **Decision (Naming Conventions):** Defined early to ensure consistency across the project, preventing chaos later . We chose **Snake Case** (`all_lowercase_with_underscores`) for object names . Specific rules were defined for tables (`SourceSystem_Entity` for Bronze/Silver, `Category_Entity` with prefixes like `dim_` or `fact_` for Gold) , columns (`table_name_key` for surrogate keys, `dw_column_name` for metadata) , and stored procedures (`action_layer`) .
    *   **Decision (Repository Setup):** Created a public GitHub repository to store code, track changes, and serve as a portfolio . Established a standard folder structure (`datasets`, `documents`, `scripts`, `tests`) to organize project files [27]. Wrote a comprehensive `README.md` file to describe the project .
    *   **Decision (Database/Schema Creation):** Created the main database (`Data_Warehouse`) in SQL Server [22]. Created separate schemas within the database for each layer (`bronze`, `silver`, `gold`) to logically separate objects according to the architecture and uphold the separation of concerns principle [22, 28]. Wrote a SQL script for this, including checks to drop the database/schemas if they already exist (`IF OBJECT_ID...DROP TABLE`), adding header comments and `GO` separators [28, 29]. Committed this script to the Git repository .

4.  **Building the Bronze Layer 🧱**
    *   **Task:** Implement the data ingestion process to load raw data from source files into the Bronze layer tables .
    *   **Process:** Analyzed the source CSV files to understand their structure and metadata [31]. Created DDL scripts to define tables in the `bronze` schema that match the source file structure exactly, following the naming convention (`SourceSystem_Entity`) . Used `IF OBJECT_ID...DROP TABLE` for idempotent script execution . Wrote SQL scripts using the **`BULK INSERT`** command to load data quickly from the CSV files into the corresponding bronze tables [34]. Included options to handle the header row (`FIRSTROW`) and the delimiter (`FIELDTERMINATOR`) . Implemented a **Full Load** strategy for the bronze layer by using `TRUNCATE TABLE` before each `BULK INSERT`, ensuring the tables are refreshed completely each time the script runs .
    *   **ETL Script Engineering:** Encapsulated the loading scripts for all bronze tables into a stored procedure (`bronze.load_bronze`) . Added detailed `PRINT` messages to track the process, indicating which source system and table is being loaded [38, 39]. Implemented **Error Handling** using `BEGIN TRY...END CATCH` to catch and report any SQL errors during the load, printing error details like message and number . Added **Load Duration** measurement using `GETDATE()` and `DATEDIFF()` to track how long it takes to load each table and the entire layer, which is crucial for performance monitoring and debugging in real projects .
    *   **Data Validation:** Performed basic quality checks after loading, like counting rows to ensure all data was loaded . Manually inspected data in SSMS to check for issues like data shifting between columns .
    *   **Documentation:** Drew a **Data Flow Diagram** visually representing the flow from Source files (CRM, ERP) to the Bronze tables, showing the lineage . Committed the DDL and stored procedure scripts to the `scripts/bronze` folder in Git .

5.  **Building the Silver Layer ✨**
    *   **Task:** Implement the data cleansing and standardization logic to transform data from the Bronze layer and load it into the Silver layer tables.
    *   **Process:** **Analyzed and explored the data quality issues** present in the Bronze layer tables. This involved writing queries to check for duplicates, nulls, unwanted spaces, invalid values, inconsistent formatting, and range issues in various columns (e.g., Customer IDs, Names, Gender, Marital Status, Product Keys, Dates, Sales/Quantity/Price) . Documented the relationships between source tables by drawing an **Integration Model** using Draw.io, which helped in understanding how data from different sources could be linked . Based on the identified issues and hypothetical business rules (e.g., master source, sales calculation logic), wrote **Data Transformation** SQL scripts to clean the data . Transformations included:
        *   **Data Cleansing:** Removing duplicates (using `ROW_NUMBER()`) , removing unwanted spaces (`TRIM()`) , handling missing/invalid data (replacing nulls/empty strings with default values, using `ISNULL()`, `NULLIF()`, `CASE WHEN`) , fixing invalid date formats/values (`CAST()`, `CASE WHEN`) , fixing invalid numerical values/calculations (`CASE WHEN`, `ABS()`, `NULLIF()`) .
        *   **Data Standardization/Normalization:** Mapping abbreviated codes to full names (`CASE WHEN`, `UPPER()`, `TRIM()`) .
        *   **Deriving New Columns:** Splitting a single column (Product Key) into multiple parts (Category ID, Product Key) using string functions (`SUBSTRING()`, `REPLACE()`, `LEN()`) .
        *   **Data Enrichment:** Calculating a new end date for historical records based on the next record's start date (`LEAD()`, `DATEDIFF()`) .
        *   **Data Type Casting:** Converting data types (e.g., integer dates to DATE, Date/Time to DATE) (`CAST()`) .
    *   **Implementation:** Created DDL scripts for Silver tables by copying Bronze DDLs and replacing the schema name (`bronze.` to `silver.`) . Updated DDLs to include necessary columns for derived data (Category ID) and adjusted data types based on transformations (Dates changed from integer/datetime to DATE) . Added **Metadata Columns** (`dw_create_date`) to all Silver tables with a default value (`GETDATE()`) generated by the database, providing audit information . Wrote INSERT statements to load the transformed data from Bronze SELECT queries into the Silver tables.
    *   **ETL Script Engineering:** Consolidated the TRUNCATE and INSERT scripts for all Silver tables into a stored procedure (`silver.load_silver`) . Maintained the same engineering standards as the Bronze ETL: including `PRINT` messages, `BEGIN TRY...END CATCH` error handling, and load duration calculation for each table and the total batch. Emphasized the importance of keeping engineering standards consistent across all ETL stored procedures.
    *   **Data Validation:** After loading, performed comprehensive **quality checks on the Silver tables** using SQL queries (checking for duplicates, nulls, spaces, consistency, etc.) to ensure the cleansing process was successful. This is an iterative process, returning to coding if issues are found [59].
    *   **Documentation:** Extended the **Data Flow Diagram** to show the lineage from the Bronze layer to the Silver layer tables [84, 85]. Committed the Silver DDL and stored procedure scripts to `scripts/silver` in Git. Committed the SQL queries used for quality checks on the Silver layer to the `tests` folder.

6.  **Building the Gold Layer 🏆**
    *   **Task:** Build the final, business-ready data model optimized for analytics and reporting by integrating and aggregating data and applying business logic from the Silver layer .
    *   **Process:** **Analyzed and explored the business objects** present across the source systems (Customers, Products, Sales) based on the integrated view developed in the Silver layer analysis . Decided to build a **Star Schema** data model, comprising **Dimension** tables (descriptive data like Customers and Products) and a **Fact** table (transactional/event data like Sales). This model is commonly used and ideal for BI and reporting . We focused on creating a **Logical Data Model** .
    *   **Data Integration & Modeling:** Wrote SQL queries to build the dimensions and fact table by selecting data from the **Silver layer** .
        *   **Dimension Customer (`dim_customers`):** Integrated customer data from three Silver tables (CRM Customer Info, ERP Customer, ERP Location) using **`LEFT JOIN`**, starting from the master table (CRM Customer Info) to ensure all customers were included . Applied integration logic for conflicting data points (Gender) by prioritizing the master source (CRM). Gave friendly, business-aligned column names following the naming convention (e.g., `customer_id`, `first_name`, `country`) . Ordered columns logically [90]. Generated a **Surrogate Key** (`customer_key`) using `ROW_NUMBER()` as the primary key for the dimension, providing a stable, system-generated identifier independent of source keys .
        *   **Dimension Product (`dim_products`):** Integrated product data from two Silver tables (CRM Product Info, ERP PX Catalog) using **`LEFT JOIN`** . Filtered data to include only the **latest product information** (`end_date IS NULL`) as per the project requirement to focus on current data [100]. Gave friendly, business-aligned column names (e.g., `product_name`, `category`, `cost`) . Ordered columns logically [91, 101]. Generated a **Surrogate Key** (`product_key`) using `ROW_NUMBER()` .
        *   **Fact Sales (`fact_sales`):** Based on the Silver CRM Sales Details table . Performed **Data Lookup** by joining the Fact query with the newly created Gold Dimensions (`dim_customers`, `dim_products`) to retrieve the **Surrogate Keys** (`customer_key`, `product_key`) . Replaced the original source system keys with these surrogate keys, as the fact table should reference dimensions via surrogate keys in a star schema . Gave friendly, business-aligned column names (e.g., `order_number`, `sales_amount`) . Ordered columns logically: Surrogate Keys first, then Dates, then Measures .
    *   **Implementation:** Created the Gold layer objects as **Views** (`CREATE VIEW gold.dim_customers AS...`, `CREATE VIEW gold.dim_products AS...`, `CREATE VIEW gold.fact_sales AS...`) . Views are preferred for the Gold layer as they are virtual and always reflect the latest data in the underlying Silver tables without needing a separate load process .
    *   **Data Validation:** Checked the quality of the Gold views by performing simple SELECTs . Crucially, verified the **integrity and joinability of the data model** by performing `LEFT JOIN` tests between the Fact table and Dimensions to ensure all records in the Fact could be linked to corresponding Dimension records . Also checked the uniqueness of the Surrogate Keys .
    *   **Documentation:** Created a **Logical Data Model diagram** using Draw.io, visually representing the Star Schema (Fact in the middle, Dimensions around) with tables listing columns (PK/FK), and showing the **one-to-many relationships** between dimensions and the fact. Added textual descriptions and business rules (like the sales calculation formula) to the diagram for clarity . Created a **Data Catalog** document describing each Gold layer view (purpose, columns, descriptions, and examples for columns) to help end-users understand and consume the data model . Extended the **Data Flow Diagram** to show the lineage from the Silver layer to the Gold layer views, completing the data lineage from sources to the final consumption layer. Committed the Gold views script to `scripts/gold` in Git . Committed the SQL queries used for Gold quality checks to the `tests` folder . Uploaded documentation diagrams and the data catalog document to the `documents` folder in Git and updated the README to link to key parts .

## Project Structure 📁

This repository is organized into the following folders:

*   `datasets`: Contains the raw source data files (CSV).
*   `documents`: Includes project documentation such as data architecture diagrams, data flow diagrams, data integration diagrams, data model diagrams, and the data catalog .
*   `scripts`: Houses the SQL scripts for database initialization and the ETL processes for the Bronze, Silver, and Gold layers .
    *   `scripts/init_database.sql`: Script to create the database and schemas .
    *   `scripts/bronze/ddl_bronze.sql`: DDL scripts for creating Bronze layer tables .
    *   `scripts/bronze/proc_load_bronze.sql`: Stored procedure for loading the Bronze layer.
    *   `scripts/silver/ddl_silver.sql`: DDL scripts for creating Silver layer tables .
    *   `scripts/silver/proc_load_silver.sql`: Stored procedure for loading the Silver layer with cleansed data .
    *   `scripts/gold/ddl_gold.sql`: Scripts for creating Gold layer views (Dimensions and Fact) .
*   `tests`: Contains SQL scripts for data quality checks on the Silver and Gold layers .
    *   `tests/quality_checks_silver.sql`: Queries to validate data quality in the Silver layer .
    *   `tests/quality_checks_gold.sql`: Queries to validate data integrity and relationships in the Gold layer .

## How to Use 🛠️

1.  Download and install **SQL Server Express** and **SSMS** .
2.  Clone this repository to your local machine.
3.  Place the downloaded project dataset files (CRM and ERP folders with CSVs) into the `datasets` folder of the cloned repository .
4.  Open SSMS and connect to your local SQL Server instance.
5.  Open and execute the `scripts/init_database.sql` script to create the `Data_Warehouse` database and the `bronze`, `silver`, and `gold` schemas .
6.  Open and execute the `scripts/bronze/ddl_bronze.sql` script to create the tables in the `bronze` schema .
7.  Open and execute the `scripts/bronze/proc_load_bronze.sql` script to create the stored procedure for loading the bronze layer. **Make sure to update the file paths within the script** to match the location of your `datasets` folder on your local machine.
8.  Execute the bronze load stored procedure: `EXEC bronze.load_bronze;` .
9.  Open and execute the `scripts/silver/ddl_silver.sql` script to create the tables in the `silver` schema .
10. Open and execute the `scripts/silver/proc_load_silver.sql` script to create the stored procedure for loading the silver layer.
11. Execute the silver load stored procedure: `EXEC silver.load_silver;` .
12. Open and execute the `scripts/gold/ddl_gold.sql` script to create the views in the `gold` schema .
13. You can now explore the views in the `gold` schema and use them for reporting and analysis! ✨

Feel free to explore the documentation in the `documents` folder and the quality check scripts in the `tests` folder to gain a deeper understanding of the project.

## Showcase Your Skills! 🌟

Having built this project demonstrates practical data engineering skills, including:

*   Designing data architecture (Medallion) .
*   Implementing ETL/ELT processes using SQL .
*   Building multi-layered data warehouses .
*   Performing comprehensive data cleansing and transformation .
*   Integrating data from multiple sources .
*   Designing and implementing a data model (Star Schema) .
*   Creating and using surrogate keys .
*   Implementing ETL best practices (stored procedures, logging, error handling, performance measurement) .
*   Documenting technical processes and data models (diagrams, data catalog) .
*   Using Git for version control and collaboration .

Feel free to take, modify, and share this project. Showcase your skills on platforms like LinkedIn! 

## License 📝

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details .

## About Me 👋

Data Enthusiast

## Support the Creator 🙏

If you found this project helpful and appreciate the tutorial it's based on, consider supporting the original creator!
