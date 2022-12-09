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