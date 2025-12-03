### Extracting field name
```javascript
"$where":"Object.key(this)[0].match('^.{0}a.*')"
```
The query searches for documents where **the first field name in the document** matches a specific regex pattern.

### Exfiltrating data using operators

You may be able to extract data using operators that don't enable to run JavaScript. you can use the `$regex` operator to extract data character by character.
Consider a vulnerable application that accepts a username and password in the body of a `POST` request. For example:

`{"username":"myuser","password":"mypass"}`

You could start by testing whether the `$regex` operator is processed as follows:

`{"username":"admin","password":{"$regex":"^.*"}}`

If the response to this request is different to the one you receive when you submit an incorrect password, this indicates that the application may be vulnerable. You can use the `$regex` operator to extract data character by character. For example, the following payload checks whether the password begins with an `a`:

`{"username":"admin","password":{"$regex":"^a*"}}`