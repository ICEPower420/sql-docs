---
title: "sp_add_log_shipping_secondary_primary (Transact-SQL)"
description: "Sets up the primary information, adds local and remote monitor links, and creates copy and restore jobs on the secondary server."
author: MashaMSFT
ms.author: mathoma
ms.reviewer: randolphwest
ms.date: 06/02/2023
ms.service: sql
ms.subservice: system-objects
ms.topic: "reference"
f1_keywords:
  - "sp_add_log_shipping_secondary_primary_TSQL"
  - "sp_add_log_shipping_secondary_primary"
helpviewer_keywords:
  - "sp_add_log_shipping_secondary_primary"
dev_langs:
  - "TSQL"
---
# sp_add_log_shipping_secondary_primary (Transact-SQL)

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

Sets up the primary information, adds local and remote monitor links, and creates copy and restore jobs on the secondary server for the specified primary database.

:::image type="icon" source="../../includes/media/topic-link-icon.svg" border="false"::: [Transact-SQL syntax conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)

## Syntax

```syntaxsql
sp_add_log_shipping_secondary_primary
    [ @primary_server = ] 'primary_server'
    , [ @primary_database = ] 'primary_database'
    , [ @backup_source_directory = ] N'backup_source_directory'
    , [ @backup_destination_directory = ] N'backup_destination_directory'
    , [ @copy_job_name = ] 'copy_job_name'
    , [ @restore_job_name = ] 'restore_job_name'
    [ , [ @file_retention_period = ] 'file_retention_period' ]
    [ , [ @monitor_server = ] 'monitor_server' ]
    [ , [ @monitor_server_security_mode = ] 'monitor_server_security_mode' ]
    [ , [ @monitor_server_login = ] 'monitor_server_login' ]
    [ , [ @monitor_server_password = ] 'monitor_server_password' ]
    [ , [ @copy_job_id = ] 'copy_job_id' OUTPUT ]
    [ , [ @restore_job_id = ] 'restore_job_id' OUTPUT ]
    [ , [ @secondary_id = ] 'secondary_id' OUTPUT ]
[ ; ]
```

## Arguments

#### [ @primary_server = ] '*primary_server*'

The name of the primary instance of the [!INCLUDE [ssDEnoversion](../../includes/ssdenoversion-md.md)] in the log shipping configuration. *@primary_server* is **sysname** and can't be NULL.

#### [ @primary_database = ] '*primary_database*'

The name of the database on the primary server. *@primary_database* is **sysname**, with no default.

#### [ @backup_source_directory = ] N'*backup_source_directory*'

The directory where transaction log backup files from the primary server are stored. *@backup_source_directory* is **nvarchar(500)** and can't be NULL.

#### [ @backup_destination_directory = ] N'*backup_destination_directory*'

The directory on the secondary server where backup files are copied to. *@backup_destination_directory* is **nvarchar(500)** and can't be NULL.

#### [ @copy_job_name = ] '*copy_job_name*'

The name to use for the [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] Agent job being created to copy transaction log backups to the secondary server. *copy_job_name* is **sysname** and can't be NULL.

#### [ @restore_job_name = ] '*restore_job_name*'

The name of the [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] Agent job on the secondary server that restores the backups to the secondary database. *restore_job_name* is **sysname** and can't be NULL.

#### [ @file_retention_period = ] '*file_retention_period*'

The length of time, in minutes, that a backup file is retained on the secondary server in the path specified by the @backup_destination_directory parameter before being deleted. *@history_retention_period* is **int**, with a default of NULL. A value of 14420 is used if none is specified.

#### [ @monitor_server = ] '*monitor_server*'

The name of the monitor server. *@monitor_server* is **sysname**, with no default, and can't be NULL.

#### [ @monitor_server_security_mode = ] '*monitor_server_security_mode*'

The security mode used to connect to the monitor server.

- `1`: Windows Authentication
- `0`: [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] authentication

*@monitor_server_security_mode* is **bit** and can't be NULL.

#### [ @monitor_server_login = ] '*monitor_server_login*'

The username of the account used to access the monitor server.

#### [ @monitor_server_password = ] '*monitor_server_password*'

The password of the account used to access the monitor server.

#### [ @copy_job_id = ] '*copy_job_id*' OUTPUT

The ID associated with the copy job on the secondary server. *@copy_job_id* is **uniqueidentifier** and can't be NULL.

#### [ @restore_job_id = ] '*restore_job_id*' OUTPUT

The ID associated with the restore job on the secondary server. *@restore_job_id* is **uniqueidentifier** and can't be NULL.

#### [ @secondary_id = ] '*secondary_id*' OUTPUT

The ID for the secondary server in the log shipping configuration. *@secondary_id* is **uniqueidentifier** and can't be NULL.

## Return code values

`0` (success) or `1` (failure).

## Result sets

None.

## Remarks

`sp_add_log_shipping_secondary_primary` must be run from the `master` database on the secondary server. This stored procedure does the following:

1. Generates a secondary ID for the specified primary server and primary database.

1. Does the following:

   1. Adds an entry for the secondary ID in `log_shipping_secondary` using the supplied arguments.
   1. Creates a copy job for the secondary ID that is disabled.
   1. Sets the copy job ID in the `log_shipping_secondary` entry to the job ID of the copy job.
   1. Creates a restore job for the secondary ID that is disabled.
   1. Set the restore job ID in the `log_shipping_secondary` entry to the job ID of the restore job.

## Permissions

Only members of the **sysadmin** fixed server role can run this procedure.

## Examples

This example illustrates using the `sp_add_log_shipping_secondary_primary` stored procedure to set up information for the primary database [!INCLUDE [ssSampleDBobject](../../includes/sssampledbobject-md.md)] on the secondary server.

```sql
EXEC master.dbo.sp_add_log_shipping_secondary_primary @primary_server = N'TRIBECA',
    @primary_database = N'AdventureWorks2022',
    @backup_source_directory = N'\\tribeca\LogShipping',
    @backup_destination_directory = N'',
    @copy_job_name = N'',
    @restore_job_name = N'',
    @file_retention_period = 1440,
    @monitor_server = N'ROCKAWAY',
    @monitor_server_security_mode = 1,
    @copy_job_id = @LS_Secondary__CopyJobId OUTPUT,
    @restore_job_id = @LS_Secondary__RestoreJobId OUTPUT,
    @secondary_id = @LS_Secondary__SecondaryId OUTPUT;
GO
```

## See also

- [About Log Shipping (SQL Server)](../../database-engine/log-shipping/about-log-shipping-sql-server.md)
- [System stored procedures (Transact-SQL)](system-stored-procedures-transact-sql.md)
