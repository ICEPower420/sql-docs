---
title: "Resource Governor Resource Pool"
description: SQL Server Resource Governor specifies limits on the amount of CPU, physical IO, and memory that incoming application requests can use within the resource pool.
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: jopilov
ms.date: 03/31/2023
ms.service: sql
ms.subservice: performance
ms.topic: conceptual
helpviewer_keywords:
  - "Resource Governor, resource pool"
  - "resource pool [SQL Server], overview"
  - "resource pool [SQL Server]"
---

# Resource Governor Resource Pool

[!INCLUDE [SQL Server SQL MI](../../includes/applies-to-version/sql-asdbmi.md)]

  In the [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] Resource Governor, a resource pool represents a subset of the physical resources of an instance of the [!INCLUDE[ssDE](../../includes/ssde-md.md)]. Resource Governor enables you to specify limits on the amount of CPU, physical IO, and memory that incoming application requests can use within the resource pool. Each resource pool can contain one or more workload groups. When a session is started, the Resource Governor classifier assigns the session to a specific workload group, and the session must run using the resources assigned to the workload group.

## Resource Pool Concepts

A resource pool, or pool, represents the physical resources of the server. You can think of a pool as a virtual [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] instance inside of a [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] instance. A pool has two parts. One part doesn't overlap with other pools, which enables minimum resource reservation. The other part is shared with other pools, which supports maximum possible resource consumption. The pool resources are defined by specifying one or more of the following settings for each resource (CPU, memory, and physical IO):

### MIN_CPU_PERCENT and MAX_CPU_PERCENT

These settings are the minimum and maximum guaranteed average CPU bandwidth for all requests in the resource pool when there's CPU contention. You can use these settings to establish predictable CPU resource usage for multiple workloads that is based on the needs of each workload. For example, assume the Sales and Marketing departments in a company share the same database. The Sales department has a CPU-intensive workload with high-priority queries. The Marketing department also has a CPU-intensive workload, but has lower-priority queries. By creating a separate resource pool for each department, you can assign a *minimum* CPU percentage of 70 for the Sales resource pool and a *maximum* CPU percentage of 30 for the Marketing resource pool. This configuration ensures that the Sales workload receives the CPU resources it requires and the Marketing workload is isolated from the CPU demands of the Sales workload. The maximum CPU percentage is an opportunistic maximum. If there's available CPU capacity the worker threads use all of it, up to 100 percent. The maximum value only applies when there's contention for CPU resources. In this example, if the Sales workload is switched off, the Marketing workload can use 100 percent of the CPU if needed.

### CAP_CPU_PERCENT

The CAP_CPU_PERCENT setting is a hard cap limit on the CPU bandwidth for all requests in the resource pool. Workloads associated with the pool can use CPU capacity above the value of MAX_CPU_PERCENT if it's available, but not above the value of CAP_CPU_PERCENT. Based on the example in the previous section, lets assume that the Marketing department is being charged for their resource usage. They want predictable billing and don't want to pay for more than 30 percent of the CPU. This goal can be accomplished by setting the CAP_CPU_PERCENT to 30 for the Marketing resource pool.

### MIN_MEMORY_PERCENT and MAX_MEMORY_PERCENT

These settings are the minimum and maximum amount of memory reserved for the resource pool that can't be shared with other resource pools. For databases without memory-optimized tables, the memory referenced is query execution grant memory, not buffer pool memory (data and index pages). For more information on query execution memory grants, see [Memory Grant Considerations](../memory-management-architecture-guide.md#memory-grant-considerations). Setting a minimum memory value for a pool ensures that the percentage of memory specified is available for any requests that runs in this resource pool. This setting is different from MIN_CPU_PERCENT, because in this case memory may remain in the given resource pool even when the pool doesn't have any requests in the workload groups belonging to this pool. Therefore, be careful when using this setting as this memory is unavailable for use by any other pool, even when there are no active requests. Setting a maximum memory value for a pool means that when requests are running in this pool, they never get more than this percentage of overall memory.

