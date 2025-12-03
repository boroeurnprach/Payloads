Most database types (except Oracle) have a set of views called the information schema. This provides information about the database.

For example, you can query `information_schema.tables` to list the tables in the database:
```sql
SELECT * FROM information_schema.tables
```
This returns output like the following:
```
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME TABLE_TYPE 
===================================================== 
MyDatabase     dbo         Products   BASE TABLE 
MyDatabase     dbo         Users      BASE TABLE 
MyDatabase     dbo         Feedback   BASE TABLE
```

This output indicates that there are three tables, called `Products`, `Users`, and `Feedback`.
You can then query `information-schema.columns` to list the columns in individual tables:
```sql
SELECT * FROM information_schema.columns WHERE table_name = 'Users'
```

This returns output like the following:

```
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME COLUMN_NAME DATA_TYPE 
================================================================= 
MyDatabase     dbo         Users      UserId      int 
MyDatabase     dbo         Users      Username    varchar 
MyDatabase     dbo         Users      Password    varchar
```

This output shows the columns in the specified table and the data type of each columns.