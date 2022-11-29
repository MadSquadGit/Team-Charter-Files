
```toc
```


## Keys

### Unique Keys

1. Open table Designer
2. Right Click
3. Select Index/Keys
4. Add
5. Make sure type is Unique Key
6. Rename to UX_`Descriptive_Name`
7. Go To Options for teh column
8. Choose Column
9. Save and close


## Indexes

### Script
```sql
USE [DigitalCreditPaper] GOCREATE NONCLUSTERED INDEX [IX_FinancialCommentaryTypeId] ON [Application].[ApplicationFinancialCommentary] ( [FinancialCommentaryTypeId] ASC)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, SORT_IN_TEMPDB = OFF, DROP_EXISTING = OFF, ONLINE = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY] GO
```

### Method
1. Open table Designer
2. Right Click
3. Select Index/Keys
4. Add
5. Make sure type is index
6. Open options for columns
7. Choose columns
8. Save and close



## Code Models

### Convert Table to C# Model
```sql
declare @TableName sysname = 'TableName'
declare @Result varchar(max) = 'public class ' + @TableName + '
{'

select @Result = @Result + '
    public ' + ColumnType + NullableSign + ' ' + ColumnName + ' { get; set; }
'
from
(
    select 
        replace(col.name, ' ', '_') ColumnName,
        column_id ColumnId,
        case typ.name 
            when 'bigint' then 'long'
            when 'binary' then 'byte[]'
            when 'bit' then 'bool'
            when 'char' then 'string'
            when 'date' then 'DateTime'
            when 'datetime' then 'DateTime'
            when 'datetime2' then 'DateTime'
            when 'datetimeoffset' then 'DateTimeOffset'
            when 'decimal' then 'decimal'
            when 'float' then 'double'
            when 'image' then 'byte[]'
            when 'int' then 'int'
            when 'money' then 'decimal'
            when 'nchar' then 'string'
            when 'ntext' then 'string'
            when 'numeric' then 'decimal'
            when 'nvarchar' then 'string'
            when 'real' then 'float'
            when 'smalldatetime' then 'DateTime'
            when 'smallint' then 'short'
            when 'smallmoney' then 'decimal'
            when 'text' then 'string'
            when 'time' then 'TimeSpan'
            when 'timestamp' then 'long'
            when 'tinyint' then 'byte'
            when 'uniqueidentifier' then 'Guid'
            when 'varbinary' then 'byte[]'
            when 'varchar' then 'string'
            else 'UNKNOWN_' + typ.name
        end ColumnType,
        case 
            when col.is_nullable = 1 and typ.name in ('bigint', 'bit', 'date', 'datetime', 'datetime2', 'datetimeoffset', 'decimal', 'float', 'int', 'money', 'numeric', 'real', 'smalldatetime', 'smallint', 'smallmoney', 'time', 'tinyint', 'uniqueidentifier') 
            then '?' 
            else '' 
        end NullableSign
    from sys.columns col
        join sys.types typ on
            col.system_type_id = typ.system_type_id AND col.user_type_id = typ.user_type_id
    where object_id = object_id(@TableName)
) t
order by ColumnId

set @Result = @Result  + '
}'

print @Result
```


### Convert Table to TypeScript Model
```sql
declare @TableName sysname = 'TableNamee'
declare @Result varchar(max) = 'export class ' + @TableName + '
{'

select @Result = @Result + '
    ' + ColumnName + ': ' + ColumnType + ';
'
from
(
    select 
        LOWER(LEFT(replace(col.name, ' ', '_'),1))+SUBSTRING(replace(col.name, ' ', '_'),2,LEN(replace(col.name, ' ', '_'))) ColumnName,
        column_id ColumnId,
        case typ.name 
            when 'bigint' then 'number'
            when 'binary' then 'byte[]'
            when 'bit' then 'boolean'
            when 'char' then 'string'
            when 'date' then 'Date'
            when 'datetime' then 'Date'
            when 'datetime2' then 'Date'
            when 'datetimeoffset' then 'Date'
            when 'decimal' then 'number'
            when 'float' then 'number'
            when 'image' then 'byte[]'
            when 'int' then 'number'
            when 'money' then 'number'
            when 'nchar' then 'string'
            when 'ntext' then 'string'
            when 'numeric' then 'number'
            when 'nvarchar' then 'string'
            when 'real' then 'number'
            when 'smalldatetime' then 'Date'
            when 'smallint' then 'number'
            when 'smallmoney' then 'number'
            when 'text' then 'string'
            when 'time' then 'Time'
            when 'timestamp' then 'number'
            when 'tinyint' then 'byte'
            when 'uniqueidentifier' then 'string'
            when 'varbinary' then 'byte[]'
            when 'varchar' then 'string'
            else 'UNKNOWN_' + typ.name
        end ColumnType,
        case 
            when col.is_nullable = 1 and typ.name in ('bigint', 'bit', 'date', 'datetime', 'datetime2', 'datetimeoffset', 'decimal', 'float', 'int', 'money', 'numeric', 'real', 'smalldatetime', 'smallint', 'smallmoney', 'time', 'tinyint', 'uniqueidentifier') 
            then '?' 
            else '' 
        end NullableSign
    from sys.columns col
        join sys.types typ on
            col.system_type_id = typ.system_type_id AND col.user_type_id = typ.user_type_id
    where object_id = object_id(@TableName)
) t
order by ColumnId

set @Result = @Result  + '
}'

print @Result
```

## Temporal System Versioned Table

```sql
--Original Table
ALTER TABLE 
  [ESnR].[Summary] 
ADD 
  [VersionStartTime] [DATETIME2](7) GENERATED ALWAYS AS ROW START HIDDEN CONSTRAINT [DF_Summary_VersionStartTime] DEFAULT SYSUTCDATETIME(), 
  [VersionEndTime] [DATETIME2](7) GENERATED ALWAYS AS ROW END HIDDEN CONSTRAINT [DF_Summary_VersionEndTime] DEFAULT CONVERT(
    DATETIME2, '9999-12-31 23:59:59.9999999'
  ), 
  PERIOD FOR SYSTEM_TIME (
    [VersionStartTime], [VersionEndTime]
  );
GO 

--Add Temporal Table
ALTER TABLE 
  [ESnR].[Summary] 
SET 
  (
    SYSTEM_VERSIONING = ON (HISTORY_TABLE = History.Summary)
  );
```

### Undo Table
- This converts a temporal table to a normal table to be deleted
```sql
ALTER TABLE ESNR.AssesssmentSummary SET (SYSTEM_VERSIONING = ON);
ALTER TABLE ESNR.AssesssmentSummary ADD PERIOD FOR SYSTEM_TIME;
```