In order to govern memory for memory-optimized tables with Resource Governor, you should bind the database to a separate resource pool. For more information, see [Bind a Database with Memory-Optimized Tables to a Resource Pool](../in-memory-oltp/bind-a-database-with-memory-optimized-tables-to-a-resource-pool.md).

### AFFINITY

This setting lets you affinitize a resource pool to one or more schedulers or NUMA nodes for greater isolation of CPU resources. To use the Sales and Marketing scenario from previous sections, lets assume that the Sales department needs a more isolated environment and wants 100 percent of a CPU core at all times. By using the AFFINITY option the Sales and Marketing workloads can be scheduled on different CPUs. Assuming the CAP_CPU_PERCENT on the Marketing pool is still in place, the Marketing workload continues to use a maximum of 30 percent of one core, while the Sales workload uses 100 percent of the other core. As far as the Sales and Marketing workloads are concerned, they're running on two isolated machines.

### MIN_IOPS_PER_VOLUME and MAX_IOPS_PER_VOLUME

These settings are the minimum and maximum physical IO operations per second (IOPS) per disk volume for a resource pool. You can use these settings to control the physical IOs issued for user threads for a given resource pool. For example, the Sales department generates several end-of-month reports in large batches. The queries in these batches can generate IOs that can saturate the disk volume and affect the performance of other higher priority workloads in the database. To isolate this workload, the MIN_IOPS_PER_VOLUME is set to 20 and the MAX_IOPS_PER_VOLUME is set to 100 for the Sales department resource pool. These settings control the level of IOs that can be issued for the workload.

### System and user-defined resource pools

Resource Governor predefines two resource pools, the internal pool and the default pool. You can create additional pools.

### Internal Pool

The internal pool represents the resources consumed by the [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] itself. This pool always contains only the internal group, and the pool can't be modified in any way. Resource consumption by the internal pool isn't restricted. Any workloads in the pool are considered critical for server function. Therefore, the Resource Governor allows the internal pool to pressure other pools even if it means the violation of limits set for the other pools.

> [!NOTE]  
> The internal pool and internal group resource usage is not subtracted from the overall resource usage. Percentages are calculated from the overall resources available.

### Default Pool

The default pool is the first predefined user pool. Prior to any configuration the default pool only contains the default group. You can't create or drop the default pool but you can alter it. The default pool can contain user-defined groups in addition to the default group. Beginning with [!INCLUDE[sssql16-md](../../includes/sssql16-md.md)] there's a default resource pool for routine [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] operations, and a default external resource pool for external processes, such as executing R scripts.

> [!NOTE]  
> The default group is alterable but it cannot be moved out of the default pool.

### External Pool

Users can create an external pool to define resources for the external processes. For R Services, this pool specifically governs `rterm.exe`, `BxlServer.exe`, `python.exe` and other processes spawned by them. For more information, see [CREATE EXTERNAL RESOURCE POOL](../../t-sql/statements/create-external-resource-pool-transact-sql.md)

### User-Defined Resource Pools

User-defined resource pools are those that you create for specific workloads in your environment. Resource Governor provides DDL statements for creating, changing, and dropping resource pools. For more information, see [Create a Resource Pool](create-a-resource-pool.md), [Delete a Resource Pool](delete-a-resource-pool.md), and [Change Resource Pool Settings](change-resource-pool-settings.md).

## Resource allocation between pools

When configuring CPU or Memory, the sum of MIN values across all pools can't exceed 100 percent of the server resources. In addition, when configuring CPU or Memory, MAX and CAP values can be set anywhere in the range between MIN and 100 percent inclusive.  
If a pool has a nonzero MIN defined, the effective MAX value of other pools is readjusted. The minimum of the configured MAX value of a pool and the sum of the MIN values of other pools is subtracted from 100 percent.

The following table illustrates a few of the preceding concepts. The table shows the settings for the internal pool, the default pool, and two user-defined pools.

