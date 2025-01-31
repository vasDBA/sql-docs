---
title: "sp_cursoroption (Transact-SQL)"
description: sp_cursoroption sets cursor options or returns cursor information created by the `sp_cursoropen` stored procedure.
author: markingmyname
ms.author: maghan
ms.reviewer: randolphwest
ms.date: 07/05/2024
ms.service: sql
ms.subservice: system-objects
ms.topic: "reference"
f1_keywords:
  - "sp_cursoroption_TSQL"
  - "sp_cursoroption"
helpviewer_keywords:
  - "sp_cursoroption"
dev_langs:
  - "TSQL"
---
# sp_cursoroption (Transact-SQL)

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

Sets cursor options or returns cursor information created by the `sp_cursoropen` stored procedure. `sp_cursoroption` is invoked by specifying `ID = 8` in a tabular data stream (TDS) packet.

:::image type="icon" source="../../includes/media/topic-link-icon.svg" border="false"::: [Transact-SQL syntax conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)

## Syntax

```syntaxsql
sp_cursoroption cursor , code , value
[ ; ]
```

## Arguments

#### *cursor*

A `handle` value generated by [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)], and returned by the `sp_cursoropen` stored procedure. The *cursor* parameter is **int** and can't be `NULL`.

#### *code*

Used to stipulate various factors of the cursor return values. The *code* parameter is **int**, and must be one of the following values:

| Value | Name | Description |
| --- | --- | --- |
| `0x0001` | `TEXTPTR_ONLY` | Returns the text pointer, and not the actual data, for certain designated text or image columns.<br /><br />`TEXTPTR_ONLY` allows text pointers to be used as *handles* to blob objects that can later be selectively retrieved or updated using [!INCLUDE [tsql](../../includes/tsql-md.md)] or DBLIB facilities (for example, [!INCLUDE [tsql](../../includes/tsql-md.md)] `READTEXT` or DBLIB `DBWRITETEXT`).<br />If a `0` value is assigned, all of the text and image columns in the select list return text pointers rather than data. |
| `0x0002` | `CURSOR_NAME` | Assigns the name specified in *value* to the cursor, which allows ODBC to use [!INCLUDE [tsql](../../includes/tsql-md.md)] positioned `UPDATE`/`DELETE` statements on cursors opened via `sp_cursoropen`.<br /><br />The string can be specified as any character or Unicode data type.<br />Since [!INCLUDE [tsql](../../includes/tsql-md.md)] positioned `UPDATE`/`DELETE` statements operate, by default, on the first row in a fat cursor, `sp_cursor SETPOSITION` should be used to position the cursor before issuing the positioned `UPDATE`/`DELETE` statement. |
| `0x0003` | `TEXTDATA` | Returns the actual data, not the text pointer, for certain text or image columns on subsequent fetches (that is, this undoes the effect of `TEXTPTR_ONLY`).<br /><br />If `TEXTDATA` is enabled for a particular column, the row is refetched or refreshed, and can then be set back to `TEXTPTR_ONLY`. As with `TEXTPTR_ONLY`, the value parameter is an integer that specifies the column number and a zero value returns all text or image columns. |
| `0x0004` | `SCROLLOPT` | Scroll option. For more information, see [Return code values](#return-code-values). |
| `0x0005` | `CCOPT` | Concurrency control option. For more information, see [Return code values](#return-code-values). |
| `0x0006` | `ROWCOUNT` | The number of rows currently in the result set.<br /><br />**Note:** The `ROWCOUNT` might have changed since the value returned by `sp_cursoropen` if asynchronous population is being used. The value `-1` is returned if the number of rows is unknown. |

#### *value*

Designates the value returned by *code*. *value* is a required parameter that calls for a `0x0001`, `0x0002`, or `0x0003` *code* input value.

A *code* value of `2` is a string data type. Any other *code* value input or returned by *value* is an **int**.

## Return code values

The *value* parameter might return one of the following *code* values.

| Return value | Description |
| --- | --- |
| `0x0004` | `SCROLLOPT` |
| `0X0005` | `CCOPT` |
| `0X0006` | `ROWCOUNT` |

The *value* parameter returns one of the following `SCROLLOPT` values.

| Return value | Description |
| --- | --- |
| `0x0001` | `KEYSET` |
| `0x0002` | `DYNAMIC` |
| `0x0004` | `FORWARD_ONLY` |
| `0x0008` | `STATIC` |

The *value* parameter returns one of the following `CCOPT` values.

| Return value | Description |
| --- | --- |
| `0x0001` | `READ_ONLY` |
| `0x0002` | `SCROLL_LOCKS` |
| `0x0004 or 0x0008` | `OPTIMISTIC` |

## Related content

- [System stored procedures (Transact-SQL)](system-stored-procedures-transact-sql.md)
- [sp_cursor (Transact-SQL)](sp-cursor-transact-sql.md)
- [sp_cursoropen (Transact-SQL)](sp-cursoropen-transact-sql.md)
