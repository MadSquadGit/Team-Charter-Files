## Create Table

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

## Undo Table
- This converts a temporal table to a normal table to be deleted
```sql
ALTER TABLE ESNR.AssesssmentSummary SET (SYSTEM_VERSIONING = ON);
ALTER TABLE ESNR.AssesssmentSummary ADD PERIOD FOR SYSTEM_TIME;
```
