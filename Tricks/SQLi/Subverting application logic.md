SQL injection allows login as any user by commenting out password check:  
Enter `administrator'--` as username, leave password blank.  
This modifies the query to ignore the password condition, granting unauthorized access.