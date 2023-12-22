#  Data-Anonymization
## Anonymization Procedures for SQL Server Databases

You're a DBA, sysadmin,devops engineer or developer who manages Microsoft SQL Servers. You are asked to provide Production copies of databases to development teams for new production enhancements and builds, bug-fixes, performance testing etc. However the Cyber Security regulations in the company require you to strip all the PII and sensitive client data before you can provide the Production copy. You have no idea what data in the database qualifies as PII and client sensitive. This set of procedures helps you to **get started** with anonymizing and obfuscating this data without requiring you to necessarily look at each column of each table and the type of data it stores.

This process as a **pre-requisite** requires Microsoft's Data Discovery and Classifications feature to be enabled and implemented.

https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-data-discovery-and-classification?view=sql-server-ver16&tabs=sql-powelshell

https://learn.microsoft.com/en-us/azure/azure-sql/database/data-discovery-and-classification-overview?view=azuresql

Azure also provides the option to customize the Data classification Labels and Information types via a feature called SQL information protection policy.

SQL information protection's data discovery and classification mechanism provides advanced capabilities for discovering, classifying, labeling, and reporting the sensitive data in your databases. You can customize the policy, according to your organization's needs. This is available under Microsoft defender for cloud and when selected, it applies at the tenant level.

Applying data classifications to the database stores the information in metadata tables and allows it to be retrieved which this process leverages to dynamically anonymize the identified tables and their columns.

**The Anonymization Procedures run on:**

- SQL Server 2016, 2017, 2019, 2022
* Azure SQL DB (with database compatibility =130 or higher)
+ Azure SQL Managed Instance

## Steps to install the data anonymization procedures

- 1.Enable and implement Microsoft's Data Discovery and Classifications feature
- 2.Execute the script **InstallALL_usp_obfuscate.sql** in your target database (example: copy of Production database) which needs to be anonymized
- 3.Execute the below procedures in sequence
-   EXEC [MASK].[usp_obfuscate_00]
    EXEC [MASK].[usp_obfuscate_01] @ExclusionTableList='DUMMY_DATA', @ExclusionSchemaList='MASK'
    EXEC [MASK].[usp_obfuscate_02] @TableName=NULL, @SchemaName=NULL
    EXEC [MASK].[usp_obfuscate_03] 
