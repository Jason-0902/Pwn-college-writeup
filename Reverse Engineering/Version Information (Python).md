# Version Information (Python)

## Challenge Description

This challenge introduces the concept of file format versioning, where a magic number is not the only piece of metadata required — the program also validates a version number immediately following the magic bytes.

The goal is to reverse the `/challenge/cimg` binary, determine the correct magic number and version, and craft a file that passes the validation.

The challenge is implemented in Python, wrapped as a setuid executable.

## Reversing the Binary

We quickly discover that /challenge/cimg is not an ELF binary:

```bash
$ file /challenge/cimg
/challenge/cimg: setuid Python script, ASCII text executable
```

So we read the script directly:

```bash
$ head -n 40 /challenge/cimg
```

Key logic found:

```Python
header = file.read1(6)
assert len(header) == 6, "ERROR: Failed to read header!"

assert header[:4] == b"cm6e", "ERROR: Invalid magic number!"
assert int.from_bytes(header[4:6], "little") == 135, "ERROR: Invalid version!"
```

We learn:

The file must be at least 6 bytes long

The magic number is: b"????"

Exploit / File Creation

We crafted a valid .cimg file using a small Python script:

## Exploit / File Creation

We crafted a valid .cimg file using a small Python script:

```Python
with open("flag.cimg", "wb") as f:
    f.write(b"????")
    f.write(b"\x00\x00") 
```

Run it:

```bash
$ python3 make_cimg.py
$ /challenge/cimg flag.cimg
```