---
title: Monitor XTP In-memory storage
description: Estimate and monitor XTP in-memory storage usage and capacity in Azure SQL Managed Instance; resolve capacity error 41823.
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: mathoma
ms.date: 01/16/2024
ms.service: azure-sql-managed-instance
ms.subservice: performance
ms.topic: how-to
monikerRange: "=azuresql||=azuresql-mi"
---
# Monitor in-memory OLTP storage in Azure SQL Managed Instance
[!INCLUDE [appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

> [!div class="op_single_selector"]
> * [Azure SQL Database](../database/in-memory-oltp-monitor-space.md?view=azuresql-db&preserve-view=true)
> * [Azure SQL Managed Instance](in-memory-oltp-monitor-space.md?view=azuresql-mi&preserve-view=true)

With [in-memory OLTP](in-memory-oltp-overview.md), data in memory-optimized tables and table variables resides in in-memory OLTP storage.

## Determine whether data fits within the in-memory OLTP storage cap

The Business Critical service tier includes a certain amount of **Max In-Memory OLTP memory**, [determined by the number of vCores](resource-limits.md?view=azuresql-mi&preserve-view=true).

Estimating memory requirements for a memory-optimized table works the same way for SQL Server as it does in Azure SQL Managed Instance. Take a few minutes to review [Estimate memory requirements](/sql/relational-databases/in-memory-oltp/estimate-memory-requirements-for-memory-optimized-tables?view=azuresqldb-mi-current&preserve-view=true).

Table and table variable rows, as well as indexes, count toward the max user data size. In addition, `ALTER TABLE` needs enough room to create a new version of the entire table and its indexes.

Once this limit is exceeded, insert and update operations might start failing with error 41823.

## Correct out-of-memory OLTP storage situations - error 41823

Meeting the in-memory OLTP storage cap in your database results in INSERT, UPDATE, ALTER and CREATE operations failing with error 41823. This error can cause the active transaction to abort.

Error 41823 indicate that the memory-optimized tables and table variables in the instance reached the maximum in-memory OLTP storage size.

To resolve this error, either:

- Delete data from the memory-optimized tables, potentially offloading the data to traditional, disk-based tables; or,
- Upgrade the vCore count add in-memory storage for the data you need to keep in memory-optimized tables.

> [!NOTE]
> In rare cases, error 41823 can be transient, meaning there is enough available in-memory OLTP storage, and retrying the operation succeeds. We therefore recommend to both monitor the overall available in-memory OLTP storage and to retry when first encountering error 41823. For more information about retry logic, see [Conflict Detection and Retry Logic with in-memory OLTP](/sql/relational-databases/in-memory-oltp/transactions-with-memory-optimized-tables#conflict-detection-and-retry-logic).

## Monitor with DMVs


- By monitoring the memory consumption periodically, you can determine how memory consumption is growing and how much head room you have left in the resource limits. Identify how much memory is being consumed by the objects in your database or instance. For example, the DMVs [sys.dm_db_xtp_table_memory_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-xtp-table-memory-stats-transact-sql?view=azuresqldb-mi-current&preserve-view=true) or [sys.dm_os_memory_clerks](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-memory-clerks-transact-sql?view=azuresqldb-mi-current&preserve-view=true).

    - You can find memory consumption for all user tables, indexes, and system objects by querying `sys.dm_db_xtp_table_memory_stats`:

        ```sql  
        SELECT object_name(object_id) AS [Name], *  
           FROM sys.dm_db_xtp_table_memory_stats;
        ```

    - Memory allocated to in-memory OLTP engine and the memory-optimized objects is managed the same way as any other memory consumer within a database. The memory clerks of type MEMORYCLERK_XTP accounts for all the memory allocated to the in-memory OLTP engine. Use the following query on `sys.dm_os_memory_clerks` to find all the memory used by the in-memory OLTP engine, including memory dedicated to specific databases.
  
        ```sql  
        -- This DMV accounts for all memory used by the in-memory engine  
        SELECT [type], [name]
             , memory_node_id  
             , pages_kb/1024 AS pages_MB   
        FROM sys.dm_os_memory_clerks 
        WHERE [type] LIKE '%xtp%';
        ```  

        ```output
        type                 name       memory_node_id pages_MB  
        -------------------- ---------- -------------- --------------------  
        MEMORYCLERK_XTP      Default    0              18  
        MEMORYCLERK_XTP      DB_ID_5    0              1358  
        MEMORYCLERK_XTP      Default    64             0  
    ```

- You can also get more information about out of memory errors in Azure SQL Managed Instance with the dynamic management view [sys.dm_os_out_of_memory_events](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-out-of-memory-events?view=azuresqldb-mi-current&preserve-view=true). For example:

    ```sql
    SELECT * FROM sys.dm_os_out_of_memory_events ORDER BY event_time DESC;
    ```

    For more information, see [Monitor and troubleshoot in-Memory OLTP memory usage](/sql/relational-databases/in-memory-oltp/monitor-and-troubleshoot-memory-usage?view=azuresqldb-mi-current&preserve-view=true).

## Related content

- [Monitor Microsoft Azure SQL Managed Instance performance using dynamic management views](monitoring-with-dmvs.md?view=azuresql-mi&preserve-view=true)
- [In-memory sample in Azure SQL Managed Instance](in-memory-oltp-sample.md?view=azuresql-mi&preserve-view=true)