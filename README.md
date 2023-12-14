#  Data-Anonymization
## Anonymization Procedures for SQL Server Databases

You're a DBA, sysadmin,devops engineer or developer who manages Microsoft SQL Servers. You are asked to provide Production copies of databases to development teams for new production enhancements and builds, bug-fixes, performance testing etc. However the Cyber Security regulations in the company require you to strip all the PII and sensitive client data before you can provide the Production copy. You have no idea what data in the database qualifies as PII and client sensitive. This set of procedures helps you to **get started** with anonymizing and obfuscating this data without requiring you to necessarily look at each column of each table and the type of data it stores.

This process as a pre-requisite requires Microsoft's Data Discovery and Classifications feature to be enabled and implemented.

__https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-data-discovery-and-classification?view=sql-server-ver16&tabs=sql-powelshell_

_https://learn.microsoft.com/en-us/azure/azure-sql/database/data-discovery-and-classification-overview?view=azuresql__

Applying data classifications to the database stores the information in metadata tables and allows it to be retrieved which this process leverages to dynamically anonymize the identified tables and their columns.
