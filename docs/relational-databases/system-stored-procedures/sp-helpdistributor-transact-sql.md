---
title: "sp_helpdistributor (Transact-SQL)"
description: "Lists information about the Distributor, distribution database, working directory, and SQL Server Agent user account."
author: markingmyname
ms.author: maghan
ms.reviewer: randolphwest
ms.date: 05/31/2023
ms.service: sql
ms.subservice: replication
ms.topic: "reference"
f1_keywords:
  - "sp_helpdistributor_TSQL"
  - "sp_helpdistributor"
helpviewer_keywords:
  - "sp_helpdistributor"
dev_langs:
  - "TSQL"
---
# sp_helpdistributor (Transact-SQL)

[!INCLUDE [SQL Server SQL MI](../../includes/applies-to-version/sql-asdbmi.md)]

Lists information about the Distributor, distribution database, working directory, and [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] Agent user account. This stored procedure is executed at the Publisher on the publication database or any database.

:::image type="icon" source="../../includes/media/topic-link-icon.svg" border="false"::: [Transact-SQL syntax conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)

## Syntax

```syntaxsql
sp_helpdistributor [ [ @distributor = ] 'distributor' OUTPUT ]
    [ , [ @distribdb = ] 'distribdb' OUTPUT ]
    [ , [ @directory = ] 'directory' OUTPUT ]
    [ , [ @account = ] 'account' OUTPUT ]
    [ , [ @min_distretention = ] min_distretention OUTPUT ]
    [ , [ @max_distretention = ] max_distretention OUTPUT ]
    [ , [ @history_retention = ] history_retention OUTPUT ]
    [ , [ @history_cleanupagent = ] 'history_cleanupagent' OUTPUT ]
    [ , [ @distrib_cleanupagent = ] 'distrib_cleanupagent' OUTPUT ]
    [ , [ @publisher = ] 'publisher' ]
    [ , [ @local = ] 'local' ]
    [ , [ @rpcsrvname = ] 'rpcsrvname' OUTPUT ]
    [ , [ @publisher_type = ] 'publisher_type' OUTPUT ]
[ ; ]
```

## Arguments

#### [ @distributor = ] '*distributor*' OUTPUT

The name of the Distributor. *@distributor* is **sysname**, with a default of `%`, which is the only value that returns a result set.

#### [ @distribdb = ] '*distribdb*' OUTPUT

The name of the distribution database. *@distribdb* is **sysname**, with a default of `%`, which is the only value that returns a result set.

#### [ @directory = ] '*directory*' OUTPUT

The working directory. *@directory* is **nvarchar(255)**, with a default of `%`, which is the only value that returns a result set.

#### [ @account = ] '*account*' OUTPUT

The [!INCLUDE [msCoName](../../includes/msconame-md.md)] Windows user account. *@account* is **nvarchar(255)**, with a default of `%`, which is the only value that returns a result set.

#### [ @min_distretention = ] *min_distretention* OUTPUT

The minimum distribution retention period, in hours. *@min_distretention* is **int**, with a default of `-1`.

#### [ @max_distretention = ] *max_distretention* OUTPUT

The maximum distribution retention period, in hours. *@max_distretention* is **int**, with a default of `-1`.

#### [ @history_retention = ] *history_retention* OUTPUT

The history retention period, in hours. *@history_retention* is **int**, with a default of `-1`.

#### [ @history_cleanupagent = ] '*history_cleanupagent*' OUTPUT

The name of the history cleanup agent. *@history_cleanupagent* is **nvarchar(100)**, with a default of `%`, which is the only value that returns a result set.

#### [ @distrib_cleanupagent = ] '*distrib_cleanupagent*' OUTPUT

The name of the distribution cleanup agent. *@distrib_cleanupagent* is **nvarchar(100)**, with a default of `%`, which is the only value that returns a result set.

#### [ @publisher = ] '*publisher*'

The name of the Publisher. *@publisher* is **sysname**, with a default of NULL.

#### [ @local = ] '*local*'

Whether [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] should get local server values. *@local* is **nvarchar(5)**, with a default of NULL.

#### [ @rpcsrvname = ] '*rpcsrvname*' OUTPUT

The name of the server that issues remote procedure calls. *@rpcsrvname* is **sysname**, with a default of `%`, which is the only value that returns a result set.

#### [ @publisher_type = ] '*publisher_type*' OUTPUT

The publisher type of the Publisher. *@publisher_type* is **sysname**, with a default of `%`, which is the only value that returns a result set.

## Result sets

| Column name | Data type | Description |
| --- | --- | --- |
| `distributor` | **sysname** | Name of the Distributor. |
| `distribution database` | **sysname** | Name of the distribution database. |
| `directory` | **nvarchar(255)** | Name of the working directory. |
| `account` | **nvarchar(255)** | Name of the Windows user account. |
| `min distrib retention` | **int** | Minimum distribution retention period. |
| `max distrib retention` | **int** | Maximum distribution retention period. |
| `history retention` | **int** | History retention period. |
| `history cleanup agent` | **nvarchar(100)** | Name of the History Cleanup Agent. |
| `distribution cleanup agent` | **nvarchar(100)** | Name of the Distribution Cleanup Agent. |
| `rpc server name` | **sysname** | Name of the remote or local Distributor. |
| `rpc login name` | **sysname** | Login used for remote procedure calls to the remote Distributor. |
| `publisher type` | **sysname** | Type of Publisher; can be one of the following:<br /><br />- MSSQLSERVER<br />- ORACLE<br />- ORACLE GATEWAY |

## Return code values

`0` (success) or `1` (failure).

## Remarks

`sp_helpdistributor` is used in all types of replication.

If one or more output parameters are specified when executing `sp_helpdistributor`, all output parameters set to NULL are assigned values on exit and no result set is returned. If no output parameters are specified, a result set is returned.

## Permissions

The following result set columns or output parameters are returned to members of the **sysadmin** fixed server role at the Publisher and the **db_owner** fixed database role on the publication database:

| Result set column | Output parameter |
| --- | --- |
| `account` | *@account* |
| `min distrib retention` | *@min_distretention* |
| `max distrib retention` | *@max_distretention* |
| `history retention` | *@history_retention* |
| `history cleanup agent` | *@history_cleanupagent* |
| `distribution cleanup agent` | *@distrib_cleanupagent* |
| `rpc login name` | none |

The following result set column is returned to users in the publication access list for a publication at the Distributor:

- directory

The following result set columns are returned to all users.

| Result set column | Output parameter |
| --- | --- |
| `distributor` | *@distributor* |
| `distribution database` | *@distribdb* |
| `rpc server name` | *@rpcsrvname* |
| `publisher type` | *@publisher_type* |

## See also

- [View and Modify Distributor and Publisher Properties](../../relational-databases/replication/view-and-modify-distributor-and-publisher-properties.md)
- [sp_adddistpublisher (Transact-SQL)](sp-adddistpublisher-transact-sql.md)
- [sp_dropdistpublisher (Transact-SQL)](sp-dropdistpublisher-transact-sql.md)
