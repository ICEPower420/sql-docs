---
title: "catalog.cleanup_server_execution_keys"
description: "catalog.cleanup_server_execution_keys"
author: chugugrace
ms.author: chugu
ms.date: "03/03/2017"
ms.service: sql
ms.subservice: integration-services
ms.topic: conceptual
---
# catalog.cleanup_server_execution_keys 

[!INCLUDE[sqlserver-ssis](../../includes/applies-to-version/sqlserver-ssis.md)]


[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

  Drops certificates and symmetric keys from the SSISDB database.  
  
## Syntax  
  
```sql
catalog.cleanup_server_execution_keys [ @cleanup_flag = ] cleanup_flag ,  
[ @delete_batch_size = ] delete_batch_size  
```  
  
## Arguments  
 [ @cleanup_flag = ] *cleanup_flag*  
 Indicates whether execution level (1) or project level (2) certificates and symmetric keys are to be dropped.  
  
 Use execution level (1) only when the SERVER_OPERATION_ENCRYPTION_LEVEL is set to PER_EXECUTION (1).  
  
 Use project level (2) only when the SERVER_OPERATION_ENCRYPTION_LEVEL is set to PER_PROJECT (2). Certificates and symmetric keys are dropped only for projects that have been deleted and for which the operation logs have been cleaned.  
  
 [ @delete_batch_size = ] *delete_batch_size*  
 The number of keys and certificates to be dropped. The default value is 1000.  
  
## Return Code Values  
 0 for success and 1 for failure.  
  
## Result Sets  
 None.  
  
## Permissions  
 This stored procedure requires one of the following permissions:  
  
-   READ and EXECUTE permissions on the project and, if applicable, READ permissions on the referenced environment.  
  
-   Membership in the **ssis_admin** database role.  
  
-   Membership to the **sysadmin** server role.  
  
## Errors and Warnings  
 This stored procedure raises errors in the following scenarios:  
  
-   There are one or more active operations in the SSISDB database.  
  
-   The SSISDB database is not in single user mode.  
  
## Remarks  
 SQL Server 2012 Service Pack 2 added the SERVER_OPERATION_ENCRYPTION_LEVEL property to the **internal.catalog_properties** table. This property has two possible values:  
  
-   **PER_EXECUTION (1)** - The certificate and symmetric key used for protecting sensitive execution parameters and execution logs are created for each execution. This is the default value. You may run into performance issues (deadlocks, failed maintenance jobs etc...) in a production environment because certificate/keys are generated for each execution. However, this setting provides a higher level of security than the other value (2).  
  
-   **PER_PROJECT (2)** - The certificate and symmetric key used for protecting sensitive parameters are created for each project. This gives you a better performance than the PER_EXECUTION level because the key and certificate are generated once for a project rather than for each execution.  
  
 You have to run the [catalog.cleanup_server_log](../../integration-services/system-stored-procedures/catalog-cleanup-server-log.md) stored procedure before you can change the SERVER_OPERATION_ENCRYPTION_LEVEL from 1 to 2 (or) from 2 to 1. Before running this stored procedure, do the following things:  
  
1.  Ensure that the value of the property OPERATION_CLEANUP_ENABLED is set to TRUE in the [catalog.catalog_properties &#40;SSISDB Database&#41;](../../integration-services/system-views/catalog-catalog-properties-ssisdb-database.md) table.  
  
2.  Set the Integration Services database (SSISDB) to single-user mode. In SQL Server Management Studio, launch Database Properties dialog box for SSISDB, switch to the Options tab, and set the Restrict Access property to single-user mode (SINGLE_USER). After you run the cleanup_server_log stored procedure, set the property value back to the original value.  
  
3.  Run the stored procedure [catalog.cleanup_server_log](../../integration-services/system-stored-procedures/catalog-cleanup-server-log.md).  
  
4.  Now, go ahead and change the value for the SERVER_OPERATION_ENCRYPTION_LEVEL property in the [catalog.catalog_properties &#40;SSISDB Database&#41;](../../integration-services/system-views/catalog-catalog-properties-ssisdb-database.md) table.  
  
5.  Run the stored procedure [catalog.cleanup_server_execution_keys](../../integration-services/system-stored-procedures/catalog-cleanup-server-execution-keys.md) to clean up certificates keys from the SSISDB database. Dropping certificates and keys from the SSISDB database may take a long time, so it should be run periodically during off-peak times.  
  
     You can specify the scope or level (execution vs. project) and number of keys to be deleted. The default batch size for deletion is 1000. When you set the level to 2, the keys and certificates are deleted only if the associated projects have been deleted.  
  
 For more info, see the following Knowledge Base article. [FIX: Performance issues when you use SSISDB as your deployment store in SQL Server 2012](https://support.microsoft.com/kb/2972285)  
  
## Example  
 The following example calls the cleanup_server_execution_keys stored procedure.  
  
```sql  
USE [SSISDB]  
GO  
  
DECLARE@return_value int  
  
EXEC@return_value = [internal].[cleanup_server_execution_keys]  
@cleanup_flag = 1,  
@delete_batch_size = 500  
  
SELECT'Return Value' = @return_value  
  
GO  
```  
  
  
