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