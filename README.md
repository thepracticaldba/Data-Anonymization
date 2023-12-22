#  Data-Anonymization
## Anonymization Procedures for SQL Server Databases

You're a DBA, sysadmin,devops engineer or developer who manages Microsoft SQL Servers. You are asked to provide Production copies of databases to development teams for new production enhancements and builds, bug-fixes, performance testing etc. However the Cyber Security regulations in the company require you to strip all the PII and sensitive client data before you can provide the Production copy. You have no idea what data in the database qualifies as PII and client sensitive. This set of procedures helps you to **get started** with anonymizing and obfuscating this data without requiring you to necessarily look at each column of each table and the type of data it stores.

## Overview of the Process
This process as a **pre-requisite** requires Microsoft's Data Discovery and Classifications feature to be enabled and implemented.

https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-data-discovery-and-classification?view=sql-server-ver16&tabs=sql-powelshell

https://learn.microsoft.com/en-us/azure/azure-sql/database/data-discovery-and-classification-overview?view=azuresql

Azure also provides the option to customize the Data classification Labels and Information types via a feature called SQL information protection policy.

SQL information protection's data discovery and classification mechanism provides advanced capabilities for discovering, classifying, labeling, and reporting the sensitive data in your databases. You can customize the policy, according to your organization's needs. This is available under Microsoft defender for cloud and when selected, it applies at the tenant level.

Applying data classifications to the database stores the information in metadata tables and allows it to be retrieved which this process leverages to dynamically anonymize the identified tables and their columns. The script **InstallALL_usp_obfuscate.sql** installs a set of database objects (tables, views, stored procedures and scalar functions) under a new schema called MASK. One of the tables created **MASK.DUMMY_DATA** is used to store as the name suggests dummy or fake PII and sensitive data which is what the target database is masked with. I have used https://www.mockaroo.com/ to generate the dummy data. There are options incorporated in the process to exclude specific tables or columns by making entries in table **MASK.DataMaskingExclusionList**. Scalar functions are used to get random values from the specific columns in MASK.DUMMY_DATA table. Due to the dynamic nature of this process, the anonymization procedure **MASK.usp_obfuscate_00** excludes PK, FK and Unique key columns and also temporal and temporal history tables from this process. It is left to the user to customize this procedure to include these columns based on their specific database schema. For further details on the anonymization process refer to the Detailed Description section. 

**The Anonymization Procedures run on:**

- SQL Server 2016, 2017, 2019, 2022
* Azure SQL DB (with database compatibility =130 or higher)
+ Azure SQL Managed Instance

## Steps to install the data anonymization procedures

- Enable and implement Microsoft's Data Discovery and Classifications feature
- Execute the script **InstallALL_usp_obfuscate.sql** in your target database (example: copy of Production database) which needs to be anonymized
- Execute the below procedures in sequence
  
  - `EXEC [MASK].[usp_obfuscate_00]`
  - `EXEC [MASK].[usp_obfuscate_01] @ExclusionTableList='DUMMY_DATA', @ExclusionSchemaList='MASK'`
  - `EXEC [MASK].[usp_obfuscate_02] @TableName=NULL, @SchemaName=NULL`
  - `EXEC [MASK].[usp_obfuscate_03]`

## Detailed Description

The script InstallALL_usp_obfuscate.sql installs a set of database objects (tables, views, stored procedures and scalar functions) under a new schema called MASK.

**Tables:**

- DUMMY_DATA

  (is used to store as the name suggests dummy or fake PII and sensitive data which is what the target database is masked with. I have used https://www.mockaroo.com/ to generate 5k rows of 
   dummy data.)
- DataMaskingExclusionList

  (allows us to exclude specific tables or columns by inserting entries)
- DataMaskingScripts

  (stores the update code snippets and filter conditions for identified tables to be anonymized)
- DataObfuscateList

  (is the logging table which has the complete update code alongwith rowcounts, start and finish times, status and error messages if any)

**Views:**

These views are used to generate random numbers which are then used to get random dummy data values from table MASK.DUMMY_DATA
- vw_getNewID
- vw_getRandID

**Scalar UDFs:**

These scalar UDFs are used to get unique random values from the MASK.DUMMY_DATA table corresponding to the length of the target column data type

- udf_getRandIPAddr
- udf_getRandUsername
- udf_getRandpwd
- udf_getRandmarks
- udf_getRandBankNum
- udf_getRandExpiryDate
- udf_getRandFirstName
- udf_getRandLastName
- udf_getRandURL
- udf_getRandPhoneNum
- udf_getRandMobNum
- udf_getRandCCNum
- udf_getRandCCType
- udf_getRandAmount
- udf_getRandAccID
- udf_getRandBankID
- udf_getRandSSN
- udf_getRandHealthRecord
- udf_getRandHealthNum
- udf_getRandPhoneNum_INT
- udf_getRandMobileNum_INT
- udf_getRandExpDate_Quantity
- udf_getRandEmail
- udf_getRandTown
- udf_getRandState
- udf_getRandCountry
- udf_getRandAddress1
- udf_getRandAddress2
- udf_getRandZip
- udf_getRandZipInt
- udf_getRandDOB

**Stored Procedures:**

- **usp_obfuscate_00**

- is the brains of the entire process
- retrieves the classified columns from sys.sensitivity_classifications or sys.extended_properties (in case of SQL Server versions below 2019)
- excludes Primary, foreign and Unique key columns alongwith temporal and history tables.
- excludes columns/tables specified in [MASK].[DataMaskingExclusionList]
- excludes data types outside of ('tinyint', 'smallint', 'int', 'bigint','float','decimal', 'numeric','char', 'nchar', 'varchar', 'nvarchar', 'text', 'ntext', 'date', 'datetime', 'datetime2', 
  'smalldatetime', 'datetimeoffset'),example: xml, varbinary sql_variant etc.
  
- **usp_obfuscate_01**

- populates the logging table [MASK].[DataObfuscateList] with update scripts and filter conditions
- adds in the rowcount of the tables to be anonymized
- excludes any tables with rowcount=0
  
- **usp_obfuscate_02**

- performs the anonymization by executing the update statements
- logs the progress in the table [MASK].[DataObfuscateList]
- allows to be run for specific tables as well (or in case of failures)
  
- **usp_obfuscate_03**

- this is the final procedure used to check the final status of the process (completed successfully or failed and needs to be re-run)
     
