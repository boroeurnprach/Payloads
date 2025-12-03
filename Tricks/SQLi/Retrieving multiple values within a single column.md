In some cases the query in the previous example may only return a single column.

You can retrieve multiple values together within this single column by concatenating the values together. You can include a separator to let you distinguish the combined values. For example, on Oracle you could submit the input:

```sql
' UNION SELECT username || '~' || password FROM users--
```

This uses the double-pipe sequence `||` which is a string concatenation operator on Oracle. The injected query concatenates together the values of the `username` and `password` fields, separated by the `~` character.

The results from the query contain all the usernames and passwords, for example:

`
```json
---
administrator~s3cure 
wiener~peter 
carlos~montoya
---
```

Different databases use different syntax to perform string concatenation. For more details, see  [[https://portswigger.net/web-security/sql-injection/cheat-sheet]].