| Pool name | MIN % setting | MAX % setting | Calculated effective MAX % | Calculated shared % | Comment |
| --- | --- | --- | --- | --- | --- |
| internal | 0 | 100 | 100 | 0 | Effective MAX% and shared% aren't applicable to the internal pool. |
| default | 0 | 100 | 30 | 30 | The effective MAX value is calculated as: min(100,100-(20+50)) = 30. The calculated shared % is effective MAX - MIN = 30. |
| Pool 1 | 20 | 100 | 50 | 30 | The effective MAX value is calculated as: min(100,100-50) = 50. The calculated Shared % is Effective MAX - MIN = 30. |
| Pool 2 | 50 | 70 | 70 | 20 | The effective MAX value is calculated as: min(70,100-20) = 70. The calculated Shared % is Effective MAX - MIN = 20. |

The following formulas are used for calculating the effective MAX% and the shared % in the table:

- Min(X,Y) means the smaller value of X and Y.

- Sum(X) means the sum of value X across all pools.

- Total shared % = 100 - sum(MIN %).

- Effective MAX % = min(X,Y).

- Shared % = Effective MAX % - MIN %.

Using the preceding table as an example, we can further illustrate the adjustments that take place when another pool is created. This pool is Pool 3 and has a MIN % setting of 5.

| Pool name | MIN % setting | MAX % setting | Calculated effective MAX % | Calculated shared % | Comment |
| --- | --- | --- | --- | --- | --- |
| internal | 0 | 100 | 100 | 0 | Effective MAX % and shared % aren't applicable to the internal pool. |
| default | 0 | 100 | 25 | 25 | The effective MAX value is calculated as: min(100,100-(20+50+5)) = 25. The calculated shared % is Effective MAX - MIN = 25. |
| Pool 1 | 20 | 100 | 45 | 25 | The effective MAX value is calculated as: min(100,100-55) = 45. The calculated Shared % is Effective MAX - MIN = 25. |
| Pool 2 | 50 | 70 | 70 | 20 | The effective MAX value is calculated as: min(70,100-25) = 70. The calculated Shared % is effective MAX - MIN = 20. |
| Pool 3 | 5 | 100 | 30 | 25 | The effective MAX value is calculated as: min(100,100-70) = 30. The calculated Shared % is effective MAX - MIN = 25. |

 The shared part of the pool is used to indicate where available resources can go if resources are available. However, when resources are consumed they go to the specified pool and aren't shared. This behavior may improve resource utilization in cases where there are no requests in a given pool and the resources configured to the pool can be freed up for other pools.

 Some extreme cases of pool configuration are:

- All pools define minimums that in total represent 100 percent of the server resources. In this case, the effective maximums are equal to minimums. This is equivalent to dividing the server resources into nonoverlapping pieces regardless of resources are consumed inside any given pool.

- All pools have zero minimums. All the pools compete for available resources and their final sizes are based on resource consumption in each pool. Other factors such as policies play a role in shaping the final pool size.

## Resource Pool Tasks

| Task Description | Article |
| --- | --- |
| Describes how to create a resource pool. | [Create a Resource Pool](../../relational-databases/resource-governor/create-a-resource-pool.md) |
| Describes how to change resource pool settings. | [Change Resource Pool Settings](../../relational-databases/resource-governor/change-resource-pool-settings.md) |
| Describes how to delete a resource pool. | [Delete a Resource Pool](../../relational-databases/resource-governor/delete-a-resource-pool.md) |

## See also

- [Resource Governor](../../relational-databases/resource-governor/resource-governor.md)
- [Resource Governor Workload Group](../../relational-databases/resource-governor/resource-governor-workload-group.md)
- [Resource Governor Classifier Function](../../relational-databases/resource-governor/resource-governor-classifier-function.md)
- [Configure Resource Governor Using a Template](../../relational-databases/resource-governor/configure-resource-governor-using-a-template.md)
- [View Resource Governor Properties](../../relational-databases/resource-governor/view-resource-governor-properties.md)
